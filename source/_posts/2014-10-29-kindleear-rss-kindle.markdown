---
layout: post
title: "使用KindleEar免费把RSS订阅定时推送到Kindle"
date: 2014-10-29 09:58:44 +0800
comments: true
categories: [KindleEar, RSS, Kindle]

---

Kindle是亚马逊推出的电纸书阅读器，采用电子墨水屏幕，其优点是非常省电且显示效果非常接近纸，可以在太阳底下阅读。前段时间入手Kindle PaperWhite后，就一直琢磨着如何把原来用Rss订阅的内容推送到kindle上来看。

###可选择的方式
目前通过搜集的可行的方式大概有三种
####Rss推送网站
现在有不少把RSS订阅推送到Kindle的服务，比较流行的有以下三个：

1.ReadCola

[ReadCola](http://readcola.com/)可以生成期刊格式，便于翻阅，媲美纸质杂志；针对阅读器对图片优化，内容重新排版，提升阅读体验。支持定时推送，相比其它推送站最大的特色就是可以推送自定义的Rss并且带有图片，缺点是只支持16个订阅。

2.狗耳朵

[狗耳朵](http://www.mydogear.com/)内置资源非常多，包括RSS和微信公众号，支持定时推送。缺点是免费用户无法推送图片，不能添加自定义Rss，付费用户是可以的，不过费用也不算贵，现在是3元每月或30元每年。

3.Kindle4Rss

[Kindle4Rss](http://kindle4rss.com/)也分免费和付费用户，二则的区别如下:
{% img center http://windstill.qiniudn.com/QQ20141029-1.png %}

<!-- more -->

####使用IFTTT创建一个Recipe
这种方式要完成一个推送动作，需要一个订阅网站，一个稍后阅读网站，一个负责检查更新并保存到稍后阅读的触发器网站。大概过程是：触发器网站监控订阅的Rss，如果Rss有更新，就保存到稍后阅读网站中，稍后阅读网站会在设定的时间将更新的内容推送到kindle。

这里三个网站对应的分别是：

>
触发器网站 [IFTTT](https://ifttt.com/)  
订阅网站 [Feedly](http://feedly.com/)  
稍后阅读网站 [Readability](http://www.readability.com/)

首先要注册这三个网站的帐号，然后在Feedly中为想要推送的Rss新建一个分类并添加Rss订阅（若所有网站都推送的话也可不建）；接着在IFTTT中创建一个新的Recipe，按提示来，this部分点击Feedly图标，然后根据需要选择New article saved for later或New article from category，that部分选择Readability，在Read later URL部分根据需要选上ArticleTitle、ArticleContent等等，最后再到Readability网站Kindle Setting中设置下推送的模式。

####使用开源应用KindleEar
这个是我目前正在使用的方法，也是我下面要具体说明的。具有免费，自定义Rss，定时推送，带图片的mobi格式，排版精美等一系列优点。

###安装KindleEar并部署到GAE
####KindleEar简介
KindleEar是一个运行在Google App Engine(GAE)上的Kindle个人推送服务器，可以生成排版精美的杂志模式MOBI格式自动每天推送至您的kindle。

此网站应用目前的功能有：

>
1.支持类似calibre的recipe格式的自定义RSS收集，需要写代码，需要有一点点python基础  
2.自定义RSS，不需要python基础，直接输入RSS链接和标题即可自动推送  
3.多账号管理，也就是支持多kindle  
4.带图的杂志格式MOBI  
5.自动每天定时推送  
6.强大而且方便的邮件中转服务  

####安装及部署
以下都是基于Mac OS X系统上操作的步骤：

* 申请GAE账号并创建一个application。 https://appengine.google.com/

* 下载GAE SDK。 https://developers.google.com/appengine/downloads?hl=zh-CN

* 安装Python 2.7（Mac系统默认已安装）

* 下载 [KindleEar源码](https://github.com/cdhigh/KindleEar)，放到一个特定的目录。

* 修改app.yaml 和 module-worker.yaml的第一行：将kindleear修改为你申请的application名字

{% codeblock lang:ruby app.yaml %}
application: app_name
version: 1
runtime: python27
api_version: 1
threadsafe: true
instance_class: F1
{% endcodeblock %}

{% codeblock lang:ruby module-worker.yaml %}
application: app_name
module: worker
version: 1
runtime: python27
api_version: 1
threadsafe: true
instance_class: B2
{% endcodeblock %}

* 修改config.py中的这几个变量 SRC_EMAIL ：你申请GAE账号时的GMAIL邮箱 DOMAIN ：你申请的应用的域名

{% codeblock lang:python config.py %}
#!/usr/bin/env python
# -*- coding:utf-8 -*-
"""Configures for KindleEar, the First two variable is must to modify.
KindleEar配置文件，请务必修改开始两个配置（如果使用uploader，则uploader自动帮你修改）
"""

SRC_EMAIL = "account@gmail.com"  #Your gmail account for sending mail to Kindle
DOMAIN = "https://appname.appspot.com" #Your domain of app
{% endcodeblock %}

* 转到KindleEar目录，执行命令：

```bash
appcfg.py update app.yaml module-worker.yaml
```

依次输入Google邮箱帐号和密码（如果Google帐号开启了两步验证，请使用应用专用密码）。

等结束后就可以打开域名： app_name.appspot.com (app_name是你申请的application名字) 如果出现如下界面，就表示安装部署成功了。

{% img center http://windstill.qiniudn.com/QQ20141030-1.png %}

初始用户为admin，密码为admin，建议登陆后及时修改密码。

####设置推送

{% img center http://windstill.qiniudn.com/QQ20141030-2.png %}

登录后，可以根据自己的需要进行设置了：

1. 我的订阅   
可以在这里添加自定义的RSS地址。也可以在下方预置的一些订阅里选择自己感兴趣的。
2. 设置   
这里是推送的详细设置，在这里填写你要推送的“Kindle E-mail”，选择投递日，投递时间。建议勾选“多本书籍合并投递为一本”、“使能自动定时投递”、“自动定时投递自定义RSS”。同时还可以在“书籍标题”项填写显示在Kindle里的个性名称。当所有设置完后还可以点击“现在投递”测试一下。
3. 投递日志   
每次投递的记录。
4. 账户管理   
可以更改密码，添加多用户等。
5. 高级设置   
有邮件白名单、归档和分享、URL过滤、导入订阅列表设置。

至此，一个完整的RSS订阅定时推送服务完成了，如果你想为你的KindleEar配置自定域名可以继续往下看。

####绑定自定义域名
如果你还没有自己的域名，得先去域名注册商（比如国内的[万网](http://www.net.cn/)，国外的[Godaddy](http://www.godaddy.com/)）那里购买一个，然后就可以按照以下步骤设置你的App Engine application了。

_1._  登录到 [Google Developers Console](https://console.developers.google.com/?_ga=1.117873033.1850141824.1414462870)，然后的选择你的project（即与上文app_name同名的project）点击进入到dashboard页面。在左侧目录树中一次选择 **Compute > App Engine > Settings** 。然后右侧的设置页面的顶部可以看到两个标签页：**Application settings 和 Custom domains**。选择 **Custom domains**，就会看到带有如下三个选项的页面：

{% img center http://windstill.qiniudn.com/domain_new_1.png %}

_2._  在图中的第一个步骤中填入你的域名，然后点击 **Verify** 按钮，会跳转到`网站管理员中心`页面，在这里你需要证明你拥有这个域名。选择好域名提供商后，页面如下：

{% img center http://windstill.qiniudn.com/QQ20141030-4.png %}

然后点击 **添加 TXT 记录**，然后就会出现对应域名提供商添加TXT记录的教程。如果你的域名已经交由第三方DNS管理，例如DNSPOD，就需要到DNSPOD中为域名添加一个TXT记录了。

_3._  添加完TXT记录后，回到`网站管理员中心`页面，点击验证按钮。

_4._  验证成功后，到 **Custom domains** 页面的第二个步骤中，添加一个域名或者二级域名。如果要添加一个顶级域名，请选择第一个选项，然后在下拉框中选择你想要的域名：

{% img center http://windstill.qiniudn.com/domain_new_2.png %}

第三步骤中会出现与之相关的说明。这里列出了你需要在域名注册商或者DNS解析服务商那里创建的指向Google服务器DNS记录。然后到你的域名注册商或者DNS解析服务商添加上述记录。添加完后回到刚才的 **Custom domains** 页面，点击 **Add** 按钮。这样域名和应用关联以后，就会出现在页面底部的 **Custom domain names** 列表中。

{% img center http://windstill.qiniudn.com/domain_list.png %}

_5._ 如果要添加二级域名，在 **Custom domains** 页面的第二个步骤中，点击第二个选项。在下拉列表中选择想要的域名，在文本框中输入二级域名。第三步骤中会出现与之相关的说明：需要在域名注册商或者DNS解析服务商创建一条 `CNAME` 记录，指向 `ghs.googlehosted.com` 子域名。添加完后回到刚才的 **Custom domains** 页面，点击 **Add** 按钮。这样域名和应用关联以后，就会出现在页面底部的 **Custom domain names** 列表中。

恭喜，你可以用你自己的域名访问了。

###参考文章
* [Rss订阅推送到Kindle的几种方法](http://xuhehuan.com/1368.html)
* [KindleEar：免费把RSS订阅定时推送到Kindle](http://runbing.me/archives/kindleear-rss-kindle.html)
* [Using a Custom Domain](https://cloud.google.com/appengine/docs/domain)