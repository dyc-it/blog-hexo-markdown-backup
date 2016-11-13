# blog-hexo-markdown备份

[博客地址](https://dyc-it.github.io/)

## 新环境搭建

* 配置brew为国内源

```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile

```

* 安装nodejs

```
brew install npm
```

如果node启动失败，执行sudo chmod 777 /usr/local/share/systemtap/tapset

* 配置nodejs使用国内源  
vi ~/.npmrc

```
registry =https://registry.npm.taobao.org

```


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

* 第一次配置备份markdown到github   

```
cd /Users/dyc/Documents/blogs/source/_posts
git init .
git remote add origin git@github.com:dyc-it/blog-hexo-markdown-backup.git
git add .
git commit -a -m "init commit"
git push --set-upstream origin master
```

* 在新环境中配置备份markdown到github  

```
cd /Users/dyc/Documents/blogs/source/_posts
git init .
git clone git@github.com:dyc-it/blog-hexo-markdown-backup.git
mv blog-hexo-markdown-backup/* blog-hexo-markdown-backup/.git* .
rm -rf blog-hexo-markdown-backup/
```
## 博客操作

* 新建文章  
	hexo new "article_name"
* 生成静态页面  
	hexo g
* 清空缓存
	hexo clean
* 启动本地服务器进行查看  
	hexo s
* 将页面部署到github
	hexo g --deploy



## ref
* [hexo官网](https://hexo.io)  
* [Mac版Markdown编辑器MacDown下载](http://macdown.uranusjr.com/)


