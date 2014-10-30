---
layout: post
title: "Mac Mavericks系统因为‘来自身份不明开发者’不能打开软件的解决方法"
date: 2014-01-03 17:29:52 +0800
comments: true
categories: [OS X]
---

自从 `10.7 Lion` 升级到新系统`10.9 Mavericks` 后，一些小问题也就跟着来了。听说苹果是从 `10.8 Mountain Lion` （这个版本系统我没装过）系统开始启用了新的安全机制，默认只信任 App Store 下载的软件和拥有开发者 ID 签名的应用程序。也就是说新系统默认只能安装正规渠道（App Store 或被认可的开发人员）的软件，否则安装的时候就会弹出下图所示警告框：`打不开“xxx”，因为它来自身份不明的开发者`，如下图所示：

{% img center http://blog-picture.qiniudn.com/94C990E2-7FC8-4117-B9EC-F17C338A2F96.png %}

<!-- more -->

这个问题解决方法就是到 `系统偏好设置` 里设置一下就行了，步骤如下：

* 点击 Mac 屏幕左上角的苹果标志，在下拉菜单里选择 `系统偏好设置…`，或者到 `应用程序` 里找到 `系统偏好设置` 应用。在弹出的设置面板里，点击第一行的小房子图标 `安全性与隐私` 。

{% img center http://blog-picture.qiniudn.com/02FAD525-62C0-450B-8C99-CBC9DD6ED1F5.png %}

* 在 `安全性与隐私` 设置面板里选择 `通用` 标签。可以看到上面截图里中间红框标出的几个选项了，把“允许从一下位置下载的应用程序”从默认的 `Mac App Store 和被认可的开发者` 改选为 `任何来源` 就行了。但按钮是灰的不能选，需要点击左下角红框内的锁按钮解锁。

{% img center http://blog-picture.qiniudn.com/4302E326-1889-4017-93F2-4C5ED6932168.png %}

* 系统会要求你输一下密码确认，解锁后就可以选择 `任何来源` 了，然后会如下图所示：

{% img center http://blog-picture.qiniudn.com/4E81CF94-FAC6-43C5-9707-9CD3F9EA17EA.png %}

* 点击 `允许来自任何来源` 按钮就 OK 了，这样被警告的应用程序就能正常安装、打开了。