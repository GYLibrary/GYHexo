---
title: Sqlite
date: 
categories: iOS
tags: [SDK集成] 
description: 本章记录如何利用Swift与OC混合开发的SDK制作Framework,记录制作Framework过程中所搜集以及遇到的坑。
---

# Swfit 制作Framework
## iOS Framework简介

> Framework是Cocoa/Cocoa Touch程序中使用的一种资源打包方式，可以将代码文件、头文件、资源文件、说明文档等集中在一起，方便开发者使用。一般如果是静态Framework的话，资源打包进Framework是读取不了的。静态Framework和.a文件都是编译进可执行文件里面的。只有动态Framework能在.app下面的Framework文件夹下看到，并读取.framework里的资源文件

* Framework类型

```
Mach-o Type(Build Settings)
Dynamic library//因为本篇文章介绍的是以Swift为基础制作Framework（Swift仅支持动态包）
Static library
```


## Framework制作过程

* 新建工程

![创建工程.png](http://upload-images.jianshu.io/upload_images/2082481-79521ef671844522.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 导入所需要的库文件

![Snip20180125_11.png](http://upload-images.jianshu.io/upload_images/2082481-a18ae7a4d849be06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
此处导入OC文件时不需要建立桥接文件,系统会自动为我们创建
供外部使用的Swift类文件以及相关属性和方法等需要用Public修饰。
```

* 利用cocoapods导入所需第三方库

![cocoapods.png](http://upload-images.jianshu.io/upload_images/2082481-55506c80c2d4fa51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
添加方法与之前项目相同use_frameworks!来告诉pod 生成动态库文件Framework类型
此处需要注意利用打包Framework之时加入的第三方库，需要通知使用者在项目中也添加，并非已经嵌入了Framework中，此时所做相当于把第三方库链接进了Framework,仍需添加此Framework的项目添加其所用到的第三方库,并且版本需要保持一致
```

![Snip20180125_6.png](http://upload-images.jianshu.io/upload_images/2082481-e3917a2b874d15e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 1.对外公布的头文件
> 2.Sources文件
> 3.cocoadpos
> 4.command + B编译后生成的Framework(当前真机或模拟器就对应,可将真机与模拟器的Framework合并,可自行百度)


![Snip20180125_10.png](http://upload-images.jianshu.io/upload_images/2082481-2c1a87f3aa36043b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
对应打包后自动生成Giant-swift的桥接文件
```

## 添加到所用项目中所遇到的问题

![Snip20180125_14.png](http://upload-images.jianshu.io/upload_images/2082481-57d4a948bd86c06a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Snip20180125_17.png](http://upload-images.jianshu.io/upload_images/2082481-bd0c1bed5ab9e348.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
此处需要导入General-> Embedded Binaries Giant.framework
对于之前提到的项目也需要导入cocopods的第三方库Xcode 9.2较为友好的提示了,但是在Xcode 9.0 并无提示
由于家中电脑Xcode版本与公司不一致,此处不再重现截图
```
![Snip20180125_15.png](http://upload-images.jianshu.io/upload_images/2082481-79e43afb60287817.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
制作Framework编译通过，但是在调用结构体的时候报错,是因为创建结构体的时候初始化方法未public 权限有问题,按如下修改,编译即可通过
```

![Snip20180125_9.png](http://upload-images.jianshu.io/upload_images/2082481-26d3e534c33f8ffb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Snip20180125_12.png](http://upload-images.jianshu.io/upload_images/2082481-63114f0f40528762.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Framework中读取自身文件的路径
Bundle.init(for: ViewController.self).resourcePath
拼接"/文件名.png"即可
```

```
[问题参考](https://www.jianshu.com/p/457c941eae5e)
```





