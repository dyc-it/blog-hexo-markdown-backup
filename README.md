# blog-hexo-markdown备份



## 新环境搭建

* 安装node.js及相关插件

```
npm install -g hexo-cli
cd ~/Documents
mkdir blogs
cd blogs
hexo init
npm install
npm install hexo-deployer-git --save
```


* 编辑_config.yml文件    

```    
new_post_name: :year-:month-:day-:title.md  
    
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
type: git
repo: git@github.com:dyc-it/dyc-it.github.io.git
branch: master
```


## 博客操作

* 新建文章  
	hexo new "article_name"
* 生成静态页面  
	hexo g
* 启动本地服务器进行查看  
	hexo s
* 将页面部署到github
	hexo g --deploy


## ref
* [hexo官网](https://hexo.io)  
* [Mac版Markdown编辑器MacDown下载](http://macdown.uranusjr.com/)














































