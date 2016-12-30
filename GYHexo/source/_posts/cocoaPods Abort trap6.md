---
title: Abort trap: 6
categories: cocoaPods
tags: [cocoaPods] 
description: 在mac系统升级到了10.12.2之后，使用cocoaPods发生生了Abort trap: 6问题，本文记录了解决方案
---
# Abort trap: 6

## 使用Cocoapods提示错误 Abort trap: 6 解决方法

   > 在升级了mac Os版本10.12.2之后，今天测试一个类似于[JsonModel](https://github.com/jsonmodel/jsonmodel)和[MJExtesion](https://github.com/CoderMJLee/MJExtension)功能的一个Swift的Json和Model之间的转化-->[ObjectMapper](https://github.com/Hearst-DD/ObjectMapper)
   
在我pod install时出现了如下:

```
Analyzing dependencies
Downloading dependencies
Installing ObjectMapper (2.2.1)
Generating Pods project
Abort trap: 6//中止...

```

刚开始以为是Xcode版本造成的，于是就用Xcode 7.3重新创建了一个项目，发生了类似的错误。于是开始着手解决

查看了网上的一系列解决方案,综合了几个得以解决cocoaPods的问题

### 删除所有版本

```
gem uninstall cocoapods
gem uninstall cocoapods-core
gem uninstall cocoapods-deintegrate
gem uninstall cocoapods-downloader
gem uninstall cocoapods-plugins
gem uninstall cocoapods-search
gem uninstall cocoapods-stats
gem uninstall cocoapods-try
gem uninstall cocoapods-trunk
```

### 安装新版本

```
gem install cocoapods --pre
```

发生如下错误
```
ERROR:  While executing gem ... (Errno::EACCES)
    Permission denied @ rb_sysopen - /usr/local/lib/ruby/gems/2.3.0/gems/xcodeproj-1.4.2/LICENSE
```

最后替换掉了淘宝的镜像源为

```
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
```

查看当前镜像源

```
gem sources -l
```

已成功替换

```
*** CURRENT SOURCES ***

https://ruby.taobao.org/
https://gems.ruby-china.org/

```

最后又执行了这个命令

```
 gem install rails
```

此时暴出错误

```
Fetching: rack-2.0.1.gem (100%)
Successfully installed rack-2.0.1
Fetching: concurrent-ruby-1.0.4.gem (100%)
Successfully installed concurrent-ruby-1.0.4
Fetching: sprockets-3.7.1.gem (100%)
Successfully installed sprockets-3.7.1
Fetching: activesupport-5.0.1.gem (100%)
Successfully installed activesupport-5.0.1
Fetching: mini_portile2-2.1.0.gem (100%)
Successfully installed mini_portile2-2.1.0
Fetching: nokogiri-1.7.0.gem (100%)
Building native extensions.  This could take a while...
ERROR:  While executing gem ... (Errno::EACCES)
    Permission denied @ dir_s_mkdir - /usr/local/lib/ruby/gems/2.3.0/extensions/x86_64-darwin-16
```

此时

```
gem install cocoapods --pre
```

```
ERROR:  While executing gem ... (Errno::EACCES)
    Permission denied @ rb_sysopen - /usr/local/lib/ruby/gems/2.3.0/gems/xcodeproj-1.4.2/LICENSE

```

最后利用管理员权限

```
 sudo gem uninstall cocoapods
```

在执行安装

```
sudo gem install cocoapods --pre
```

输出

```
Successfully installed xcodeproj-1.4.2
Fetching: ruby-macho-0.2.6.gem (100%)
Successfully installed ruby-macho-0.2.6
Fetching: nap-1.1.0.gem (100%)
Successfully installed nap-1.1.0
Fetching: molinillo-0.5.4.gem (100%)
Successfully installed molinillo-0.5.4
Fetching: gh_inspector-1.0.2.gem (100%)
Successfully installed gh_inspector-1.0.2
Fetching: fourflusher-2.0.1.gem (100%)
Successfully installed fourflusher-2.0.1
Fetching: escape-0.0.4.gem (100%)
Successfully installed escape-0.0.4
Fetching: cocoapods-try-1.1.0.gem (100%)
Successfully installed cocoapods-try-1.1.0
Fetching: netrc-0.7.8.gem (100%)
Successfully installed netrc-0.7.8
Fetching: cocoapods-trunk-1.1.2.gem (100%)
Successfully installed cocoapods-trunk-1.1.2
Fetching: cocoapods-stats-1.0.0.gem (100%)
Successfully installed cocoapods-stats-1.0.0
Fetching: cocoapods-search-1.0.0.gem (100%)
Successfully installed cocoapods-search-1.0.0
Fetching: cocoapods-plugins-1.0.0.gem (100%)
Successfully installed cocoapods-plugins-1.0.0
Fetching: cocoapods-downloader-1.1.3.gem (100%)
Successfully installed cocoapods-downloader-1.1.3
Fetching: cocoapods-deintegrate-1.0.1.gem (100%)
Successfully installed cocoapods-deintegrate-1.0.1
Fetching: fuzzy_match-2.0.4.gem (100%)
Successfully installed fuzzy_match-2.0.4
Fetching: cocoapods-core-1.2.0.beta.3.gem (100%)
Successfully installed cocoapods-core-1.2.0.beta.3
Fetching: cocoapods-1.2.0.beta.3.gem (100%)
Successfully installed cocoapods-1.2.0.beta.3
Parsing documentation for xcodeproj-1.4.2
Installing ri documentation for xcodeproj-1.4.2
Parsing documentation for ruby-macho-0.2.6
Installing ri documentation for ruby-macho-0.2.6
Parsing documentation for nap-1.1.0
Installing ri documentation for nap-1.1.0
Parsing documentation for molinillo-0.5.4
Installing ri documentation for molinillo-0.5.4
Parsing documentation for gh_inspector-1.0.2
Installing ri documentation for gh_inspector-1.0.2
Parsing documentation for fourflusher-2.0.1
Installing ri documentation for fourflusher-2.0.1
Parsing documentation for escape-0.0.4
Installing ri documentation for escape-0.0.4
Parsing documentation for cocoapods-try-1.1.0
Installing ri documentation for cocoapods-try-1.1.0
Parsing documentation for netrc-0.7.8
Installing ri documentation for netrc-0.7.8
Parsing documentation for cocoapods-trunk-1.1.2
Installing ri documentation for cocoapods-trunk-1.1.2
Parsing documentation for cocoapods-stats-1.0.0
Installing ri documentation for cocoapods-stats-1.0.0
Parsing documentation for cocoapods-search-1.0.0
Installing ri documentation for cocoapods-search-1.0.0
Parsing documentation for cocoapods-plugins-1.0.0
Installing ri documentation for cocoapods-plugins-1.0.0
Parsing documentation for cocoapods-downloader-1.1.3
Installing ri documentation for cocoapods-downloader-1.1.3
Parsing documentation for cocoapods-deintegrate-1.0.1
Installing ri documentation for cocoapods-deintegrate-1.0.1
Parsing documentation for fuzzy_match-2.0.4
Installing ri documentation for fuzzy_match-2.0.4
Parsing documentation for cocoapods-core-1.2.0.beta.3
Installing ri documentation for cocoapods-core-1.2.0.beta.3
Parsing documentation for cocoapods-1.2.0.beta.3
Installing ri documentation for cocoapods-1.2.0.beta.3
Done installing documentation for xcodeproj, ruby-macho, nap, molinillo, gh_inspector, fourflusher, escape, cocoapods-try, netrc, cocoapods-trunk, cocoapods-stats, cocoapods-search, cocoapods-plugins, cocoapods-downloader, cocoapods-deintegrate, fuzzy_match, cocoapods-core, cocoapods after 7 seconds
18 gems installed
```

此时再去执行

```
pod install
```
就会成功输出

```
Installing ObjectMapper (2.2.1)
Generating Pods project
Integrating client project

[!] Please close any current Xcode sessions and use `Demo.xcworkspace` for this project from now on.
Sending stats
Pod installation complete! There is 1 dependency from the Podfile and 1 total pod installed.
```

此时在查看Demo,发现已经成功显示出来了.xcworkspace以及相关组件。

## 总结:

- 或许你只需要执行这两步即可

 ```
  sudo gem uninstall cocoapods
  sudo gem install cocoapods --pre
 ```
 
- 也可能你需要去更新一下ruby到较新的版本

	```
	 ruby -v
	 rvm list known
	 rvm install 2.2.X
	```

## 参考资料:
* [stackoverflow](http://stackoverflow.com/questions/39980096/xcode8-cocoapods-abort-trap6/40438232#40438232)
* [简书](http://www.jianshu.com/p/0fc3e0ebd78a)
*  [CocoaPods 安装与卸载](http://www.jianshu.com/p/d373181af526)











