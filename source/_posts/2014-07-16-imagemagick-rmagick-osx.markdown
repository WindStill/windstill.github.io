---
layout: post
title: "解决ImageMagick和RMagick在OS X下使用出现的问题"
date: 2014-07-16 10:50:42 +0800
comments: true
categories: [ImageMagick, RMagick, OS X, Rails]

---

每次安装 `RMagick` 这个gem都会遇到问题，并且每次的问题还都不一样，有的时候是 `ImageMagick` 的问题，有时候是 `RMagick` 不能安装，这次是安装了以后，启动Rails的时候，报错说是 `libMagickCore-6.Q16.2.dylib` 版本不对。错误详情如下：

<!-- more -->

```
/Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/activesupport-3.2.12/lib/active_support/dependencies.rb:251:in `require': dlopen(/Users/windstill/.rvm/gems/ruby-1.9.3-p194/extensions/x86_64-darwin-13/1.9.1/rmagick-2.13.2/RMagick2.bundle, 9): Library not loaded: /usr/local/lib/libltdl.7.dylib (LoadError)
  Referenced from: /usr/local/lib/libMagickCore-6.Q16.2.dylib
  Reason: Incompatible library version: libMagickCore-6.Q16.2.dylib requires version 11.0.0 or later, but libltdl.7.dylib provides version 10.0.0 - /Users/windstill/.rvm/gems/ruby-1.9.3-p194/extensions/x86_64-darwin-13/1.9.1/rmagick-2.13.2/RMagick2.bundle
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/activesupport-3.2.12/lib/active_support/dependencies.rb:251:in `block in require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/activesupport-3.2.12/lib/active_support/dependencies.rb:236:in `load_dependency'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/activesupport-3.2.12/lib/active_support/dependencies.rb:251:in `require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/rmagick-2.13.2/lib/rmagick.rb:11:in `<top (required)>'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler/runtime.rb:76:in `require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler/runtime.rb:76:in `block (2 levels) in require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler/runtime.rb:72:in `each'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler/runtime.rb:72:in `block in require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler/runtime.rb:61:in `each'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler/runtime.rb:61:in `require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194@global/gems/bundler-1.6.3/lib/bundler.rb:132:in `require'
	from /Users/windstill/dev/rails/movie/config/application.rb:13:in `<top (required)>'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/railties-3.2.12/lib/rails/commands.rb:39:in `require'
	from /Users/windstill/.rvm/gems/ruby-1.9.3-p194/gems/railties-3.2.12/lib/rails/commands.rb:39:in `<top (required)>'
	from script/rails:6:in `require'
	from script/rails:6:in `<main>'
```

使用 `brew unlink libtool && brew link libtool` 命令解决问题。可能会出现如下问题：  

```  
Linking /usr/local/Cellar/libtool/2.4.2...
Error: Could not symlink include/ltdl.h
Target /usr/local/include/ltdl.h
already exists. You may want to remove it:
  rm /usr/local/include/ltdl.h

To force the link and overwrite all conflicting files:
  brew link --overwrite libtool

To list all files that would be deleted:
  brew link --overwrite --dry-run libtool
```

按照提示输入 `brew link --overwrite libtool`。

###参考文章
* [ImageMagick and OS X Lion trouble](http://stackoverflow.com/questions/7412208/imagemagick-and-os-x-lion-trouble)
