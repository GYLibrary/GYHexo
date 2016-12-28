---
title: Dispatch once in Swift 3
date: 2016-09-22 20:50:16
categories: swift
tags: [多线程] 
description: 记录Swift 3.0 中 Dispatch once的修改，这里提供两个类似于此功能的方法
---

## Dispatch once in Swift 3

### Old Version
在Swift2.3中,为保证一段代码只被执行一次，可以利用dispatch_once
如下代码：

```
var onceToken:dispatch_once_t = 0
func dosomeThing() {
	dispatch_once(&onceToken){
	}
}
```
此方法经常用于单利的创建等等,但是在Swift3.0中已被弃用,这里提供两个类似于dispatch_once的功能实现方式

## 方法一

### New 

因为此方法已经被弃用,所以可以利用分类来实现类似于dispatch_once的功能,如下代码

```
public extension DispatchQueue {
    
    public static var _onceTracker = [String]()
    
    
    public class func  once(token: String, block:(Void)->Void) {
        
     objc_sync_enter(self)
        defer {
            objc_sync_exit(self)
        }
        
        if _onceTracker.contains(token) {
            return
        }
        
        _onceTracker.append(token)
        block()
        
    }
    
}
```

此时我们调用此方法只需要

```
DispatchQueue.once(token: "com.gybar.new") { (_) in
   //doSomeThing         
}
```

### 进一步精简化

```
public extension DispatchQueue {
    
    public static var _onceTracker = [String]()
    
    public class func once(file: String = #file, function: String = #function, line: Int = #line, block:(Void)->Void) {
        let token = file + ":" + function + ":" + String(line)
        once(token: token, block: block)
    }
    
    public class func  once(token: String, block:(Void)->Void) {
        
     objc_sync_enter(self)
        defer {
            objc_sync_exit(self)
        }
        
        if _onceTracker.contains(token) {
            return
        }
        
        _onceTracker.append(token)
        block()
        
    }
    
}

```

就可以实现类似于dispatch_once的功能
### 方法二

```
class Object {
    static let sharedInstance = Object()
}

// Global constant.
let constant = Object()

// Global variable.
var variable: Object = {
    let variable = Object()
    variable.doSomething()
    return variable
}()
```

[参考](https://gist.github.com/marmelroy/15da0fa6f0936ed6b92392e9df07e235)







