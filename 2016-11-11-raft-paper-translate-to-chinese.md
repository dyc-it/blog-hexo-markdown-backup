---
title: Raft论文翻译
date: 2016-11-11 15:53:05
categories: translate
tags: Raft
---

翻译了Diego Ongaro和John Ousterhout的论文《In Search of an Understandable Consensus Algorithm(Extended Version)》[原文地址](https://raft.github.io/raft.pdf)

<!-- more -->

# 摘要

&emsp;&emsp;Raft是一种管理复制日志的一致性算法。Raft产生的结果和Paxos或者multi-Paxos是一样的，并且Raft和Paxos一样高效，但是Raft的结构和Paxos是不一样的；这样让Raft比Paxos更容易理解并且为构建使用系统提供了更好的基础。为了增强可理解性，Raft分离了一致性算法的关键要素，比如leader选举，日志复制和安全性，并强制强一致性来减少必须要被考虑的状态。用户的研究结果表明，对学生来说，Raft比Paxos更容易学习。Raft也包括改变集群成员的新机制：使用重叠大多数成员保证安全性。

# 1 介绍
&emsp;&emsp;一致性算法可以让一批机器作为一个相关的组，这样即使组里面有机器故障，但是整个组仍然能够对外提供服务。因为这样，一致性算法在构建一个可靠的大规模软件系统中扮演者关键角色。在过去十年中，Paxos算法在一致性算法的讨论中一直占主导地位。大多数一致性算法的实现都基于Paxos算法或者受其影响，并且Paxos已经作为教学生一致性算法的主要工具。  
&emsp;&emsp;不幸的是，Paxos非常难于理解，尽管为了让它更通熟易懂做了无数的尝试。此外，Paxos的架构让它需要作出复杂的改变才能支持实用的系统。因此，系统集成商和学生都在和Paxos算法作斗争。  
&emsp;&emsp;在我们和Paxos算法斗争之后，我们开始寻找一个新的一致性算法，新的算法可以为软件系统建设和教学提供一个更好的基础。我们的方法——主要目标是易于理解——是与众不同的：我们可以为实用系统定义一个一致性算法，并且这个算法比Paxos更容易学习？此外，我们希望该算法对系统建设者来说更为直观。不仅仅是让算法有效，并且让该算法有效看起来显而易见也很重要。  
&emsp;&emsp;这个工作的成果就是被称为Raft的一致性算法。在设计Raft时，我们采用了特定的技术来提高可理解性，包括分解（Raft分离了leader选举，日志复制和安全性）和状态空间减少（相对于Paxos，Raft减少了不确定性程度和某个server会与其他server不一致的途径）。在两所大学共43名学生参与的用户研究表明，Raft比Paxos更容易理解：在学完两个两个算法后，其中33名学生在Raft算法上回答的比Paxos更好。  
&emsp;&emsp;Raft在很多方面和已有的一致性算法很像，但是它有一些新颖的特点：
  
- strong leader: Raft比其他一致性算法使用更强形式的领导力。比如，日志入口只能从leader流向其他server。这简化了日志复制的管理并使Raft更容易理解。
- leader选举：Raft使用随机的定时器去选举leader。这在其他一致性算法都需要的心跳检测上增加了一点机制，但是快速且简单地解决了冲突。
- 成员变化：Raft在变更集群中server集合成员时，使用了新的联合一致性方法，在过度期间两个不同配置的大多数成员有交集。这可以让集群在配置改变期间也可以继续正常操作。  

&emsp;&emsp;我们认为Raft算法优于Paxos算法和其他共识算法，不论是作为教学还是作为实现的基础。Raft算法比其他算法更简单并更容易理解；它被充分描述足以满足一个实际系统的需求；它有很多开源实现并且被很多公司使用；它的安全性已经被证实指定和证明；其效率也可以和其他算法相媲美。这篇论文剩下的部分介绍了复制状态机问题（第2部分），讨论了Paxos算法的优缺点（第3部分），描述了为了可理解性我们采用的一般方法（第4部分），展示了Raft一致性算法（第5到8部分），评估了Raft算法（第9部分），并且讨论了相关的工作（第10部分）。

# 2 复制状态机
&emsp;&emsp;一致性算法通常出现在复制状态机的情景中。在这个方法中，状态机在一批server上，计算相同状态的相同副本，即使某些server宕机了也可以继续操作。复制状态机在分布式系统中被用来解决很多容错问题。比如，只有一个leader的大规模系统，GFS，HDFS和RAMCloud，通常使用单独的复制状态机去管理leader选举和leader宕机时也必须可用的配置信息。复制状态机的例子包括Chubby和Zookeeper。 
![](1.png)  
>**图1:** 复制状态机架构。一致性算法管理着复制日志，日志包含从client发来的命令。状态机按照完全相同的顺序处理日志中的命令，所以他们产生一样的输出。

&emsp;&emsp;复制状态机通常使用复制日志的方式实现，像图1中的那样。每一个存储了一个包括一系列命令的日志，状态机按照顺序执行这些日志。每一个日志包含相同的命令，并且顺序也相同，所以每一个状态机执行相同的命令序列。因为状态机是确定性的，每一个计算相同的状态并有相同的输出序列。  
&emsp;&emsp;一致性算法的工作是保持复制日志一致。server上的一致性模块从client接受命令，让后把命令加入到一致性模块的日志中。server上的一致性模块和其他server上的一致性模块通信，确保每一份日志最终都包含相同的请求，并且顺序是一样的，即使某些server故障了。一旦命令被正确地复制了，每一个server上的状态机使用日志中的顺序处理它们，然后输出被返回给client。结果是，所有的server看起来组成了一个单一的、高度可靠的状态机。
&emsp;&emsp;为实际系统设计的一致性算法通常具有下面的属性：

- 在所有非拜占庭错误的情况下保证安全性（从来不会返回一个不正确的值），包括在网络延迟、分区、丢包、重复和重新排序的情况下都可以保证安全性
- 只要大多数的server正常工作并且可以和其他server和client通信，那么算法是充分可用的。因此，通常有5个节点的集群可以容忍2个server失效。假设server是通过停止来模拟失效的；以后失效的server可以从永久存储上的状态恢复过来，然后重新加入集群。
- 不依赖计时来保证日志的一致性：时钟错误和极端的消息延时在最坏的情况下会导致可用性问题
- 在通常情况下，集群中的大多数节点响应了一轮远程过程调用后，一条命令就执行完了；少部分慢的server不需要影响整体系统性能。

# 3 Paxos的问题
&emsp;&emsp;在过去十年，Leslie Lamport的Paxos协议几乎成了一致性算法的代名词：这是在课堂上被教的最多的协议，并且大多数一致性算法的实现都使用Paxos作为起点。Paxos首先定义了一个能够在单个决策达成一致的协议，例如单个复制日志条目。我们将这个子集称为单一法令Paxos。然后Paxos组合这个协议的多个实例以促进一些列决策，比如日志（multi-Paxos）。Paxos确保安全性和liveness，并且它支持集群成员的改变。Paxos的正确性已经被证明，并且在一般情况下是高效的。  
&emsp;&emsp;不幸的是，Paxos有两个明显的缺点。第一个，Paxos是非常难以理解的。完整的解释是臭名昭著的不透明；很少有人能够理解，并且必须付出巨大的努力。结果，有很多尝试使用更简单的术语解释Paxos。这些解释集中在单一法令子集，即使这样也是充满挑战。在对NSDI 2012与会者做的非正式调查中，我们发现很少有人喜欢Paxos，即使是经验丰富的研究人员。我们自己与Paxos斗争；直到我们阅读了很多简单的解释并设计了我们自己的可选协议后，我们才能理解完整的Paxos算法，这个过程花了我们将近一年的时间。   
&emsp;&emsp;我们假设Paxos的不透明来源于：Paxos选择了单一法令子集作为它的基础。单一法令Paxos是密集和微妙的：它被分为两个阶段，对这两个阶段没有直观的解释并且不能被独立理解。因此，对于单一法令协议为什么能够工作很难有直观的看法。multi-Paxos的组合规则明显增加了复杂性。我们认为，就多项决定达成共识的总体问题（比如，日志而不是日志项）可以被分解为更直接和明显的方式。  
&emsp;&emsp;Paxos的第二个问题是不能为构建一个实用的实现提供一个好的基础。一个原因是在multi-Paxos上没有达成广泛的共识。Lamport的描述大多数是关于single-decree Paxos的，他描绘了multi-Paxos的多种方法，但是很多细节都缺失。已经有很多尝试去丰富和优化Paxos，但是这些尝试都不一样并且和Lamport的描绘也不太一样。比如像Chubby这样的系统实现了类Paxos算法，但是在大多数情况下，它们的细节并没有公布。  
&emsp;&emsp;此外，Paxos架构对于构建实际系统是一个糟糕的架构；这是single-decree分解的另一个结果。例如，独立选择一批日志项并把它们融合到顺序日志中并没有什么好处；这样只是增加了复杂度。设计一个围绕日志的系统更为简单和高效，新的日志项按照约定的顺序顺序地附加。Paxos的另外一个问题是它使用对称的点对点方法（尽管它最终表明一种弱形式的领导作为性能优化）。在一个只需要作出一个决定的简单世界中，这就说得通了，但是很少有实际系统使用这种方法。如果必须作出一系列的决定，那么先选举一个leader然后让这个leader协调作出决定，就更为简单和快速了。  
&emsp;&emsp;结果，实际系统与Paxos几乎没有什么相似之处。每一种实现都是从Paxos开始，发现实现它很困难，然后开发了一个显著不同的架构。这是耗时且容易出错的，理解Paxos的困难加剧了这个问题。Paxos的公式可能是一个很好证明其正确的定理，但是实际的实现和Paxos差别这么大，让Paxos易于证明的特性就没有很大的价值了。以下来自Chubby实现者的注释是典型的：

```
在Paxos算法的描述和真实世界系统的需求之间，存在着巨大的鸿沟
...
最终的系统将会基于未经证实的协议。
```   
由于这些原因，我们认为Paxos不能为系统建设或者教学提供良好的基础。鉴于在大型软件系统中一致性算法的重要性，我们决定看我们能不能设计一个比Paxos更好的替代的一致性算法。Raft是这次试验的结果。
# 4 为了易于理解设计
&emsp;&emsp;我们在设计Raft的时候有很多目标：它必须为系统建设提供一个完整和实际的基础，从而可以显著减少开发者的设计工作量；在所有情况下它必须保证安全，在典型的操作条件下必须可用；在通常的操作中必须高效。但是我们最重要的目标——也是最具挑战性——是易于理解。对于大量听众来说，必须能够很好地理解该算法。此外，这个算法必须看起来很直观，所以系统建设者可以在现实世界的实现中做扩展（扩展不可避免）。  
&emsp;&emsp;在Raft的设计中，有很多点，我们不得不在替代方法中选择。在这样的情况下，我们基于易于理解性来评估可选的方法：解释每个选择的难易程度是怎么样的（比如，状态空间的复杂度是怎么样的？它是否具有微妙的含义？），让读者完全理解这个方法及其实现是否很容易？  
&emsp;&emsp;我们认识到这种分析存在高度的主观性；尽管如此，我们使用两种通用的技术。第一种技术是众所周知的问题分解方法：我们尽可能将问题分隔为我们能够相对独立解决、解释和理解的部分。比如，在Raft中我们将leader选举、日志复制、安全和成员变化分开。  
&emsp;&emsp;我们的第二个方式是通过减少需要考虑的状态数量来简化状态空间，这样就可以让系统更加一致并在可能的情况下消除不确定性。具体来说，日志不允许有空洞，Raft限制了日志可能变得不一致的方式。尽管在大多数情况下我们尝试消除不确定定性，有些情况下不确定性实际上提高了可理解性。特别地，随机方法引入非确定性，但是它们倾向于通过用类似方式处理所有可能的选择来减少状态空间（“随便选择；没有关系”）。我们使用随机化简化Raft的leader选举算法。
# 5 Raft一致性算法
&emsp;&emsp;Raft是一个用来管理像第2节中描述的复制日志的算法。图2总结了浓缩形式的算法供参考；图3列出了算法的关键特征；插图的元素将在本节其余的部分分段讨论。  
Raft通过选举一个独特的leader并让这个leader完全负责复制日志的管理来实现一致性算法。这个leader从client接受日志项，在其他server上复制这个日志项，并告诉其他server什么时候将日志项应用到状态机上是安全的。有一个leader简化了复制日志的管理。比如，这个leader可以不用咨询其他的server就可以决定把日志项放在日志中的什么地方，数据流是简单地从leader流向其他server。一个leader可能会故障或者和其他server失去联系，在这种情况下，一个新的leader就会被选举出来。  
&emsp;&emsp;鉴于领导者的方法，Raft将一致性问题分解为三个相关的独立子问题，会在下面的章节中讨论：

- **Leader选举：**在已有的leader故障后，一个新的leader必须被选举出来（5.2节）。
- **日志复制：**leader必须接受client发来的日志项，并且在集群中复制日志项，强制其他节点上的日志和leader一样（5.3节）。
- **安全性：**Raft算法安全性的关键是图3中状态机的安全特性：如果任何server已将一个特定的日志项应用到它的状态机上，那么其他server不会再相同的索引上应用一个不同的命令。5.4节展示了Raft是如何保证这个特性的；解决方法在选举机制上使用了一个附加的限制（5.2节）。

在介绍一致性算法后，本节讨论系统中可用性问题和时序起到的作用。


![](2.png)  
![](3.png)
>**图3:** Raft保证所有时间内这些特性都成立。章节编号指出了讨论这些特性的位置。

## 5.1 Raft基础
&emsp;&emsp;一个raft集群包括多个server，通常有5个，这样两个节点宕机后系统仍能正常工作。在任何时刻，server的状态都属于下面三个状态之一：leader，follower或者candidate。在正常情况下，只有一个leader，并且其他的server都是follower。follower是被动的：它们对自己不发出任何请求，它们只响应leader和candidate的请求。leader处理所有client的请求（如果client向follower发送请求，follower会把请求转发给leader）。第三种状态，candidate被用来选举一个新的leader，在5.2节中会详细介绍。图4展示了这三种状态及他们之间的转换，下面会讨论状态变迁。  
&emsp;&emsp;raft把时间分为任意长度的term，如图5。term被编号为连续的数字。每个term以选举开始，一个或者多个candidate尝试成为leader。如果一个candidate赢得了选举，那么在term的余下部分，它将作为leader对外提供服务。在某些情况下，选举将会导致分裂的投票。在这种情况下，term会结束，并且没有leader；一个新的term（一个新的选举）会马上开始。raft保证在一个给定的term中，最多只有一个leader。  
&emsp;&emsp;不同的server会在不同的时间观察状态的改变，某些场景下一个在整个term内server可能不会观察选举。在Raft中term扮演逻辑时钟的角色，term允许server检测过时的信息比如陈旧的leader。每一个server保存一个当前term编号，随着时间的推移，单调递增。只要server之间相互通信，当前term就会被交换；如果一个server的当前term编号比其他的server小，这个server就会把自己的当前term的值修改为大的那个。如果一个candidate或者leader发现他们的term过时了，它们会立即恢复到follower的状态。如果一个server接收到一个陈旧的term编号，它会拒绝这个请求。  
&emsp;&emsp;Raft的server使用远程方法调用（RPC）进行通信，基本的一致性算法需要两种类型的RPC。RequestVote类型的RPC调用在选举期间被candidate启动（5.2节），AppendEntries类型的PRC调用在leader复制日志项和提供一种形式的心跳的时启动（5.3节）。第7节增加了第3种用于在server之间传输快照的PRC调用。如果没有及时收到RPC响应，server会重发RPC请求，并且多个server并发地发出RPC调用以获取最好的性能。  
## 5.2 leader选举
&emsp;&emsp;Raft使用心跳机制出发leader的选举。当server启动起来时，刚开始他们都是follower。一个server只要它从leader或者candidate收到有效的RPC消息，它就会保持在follower状态。leader周期性地向所有的follower发送心跳（没有日志项的AppendEntries类型的RPC请求）来保持它的权威性。如果一个follower在一段时间内没有收到消息，被称为选举超时，那么follower会假设没有可行的leader然后开始选举新的Leader。  
&emsp;&emsp;为了开始选举，follower增加它当前的term编号然后转变为candidate状态。然后它给自己投票，并行地给集群中的其他server发送RequestVote类型的RPC请求。一个candidate继续在这个状态，直到下面三件事情中，有一件发生了：(a)它赢得了选举(b)其他的server建立的领导地位(c)一段时间过去了还是没有胜出者。在下面会分开讨论这些结果。  
&emsp;&emsp;在某个term内，如果candidate收到了集群中大多数节点的投票，那么它就赢得了选举。在某个给定的term内，一个server最多会给一个candidate投票，基于先来先服务的原则（注意：5.4节在投票上增加了额外的约束）。大多数节点投票的规则保证了在一个term内最多只有一个candidate会被选举为leader（图3中的列出的选举安全特性）。一旦一个candidate赢得了选举，它就成为了leader。然后它会向所有其他的server发送心跳消息来确立它的权威地位并防止新的选举的产生。  
&emsp;&emsp;在等待投票的时候，一个candidate可能会收到其他server发来的声称它是leader的AppendEntries类型的RPC请求。如果leader的term（在它的RPC请求里面）至少和当前这个candidate的term是一样大的，然后这个candidate识别出了这个leader是合法的，并返回到follower状态。如果RPC里面的term比candidate当前的term编号小，这个candidate会驳回这个RPC请求并保持在candidate状态。  
&emsp;&emsp;第三种可能的结果是这个candidate既没有赢得选举，也没有输掉选举：如果同一时刻有很多个follower编程candidate，投票可能会被分散导致没有一个candidate获得大多数的投票。当这种情况发生时，每一个candidate都会超时，增加它们自己的term编号，然后开始新的选举，启动另外一轮RequestVote类型的RPC请求。但是，没有额外的方法的话，投票分散会无止尽地重复下去。  
&emsp;&emsp;Raft使用随机的选举超时时间来确保投票分散很少发生，即使发生也会被很快解决。为了防止投票分散，首先，选举超时时间是在一个固定时间范围内的随机数（比如，150-300ms）。这会将server超时时间点分散开来，所以在大多数情况下只有一个server会超时；在其他server超时之前，赢得了选举并且发送心跳信息。相同的机制被用来处理投票分散。在开始选举的时候，每一个candidate重新开始它的随机选举超时时间，并且它等到超时时间完了后才开始下一次选举；这降低了在新的选举中出现分散投票的可能性。9.3节展示了这种方法可以让leader选举很快完成。  
&emsp;&emsp;选举是一个说明了可理解性是如何知道我们在设计方案之间选择的例子。最初，我们计划使用排名系统：每一个candidate被分配了一个唯一的排名，用于在竞争的候选人之间选择。如果一个candidate发现另一个candidate的排名更高，那么排名低的candidate会返回到follower状态，所以，排名更高的candidate更容易赢得下一次选举。我们发现这种方法引起了可用性问题（如果排名高的server故障了，一个排名更低的server可能需要超时并重新成为candidate，但是如果这样进行的太快，它可以重置选举一个leader的进展）。我们对算法做了多次调整，但是每次调整后，新的问题就来了。最终我们总结，随机尝试的方法更为明显且更容易理解。
# 5.3 log复制
&emsp;&emsp;只要一个leader被选举出来，它开始服务client的请求。每个client的请求包括被复制状态机执行的命令。leader将命令加入到它的log中作为一个新的entry，然后向其他的server并发地发起AppendEntries类型的RPC请求来复制该entry。当entry被成功复制后（像下面描述的那样），leader将entry应用到它的状态机并将执行的结果返回给client。如果follower崩溃或者运行缓慢，或者网络丢包了，leader将会无限期地发送AppendEntries类型的RPC请求（即使leader已经响应了client），直到follower存储了所有的entry。  
![图6](6.png)
>  **图6:** log是由entry组成的，entry按顺序编号。每一个entry包含创建它的term的编号（每个框中的数字）和状态机的命令。如果entry可以安全地被应用到状态机上，那么这个entry就被认为是已经提交了的。 
 
&emsp;&emsp;log被组织成图6中的样子。每一个entry存储了一个状态机命令和leader收到这条命令时对应的term编号。entry中的term编号用于检测log的不连续性，确保图3中的某些特性。每一个entry也包含一个标识它在log中位置的整数索引。  
&emsp;&emsp;leader决定什么时候将entry应用到状态机是安全的；这样的entry被称为已提交。Raft保证所有已经提交的entry是持久的，并且最终会被所有的可用的状态机执行。只要leader创建了这个entry并且把它复制到大多数的server上，这个entry就被提交了（比如，图6中索引为7的entry）。这也提交了leader的log中前面的entry，包括被以前的leader创建的entry。5.4节讨论了在leader改变后应用这条规则的细微之处，并且这也显示了对提交的定义是安全的。leader跟踪它知道的提交中的最高索引，包括将来的AppendEntries类型RPC请求（包括心跳）中的索引，这样其他的server最终会发现。只要一个follower学习到一个entry被提交了，follower就会把entry应用到它本地的状态机（依照log中的顺序）。  
&emsp;&emsp;我们设计Raft的日志机制来保持不同server上日志的高度一致性。这不仅仅是让系统的行为更为简单，并且让它更容易预测，但这是一个用来保证安全的重要组件。Raft维护下面的一些特性，一起构成图3中的Log Matching Property：

- 如果不同log中的两个entry拥有相同的索引和term编号，那么它们存储的是相同的命令。
- 如果不同log中的两个entry拥有相同的索引和term编号，那么这个entry前面的log是完全一样的。

&emsp;&emsp;第一个特性来自这样的事实，leader基于给定的log索引和term编号，最多创建一个entry，在log中entry永远不会改变它们自己的位置。第二个特性是通过AppendEntries类型的RPC请求执行的一致性检查来保证的。当发送一个AppendEntries类型的RPC请求时，leader将前一个entry的log编号和term编号包含到请求里面。如果follower在它的log中没有找到相同索引和term编号的entry，那么就拒绝这个新的entry。（译注：即leader中的前一个entry在follower中不存在，就拒绝加入entry）一致性检查用作规划步骤：log的初始的空的状态满足Log Matching Property，并且一致性检查在log扩展的时候可以保持Log Matching Property。结果，不管什么时候，只要AppendEntries类型的RPC请求返回成功，leader都知道follower的log在新entry之前和leader都是一样的。  
&emsp;&emsp;在正常操作期间，leader和follower的log保持一致，所以AppendEntries一致性检查永远不会失败。但是，leader崩溃可能会让log不一致（老的leader可能没有将它所有的log复制到其他server上去）。这些不一致可能会由一系列leader和follower的崩溃组成。图7说明了follower的log可能和新leader不一致的方法。一个follower可能会丢失已经在leader上存在的entry，它可能会有在新的leader不存在的entry。log中丢失和外来的entry可能会导致多个term编号。
![图七](7.png)
> **图7:** 当顶部的leader上台后，在follower的log中，（a-f）场景中的任意一个都有可能出现。每个框都代表一个entry；框里面的数字是term编号。一个follower可能会丢失entry（a-b），可能会有额外的未提交的entry（c-d），或者都有（e-f）。比如，场景f产生的原因可能是这样的，在编号为2的term中，该server是一个leader，添加了一些entry到它的log中，但是在没有提交这些entry之前，server崩溃了；然后这个快速重启，在编号为3的term中继续做leader，然后又加了几个entry在它的log中；在编号为2或3的term中，没有一个entry被提交，然后这个server又挂了，在接下来的几个term中这个server也没有起起来。

&emsp;&emsp;在Raft中，leader通过强制follower复制leader的log来处理非一致性。这意味着follower的log中和leader不一致的地方将会被覆盖。5.4节将展示当和一个或多个限制关联的时候，这是安全的。  
&emsp;&emsp;为了让follower的log和自己一致，leader必须找到两者的log之间最新的一样的位置，然后将这个位置后面所有follower的entry都删掉，并将leader在这个位置之后的entry都发送给follower。所有的这些操作都是由AppendEntries类型的RPC请求在执行一致性检查的时候作出的。leader对所有的follower都维护了一个nextIndex，就是leader将发送给follower的下一个entry的索引。当一个leader上台之后，它将所有的nextIndex初始化为leader的log中的最后一个索引（比如图7中的索引11）。如果一个follower的log和leader不一致，在下一个AppendEntries类型RPC请求时AppendEntries一致性检查将会失败。在被拒绝之后，leader减少nextIndex然后重新发送AppendEntries类型的RPC请求。最终nextIndex会在leader和follower的log一致的地方。当这发生时，AppendEntries类型的RPC请求将会成功，会移除follower中和leader冲突的entry，并将leader中的新的log添加到follower中（如果有的话）。只要AppendEntries类型的RPC请求成功，follower上的log将会和leader一致，并且在这个term剩下的部分也会保持这种状态。  
&emsp;&emsp;如果需要,可以优化协议来减少被拒绝的AppendEntries类型RPC请求的数量。比如,当拒绝一个AppendEntries类型的RPC请求时,follower可以将冲突entry的term编号和它存储这个term的第一个索引包含进来。有了这个信息,leader通过跳过这个term内所有冲突的entry来减少nextIndex;包含冲突entry的AppendEntries类型的RPC请求只需要发送一次,而不是每个entry一个RPC请求。在实践中,我们怀疑这个优化是否是必须的,因为故障并不是经常发生并且好像也不会有很多不一致的entry。  
&emsp;&emsp;有了这个机制,leader在上台后不需要采取任何特殊的措施来恢复log的一致性。它开始正常的操作,AppendEntries一致性检查失败后,日志会自动收敛。leader重来不会覆盖或者删除它自己log里面的entry(图3中leader的Append-Only特性)。
  &emsp;&emsp;log复制机制展示了第2节中描述的合适的一致性特性:只要集群中大多数server存活,Raft就可以接受,复制和应用新的log entry;在正常情况下,只需要一轮RPC请求发送到大多数的server上,新的entry就被复制成功了;一个反应缓慢的follower不会影响性能。  
  ## 5.4 安全性
&emsp;&emsp;前面的小节描述了Raft是如何选举和复制log的。但是,到目前为止,这个机制还不能完全保证每一个状态机按照相同的顺序执行相同的命令。比如,当一个leader提交了一些log entry后,某个follower可能故障了,后面这个follower可能恢复过来然后被选举为leader,那么以前leader提交的entry就被覆盖了;结果是,不同的状态机可能执行不同的命令序列。  
&emsp;&emsp;本节在server被选举为leader的过程中添加了一条限制,进一步完善了Raft算法。这个限制保证了,leader在某个给定的term中,该term的前一个term中的所有entry都已经被提交了(图3中的Leader Completeness特性)。鉴于这个选举限制,我们制定规则对提交进行更为精准的定义。最终,我们为Leader Completeness特性展示了一个证明草图,并且展示了它是如何让复制状态机的行为正确的。  
### 5.4.1 选举限制
&emsp;&emsp;在任何基于leader的一致性算法中,leader最终必须存储所有的提交了的log entry。在某些一致性算法中,比如Viewstamped Replication,即使一个server并不包含所有的提交了的entry它也可以被选举为leader。这些算法包含附加的机制去识别遗失的entry并将它们传输到新的leader,不是在选举过程中就是在选举完后很短的一段时间内。不幸的是,这导致了值得考虑的附加机制和复杂性。Raft使用了简单的方法来保证上一个term中所有提交了的entry,在新leader选举出来的时候,会出现在新leader中,不需要将这些entry传输到新的leader。这意味着log entry的流向只有一个方向:从leader流向follower,并且leader不会覆盖他们自己的log。  
&emsp;&emsp;Raft使用投票过程来阻止一个candidate当选为leader,除非这个candidate的log包含了所有已经提交了的entry。为了当选,一个candidate必须和集群的大多数节点通信,意味着每一个被提交了的entry在这些和candidate通信的server中,至少出现一次。如果candidate的log至少和集群中的其他日志一样新("新"会在下面精确定义),让后它会保持所有的提交了的日志。RequestVote PRC实现了这个限制:这个RPC包含了candidate的log信息,如果投票者的log比candidate更新,那么投票者会拒绝自己的投票。  
&emsp;&emsp;Raft通过比较索引和log中最后一些entry的term编号,来判断哪个log新一点。如果两个log的最后的entry有不同的term编号。那么term编号大的log新一点。如果两个log的term编号一样,log的长度长的更新一点。  
### 5.4.2 从前面的term提交entry
![图8](8.png)
>**图8:**这个时间序列展示了,为什么一个新的leader无法使用上一个term的log entry来判断提交与否。在(a)中S1是leader,在索引为2的位置,leader只复制了一部分log entry。在(b)中,S1崩溃了;S5收到了S3、S4和它自己的投票在编号为3的term中被选为leader,在索引为2的位置,S5已经接受了一个和上一个leader不同的log entry。在(c)中,S5崩溃了,S1重启启动并被选为leader,然后继续复制。在这时,编号为2的term已经被复制到大多数的server上了,但是它没有被提交。如(d)所示,如果S1崩溃了,S5可能被选举为leader(来自S2、S3、S4的投票),S5会用它在编号为3的term里的entry覆盖其他server的log entry。但是, 在(e)中,如果S1在它崩溃前把它当前term上的entry复制到大多数server上,然后这个entry被提交了(S5就不能赢得选举了)。在这时,log中所有前面的entry也被提交了。

&emsp;&emsp;就像5.3节描述的那样,leader知道,如果它当前term中的entry被存储到大多数的server上,这个entry就被提交了。如果一个leader在提交一个entry前宕机了, 将来的leader会继续这个复制过程。但是,leader不能立即判断前一个term里面的entry被存储到大多数server上就被提交了。图8展示了一种场景,所有旧的log entry都被存储到大多数的server上,但是还是被后面的leader覆盖了。  
&emsp;&emsp;为了消除像图8中那样的错误,Raft不使用计算副本数的方式来提交前一个term中的log entry。只有leader当前的term中的log entry使用计算副本数的方式进行提交;只要一个当前term的entry通过这种方式被提交了,基于Log Matching Property那么这个entry前面的entry都被间接提交了。有一些场景下,leader可以可以安全地推断出一个旧的log entry被提交了(比如,如果那个entry被存储在每个server上),但是Raft为了简单采用了更为保守的方法。  
&emsp;&emsp;Raft在提交规则中引入了额外的复杂性,是因为当leader从前面的term复制log entry时,log entry保留了它们以前的term编号。在其他一致性算法中,如果一个新的leader从前面的term中复制entry,它必须使用新的term编号。Raft的方法使调试log entry更为容易,因为log entry在不同的时间和log中保持了相同的term编号。另外,Raft中的新leader比其他算法从前面的term发送更少的log entry(其他算法在提交log entry前,必须发送冗余的log entry来给他们重新编号)。
### 5.4.3 安全争论
&emsp;&emsp;给出了完整的Raft算法,现在我们可以在Leader Completeness Property城里上做更为精确的争论(这个争论是基于安全证明;见9.2节)。我们假设Leader Completeness Property不成立,然后我们证明了矛盾。假设term T的leader(leaderT)从它的term中提交了一个log entry,但是这个log entry没有被leader存储在后面的term中。定义




## 5.5 follower和candidate崩溃
&emsp;&emsp;前面的部分我们集中在leader崩溃的场景。follower和candidate崩溃的场景比leader崩溃更容易处理,对follower和candidate崩溃的处理方式是一样的。如果一个follower或者candidate崩溃了,那么后面发送给它们的RequestVote和AppendEntries请求会失效。Raft通过无穷地尝试的方式来处理这些错误;如果崩溃的server恢复了,那么这些RPC会成功地完成。如果一个server在完成了RPC请求但是还没有响应这个请求时崩溃了,那么在这个server恢复后它还是会收到相同的RPC请求。Raft的RPC请求是幂等的,所以不会有什么影响。比如,如果一个follower接受到了一个AppendEntries RPC请求,但是请求中的log entry已经存在于follower,follower会忽略这个请求。
## 5.6 时间和可用性
&emsp;&emsp;我们对Raft的一个要求是,它的安全性不能依赖于时间:仅仅是因为某些事件发生的快或者慢一点时,系统不能产生错误的结果。但是,可用性(系统及时响应客户的能力)不可避免地依赖于时间。比如,如果消息交换的时间比服务器崩溃然后恢复的的典型时间长,候选人将不会停留足够长的时间来赢得选举;没有一个稳定的leader,Raft就不能工作了。  
&emsp;&emsp;leader选举是Raft中时间很重要的一个方面。只要系统满足下面的时间需求,Raft就可以选举并维持一个稳定的leader:  

```
broadcastTime << electionTimeout << MTBF
```
&emsp;&emsp;在这个不等式中,broadcastTime是一个server并行地发送RPC请求到集群中的其他server并收到响应的平均时间;electionTimeout是5.2节中描述的选举超时时间;MTBF(Mean Time Between Failures)是一个server故障后恢复的平均时间。broadcastTime应该比electionTimeout小一个数量级,以便leader可以在开始选举的时候可靠地发送心跳消息来保持follower;选举超时使用的随机方法,这个不等式也使投票分散基本不可能。electionTimeout比MTBF小一个数量级,这样系统就可以稳定地运行。当leader崩溃时,系统可能在electionTimeout这个时间段内不可用;我们希望这只代表总时间的一小部分。  
&emsp;&emsp;broadcastTime和MTBF是底层系统的属性,但是electionTimeout是我们必须选择的。Raft的RPC需要接受者将消息持久化到稳定存储上,所以broadcastTimeout从0.5ms到20ms不等,这依赖于使用的存储技术。结果是,electionTimeout大概在10ms到500ms之间。典型地,server的MTBF是几个月或者更多,轻松地满足了时间需求。  

# 6 集群成员改变



# 7 日志压缩


# 8 和client交互
&emsp;&emsp;本节描述了client如何和Raft交互,包括client如何找到集群的leader,Raft如果支持线性化语义。这些问题适用于所有基于一致性的系统,并且Raft的解决方案和其他系统类似。  
&emsp;&emsp;Raft的client将它们的所有请求都发送到leader。当一个client启动后,它连接到一个随机选中的server。如果client的第一选择不是leader,那么server会拒绝这个请求,并提供server醉倒的最近当选的leader的信息(AppendEntries请求包含了leader的网络地址)。如果leader崩溃了,client的请求将会超时;client将会继续尝试随机选择server。  
&emsp;&emsp;我们在Raft上的目标是实现线性语义(每个操作似乎只在其调用和响应之间的某一瞬间执行一次)。但是,到现在为止的描述中我们发现Raft可以多次执行一个命令:比如,如果leader在提交了log entry后,在响应client前崩溃了,这个client会向新leader发送同样的命令,导致这个命令被执行两次。解决方法是为client的每个命令分配一个唯一的序列号。然后,状态机跟踪每个client最近处理的命令的序列号,还有响应一起。如果它接受到了一个命令,发现它的序列号已经被处理了,状态机不会重新执行这个命令,然后立即响应client。  
&emsp;&emsp;不写任何东西到log中可以处理只读的操作。但是,没有额外的方法,这将会有返回陈旧数据的风险,因为响应请求的leader可能已经被新的leader取代了,但是它并不知情。线性读必须不能返回过期的数据,Raft需要两个额外的预防措施来保证它并且不使用日志。首先,leader必须有哪些entry被提交了的最少的信息。Leader Completeness Property保证了一个leader拥有所有提交了的entry,但是在term开始时,它可能不知道这些是什么。为了弄明白,它需要在它的term中取提交一个entry。Raft通过让每个leader在每个term开始时,提交一个no-op空的entry到log中来处理这样的情况。其次,leader在处理一个只读请求前(如果一个最近的leader被选举出来了,它的信息可能是过时的)必须检查它是否还是leader。Raft通过让leader和集群中的大多数交换心跳信息然后再响应只读请求来处理这样的问题。或者leader可以依赖心跳机制提供一种新式的出租,但是这样就需要时间来保证安全性(它假定有界时钟偏移)。


# 9 实现和评估



# 10 相关的工作



# TODO
* 这篇论文是extended版本，第一版也需要看。
