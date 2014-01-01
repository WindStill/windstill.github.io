---
layout: post
title: "Mac OSX 10.9 Mavericks系统安装MacVim"
date: 2013-12-29 18:26:08 +0800
comments: true
categories: [MacVim, Mavericks]
---

今天下午把Mac系统从Lion直接升级到了Mavericks，增加了不少新的功能和体验，具体参考官方说明 [OS X Mavericks的全新功能](https://help.apple.com/osx-mavericks/whats-new-from-lion)。这次升级基本保持了原有的数据和配置，但是个别应用会因为系统升级瘦到影响，比如这次所说的`MacVim`应用。

<!-- more -->

###问题及解决
升级完系统后，终端输入`mvim .`会出现以下错误信息：

```
dyld: Library not loaded: /System/Library/Perl/lib/5.10/libperl.dylib
Referenced from: /Applications/MacVim.app/Contents/MacOS/Vim
Reason: image not found
Trace/BPT trap: 5
```

原因是Mac X Mavericks 只有5.12 、5.16 已经没有了5.10。解决办法就是下载最新的Mavericks版本的MacVim [MacVim-snapshot-72-Mavericks.tbz](https://github.com/b4winckler/macvim/releases/download/snapshot-72/MacVim-snapshot-72-Mavericks.tbz)。

其他OSX系统版本可以到 https://github.com/b4winckler/macvim/releases 下载。

下载解压后有三个文件`MacVim`、`mvim`、`reader.txt`，把`MacVim`放到你的应用程序也就是`/Applications`目录下，把`mvim`拷贝到`/usr/bin/`目录下。这样`MacVim`就可以正常使用了：终端下使用`mvim`或直接打开MacVim应用。

###安装插件(plugin)
vim 有好多方便、实用的插件，作为一个普通开发人员，经常使用的就有一二十个，若一个一个去安装确实有点麻烦。

[`Janus`](https://github.com/carlhuda/janus)可以为我们解决这个烦恼，它是一个为`Vim`、`Gvim`、`MacVim`准备的插件和映射的集合。Janus用一些最流行的插件和最常用的映射组合成一个最小型的开发环境。参见 [Janus的Github页面](https://github.com/carlhuda/janus)。

1、安装


```
$ curl -Lo- https://bit.ly/janus-bootstrap | bash
```

2、自定义  

可以在`~/.vimrc.before`中做一些 *leader* 设置，其他设置可以放在 `~/.vimrc.after`里。

如果想安装其他的插件，就要创建一个`~/.janus`目录存放插件。也可以使用`git clone`，例如：

```
$ cd ~/.janus
$ git clone https://github.com/nathanaelkane/vim-indent-guides.git
```

这是一个缩进对齐插件（效果参见配图中明暗相间的竖条），需要在下面所说的`~/.vimrc.before`文件中添加：

```
autocm vimenter * :IndentGuidesEnable
let g:indent_guides_guide_size = 1
```

###安装主题(theme)
我是做`Rails`开发的，之前一直用的`Textmate`编辑器，比较喜欢之前的配色方案。有两款合适的配色方案：[Railscasts.vim](https://github.com/ryanb/dotfiles/blob/master/vim/colors/railscasts.vim) 和 [Sunburst.vim](https://github.com/gigamo/sunburst.vim/blob/master/.vim/colors/sunburst.vim) 。

在`~/.vim`目录下新建`colors/`目录，将`Railscast.vim`或`Sunburst.vim`放到此目录中，然后在`~/.vimrc.before`文件中添加启用此主题的设置项：

```
colorscheme Sunburst 
```

其他配色方案请参考 [10 个你值得拥有的 Vim 配色方案](http://www.oschina.net/news/32306/10-vim-color-schemes-you-need-to-own) ，设置与上面一样。

###其他设置
一般情况下Janus安装插件的时候已经做了许多默认的设置，下面是根据我自己的需求做的设置`~/.vimrc.before`：

```
" 设置主题（配色方案）
colorscheme Sunburst
" 设置字体字号
set guifont=Menlo:h16
" 启用缩进高亮对齐（下图中明暗相间的竖条） 
autocm vimenter * :IndentGuidesEnable
" 设置缩进距离单位
let g:indent_guides_guide_size = 1
" 设置mapleader键位
let mapleader = ","
" 设置文件打开时代码段默认为折叠状态
set foldmethod=indent
```

###示例配图
![example](/images/post_img/A84412D6-BA8F-49F3-B48A-1A4AB8F67E48.png "示例配图")

###参考文章
* [MacVim for OSX 10.9 Mavericks](http://kodira.de/2013/10/macvim-osx-10-9-mavericks/)
* [mac osx 10.9安装配置macvim](http://blog.csdn.net/wildcatlele/article/details/16330009)

