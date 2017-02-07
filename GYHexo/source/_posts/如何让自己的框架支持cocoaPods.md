---
title: 如何让自己的框架支持cocoaPods
tags: [cocoaPods] 
description: 制造轮子，为了方便版本管理，一般会让自己的框架支持cocoaPods进行版本管理，加上描述这样可以一目了解，因为之前生成过cocoaPods，可是没有记录，今天特别记录一下，以及框架在生成支持cocoaPods时候遇到的问题。
---

# 如何让自己的框架支持cocoaPods

## 创建github新仓库

![图片.png](http://upload-images.jianshu.io/upload_images/2082481-6a15428e8292325e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Git Tag的使用

Tag
> tag 用于创建一个标签 用于在开发阶段，某个阶段的完成，创建一个版本，在开发中都会使用到, 可以创建一个tag来指向软件开发中的一个关键时期，比如版本号更新的时候可以建一个1.0.0,1.1.1之类的标签，这样在以后回顾的时候会比较方便。tag的使用很简单

基本操作有：查看、创建、验证、共享`tag`。

1. 查看tag
   
   ```
   	git tag
  	```
  	 	
2. 创建tag

	```
	git tag version 1.0  也可以 git tag 1.0
	```
3. 删除tag

	```
	git tag -d 1.0
	```

4. 共享tag

	我们在执行 `git push` 的时候，`tag`是不会上传到服务器的，比如现在的`github`，创建 `tag` 后 `git push` ，在github网页上是看不到`tag` 的，为了共享这些tag，你必须这样：

	```
	git push origin --tags
	```
	输出
	
	```
	zgy:GYNetWorkING zhuguangyang$ git push origin --tags 0.0.1
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/GYLibrary/GYNetWorking.git
 * [new tag]         0.0.1 -> 0.0.1

	```

## 创建podspec文件

首先你需要打开终端进入自己的工作目录的根目录，输入以下命令

```
touch name.podspec(name就是最终确定上传到cocoapods上的命名)
```
或者

```
 pod spec create name(name就是最终确定上传到cocoapods上的命名)
```

修改对应的部分信息如下:

```
Pod::Spec.new do |s|

  s.name         = "GYNetWorking"
  s.version      = "0.0.9"
  s.summary      = "网络请求测试库"
  s.homepage     = "https://github.com/GYLibrary/GYNetWorking"
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.author       = { "name" => "XXX@qq.com" }
  s.platform     = :ios, "8.0"
  s.source       = { :git => 'https://github.com/GYLibrary/GYNetWorking.git', :tag => s.version }
  s.source_files = "GYNetWorking/NetWork/*.swift"
  s.requires_arc = true

  s.pod_target_xcconfig = { 'SWIFT_VERSION' => '3.0' }
  
end
```


## 注册cocoapods账号

打开终端,在终端中输入如下命令

```
pod trunk register XXX@qq.com 'name' ----description='描述信息'

```
终端输出:

```
[!] Please verify the session by clicking the link in the verification email that has been sent to XXX@qq.com
[请进入邮箱点击验证链接即可]
```

## 验证Podspec

```
pod lib lint 或者 pod lib lint --allow-warnings
```

```
pod trunk push [NAME].podspec 或者 pod lib lint --allow-warnings
```

如果有如此类的错误输出

```
[!] The spec did not pass validation, due to 3 warnings (but you can use `--allow-warnings` to ignore them).
```
使用如下命令行即可解决

```
pod lib lint --allow-warnings
```

最后成功后控制台输出

```
 🎉  Congrats

 🚀  GYNetWorking (0.0.9) successfully published
 📅  February 7th, 00:18
 🌎  https://cocoapods.org/pods/GYNetWorking
 👍  Tell your friends!
```

恭喜 到此时您已成功发布框架到cocoaPods

您可输入一下命令进行查看您已成功上传到cocoaPods的框架

```
pod trunk me
```

```
  - Name:     xxx
  - Email:    xxx@qq.com
  - Since:    September 24th, 2016 17:27
  - Pods:
    - SwiftDrawEnum
    - GYNetWorking
  - Sessions:
    - September 24th, 2016 17:27 - January 31st, 04:56. IP: 192.155.208.125
    Description: Draw E
    - February 6th, 23:44        -    June 15th, 00:20. IP:
    183.195.156.242

```

## 参考资料:

http://blog.csdn.net/keleyundou/article/details/49635589

http://www.jianshu.com/p/f332b2c53280