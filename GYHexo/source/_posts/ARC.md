---
title: iOS开发ARC内存管理
date: 2015-02-22 21:15:16
categories: iOS
tags: [iOS]
description: ARC的学习与总结
---
# iOS开发ARC内存管理
## ARC的本质

```
ARC是编译时特性，而不是运行时特性,更不是垃圾回收机制(GC)
ARC(Automatic Reference Counting) 
[ARC官方文档](http://clang.llvm.org/docs/AutomaticReferenceCounting.html)
```
如果需要对特定文件开启或关闭ARC，可以在工程选项中选择Targets -> Build Phases -> Compile Sources，在里面找到对应文件，添加flag:
	•	打开ARC：-fobjc-arc
	•	关闭ARC：-fno-objc-arc
	
	
## ARC的修饰符
ARC主要提供了4种修饰符,他们分别是:__strong,__weak,__autorelleasing,__unsafe_unretained

###__strong
表示引用为强引用,对应在定义property时的strong，所有对象只有当没有任何一个强引用指向时，才会被释放

注意：如果在声明引用时不加修饰符，那么引用将默认是强引用。当需要释放强引用指向的对象时，需要将强引用置nil
###__weak
表示引用为弱引用。对应在定义property时的"weak".弱引用不会影响对象的释放,即只要对象没有任何强引用指向,即使有1001个弱引用对象指向也无用,该对象依然会被释放。不过好在，对象在被释放的同时，指向它的弱引用会自动被置nil，这个技术叫zeroing weak pointer。这样有效得防止无效指针、野指针的产生。__weak一般用在delegate关系中防止循环引用或者用来修饰指向由Interface Builder编辑与生成的UI控件。

###__autoreleasing
表示在autorelease pool中自动释放对象的引用，和MRC时代autorelease的用法相同。定义property时不能使用这个修饰符，任何一个对象的property都不应该是autorelease型的。
一个常见的误解是，在ARC中没有autorelease，因为这样一个“自动释放”看起来好像有点多余。这个误解可能源自于将ARC的“自动”和autorelease“自动”的混淆。其实你只要看一下每个iOS App的main.m文件就能知道，autorelease不仅好好的存在着，并且变得更fashion了：不需要再手工被创建，也不需要再显式得调用[drain]方法释放内存池。

###__unsafe_unretained
ARC是在iOS 5引入的，而这个修饰符主要是为了在ARC刚发布时兼容iOS 4以及版本更低的设备，因为这些版本的设备没有weak pointer system，简单的理解这个系统就是我们上面讲weak时提到的，能够在weak引用指向对象被释放后，把引用值自动设为nil的系统。这个修饰符在定义property时对应的是"unsafe_unretained"，实际可以将它理解为MRC时代的assign：纯粹只是将引用指向对象，没有任何额外的操作，在指向对象被释放时依然原原本本地指向原来被释放的对象（所在的内存区域）。所以非常不安全。
现在可以完全忽略掉这个修饰符了，因为iOS 4早已退出历史舞台很多年。

转自[123](http://www.cnblogs.com/flyFreeZn/p/4264220.html)
	













