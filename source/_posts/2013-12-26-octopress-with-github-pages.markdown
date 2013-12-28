---
layout: post
title: "用Octopress和Github-pages搭建博客"
date: 2013-12-26 21:07:44 +0800
comments: true
categories: "Octopress"
---

###1 相关介绍
####Octopress
[`Octopress`](http://octopress.org/)是利用[`Jekyll`](http://github.com/mojombo/jekyll)博客引擎开发的一个博客系统，生成的静态页面能够很好的在github page上展现。如果用[`Jekyll`](http://github.com/mojombo/jekyll)写博客的话，得要自己写HTML模板、CSS、Javascript，并且要自己做配置。但是如果用[`Octopress`](http://octopress.org/)，这些都帮你做好了，只要[`clone or fork Octopress`](https://github.com/imathis/octopress)，安装些依赖的库和主题就行了。

<!-- more -->

####Github pages
[`GitHub`](https://github.com/)是全球最大的社交编程及代码托管网站，GitHub可以托管、分享各种代码库或项目，并提供一个web界面。 
  
[`Github Pages`](http://pages.github.com/)就是为你或你的项目提供一个静态的网站。原理就是把你的一个特殊的repository (yourname.github.io)，或者某个repository特殊分支(gh-pages)的静态文件以网页形式展现出来，前者一般作为个人主页，后面一般作为项目的主页，这两种情况后面会说到。  

这种方案优势：1.搭建、发布简单；2.免费、无限流量，3.绑定自己域名

###2 安装Octopress
####安装前准备
* [安装 Git](http://git-scm.com/)
* 安装 Ruby 1.9.3  


可以通过安装RVM，来安装、管理ruby版本 
 
```
curl -L https://get.rvm.io | bash -s stable --ruby
```

接着是安装Ruby 1.9.3   

```
rvm install 1.9.3
rvm use 1.9.3
```

####安装Octopress
克隆Octopress的代码库  

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```  

安装依赖库

```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
```

安装Octopress默认主题

```
rake install
```

###3 配置Octopress
Octopress的作者已经让配置尽量简化了，一般情况只需要配置`_config.yml`和`Rakefile`文件即可。下面是Octopress的配置文件列表。

```
    _config.yml       # 主要配置文件 (Jekyll的设置)
    Rakefile          # 部署相关配置
    config.rb         # Compass config
    config.ru         # Rack config
```

`_config.yml`文件中主要有三大配置段：`Main Configs`、`Jekyll & Plugins`和`3rd Party Settings`。一般，该文件中其中url是必须要填写的，这里的url是在github上创建的一个仓库地址，具体请看第四步中创建的地址。另外再修改一下title、subtitle和author，根据需求，再开启一些第三方组件服务。

参见 [Configuring Octopress](http://octopress.org/docs/configuring/)

###4 部署到Github Pages
####Github User/Organization pages
如果想放到`http://username.github.io`域名下，就用这种方式。以前是`.com`域名，现在改成`.io`了。不过可以配置自己的域名，这个后面会说到。

首先需要在GitHub上创建一个[新的仓库](https://github.com/repositories/new)，命名为`username.github.io`，`username`就是你在Github的用户名。等后面配置好部署到Github上以后，就可以通过`http://username.github.io`访问了。这种方式需要一个master分支来存放生成的博客内容（_deploy目录），source分支存放整个源码。用octopress的一个`rake任务`可以自动完成以上配置。

```
rake setup_github_pages
```

这条rake任务会让你输入仓库的地址，SSH和HTTPS的URL都可以。执行后，会做以下事情：  
1. 存储Github Pages的仓库地址  
2. 将octopress的远程仓库`origin`重命名为`octopress`  
3. 将Github Pages仓库作为默认的远程origin  
4. 将当前分支从`master`切换到`source`  
5. 根据仓库地址配置博客的URL  
6. 在部署目录`_deploy`中创建`master`分支  

然后执行：

```
rake generate
rake deploy
```

这两条命令会生成博客系统的主要文件并拷贝到`_deploy/`目录，然后添加到git版本控制，提交并推送到远程主分支。过一会就可以访问`http://username.github.com`了。

然后把源码提交到`source`分支

```
git add .
git commit -m 'your message'
git push origin source
```

####Github Project pages (gh-pages分支)

Github的Project Pages服务可以为项目提供网站页面。它会在项目仓库中寻找`gh-pages`分支，然后通过`http://username.github.io/project`访问项目网站。

执行Octopress的rake任务命令

```
rake setup_github_pages
```

然后执行

```
rake generate
rake deploy
```

这几条命令的执行结果基本与上面的类似。

参见 [Deploying to Github Pages](http://octopress.org/docs/deploying/github/)

####自定义域名
首先在`source/`目录下新建一个名为`CNAME`的文件，并填入你的域名，比如`example.com`或`blog.example.com`。

然后到你的域名登记或DNS管理网站添加一条域名记录  

* 如果是顶级域名，比如`example.com`，那么需要创建一条A记录，并指向`204.232.175.78`。  
* 如果是二级域名，比如`blog.example.com`，那么创建一条CNAME记录，并指向`username.github.io`。

然后执行

```
rake generate
rake deploy
```

大概过个10分钟，就可以用新的域名访问了。

###5 开始写博客
Octopress提供了一些rake任务来创建博文和页面。博文必须存储在source/_posts目录下，其中预置了一些元数据，并且以Jekyll的命名规范命名：YYYY-MM-DD-post-title.markdown。博文的名字会被当做url的一部分，而其中的日期用于对博文的区分和排序。

####创建博文
通过Octopress提供的rake任务可以正确的按照命名规范创建博文，并且在博文中会预知一些常用的yaml元数据。

```
rake new_post["title"]
```

其中title为博文的文件名，创建出来的文件默认是markdown格式。上面的命令会创建出这样一个文件：source/_posts/2013-08-03-title.markdown。用文本编辑器打开这个文件，可以看到里面有一块yaml元数据(Jekyll博客引擎会根据这些内容处理博文和页面)：

```
---
layout: post
title: "Zombie Ninjas Attack: A survivor's retrospective"
date: 2011-07-03 5:59
comments: true
external-url:
categories:
---
```

####设置博文
在以上的yaml元数据块里可以：  

* 关闭评论  
* 给博文添加分类  
* 如果是多人博客，可以添加`author: Your Name`到元数据  
* 如果只是想作为草稿，不想在发布后公开，那么可以添加`published: false`  
* 如果是想发布个外链博文，只要为博文添加个`external-url`  

你还可以想下面这样添加一个或多个分类：

```
# 一个分类
categories: Sass
 
# 多个分类，示例1
categories: [CSS3, Sass, Media Queries]
 
# 多个分类，示例2
categories:
- CSS3
- Sass
- Media Queries
```


####撰写博文
在元数据块下面就可以撰写博文正文了。

如果想在首页中只显示博文的部分正文，可以在博文中写入`<!--more-->`，这样首页的博文就只显示`<!--more-->`标识之前的内容，并在每篇博文底下加一个`Read on`超链接。  

####生成并预览
```
rake generate   # 生成post和page到public目录
rake preview    # 启动服务并预览 http://localhost:4000
```
预览无勿的话，就可以部署了  

```
rake deploy
```

参见 [Start blogging with Octopress](http://octopress.org/docs/blogging)

###参考文章

* [利用Octopress搭建一个Github博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)
* [象写程序一样写博客：搭建基于github的博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)


