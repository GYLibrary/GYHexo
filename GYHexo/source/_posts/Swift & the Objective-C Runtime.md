---
title: Swift & the Objective-C Runtime
categories: 
tags: [Runtime] 
description:Method swizzling 用于改变一个已经存在的 selector 的实现。这项技术使得在运行时通过改变 selector 在类的消息分发列表中的映射从而改变方法的掉用成为可能
---

# Swift & the Objective-C Runtime

## Method Swizzling(方法交叉)

### Objective-C

> Method swizzling用于改变一个已经存在的selector的实现。我们可以利用它使得在运行时通过改变selector在类的消息分发列表中的映射从而改变方法的调用。如下我们利用此功能在category中实现method swizzling：

``` python
#import <objc/runtime.h>

@implementation UIViewController (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(gy_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        // ...
        // Method originalMethod = class_getClassMethod(class, originalSelector);
        // Method swizzledMethod = class_getClassMethod(class, swizzledSelector);

        BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)gy_viewWillAppear:(BOOL)animated {
    [self gy_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}

@end
```

### +load vc +initialize

swizzling应该只在+load中完成，在 Objective-C 的运行时中，每个类有两个方法都会自动调用。+load 是在一个类被初始装载时调用，+initialize 是在应用第一次调用该类的类方法或实例方法前调用的。两个方法都是可选的，并且只有在方法被实现的情况下才会被调用。

### dispatch_once 

**swizzling 应该只在 dispatch_once 中完成**

由于 swizzling 改变了全局的状态，所以我们需要确保每个预防措施在运行时都是可用的。原子操作就是这样一个用于确保代码只会被执行一次的预防措施，就算是在不同的线程中也能确保代码只执行一次。Grand Central Dispatch 的 dispatch_once 满足了所需要的需求，并且应该被当做使用 swizzling 的初始化单例方法的标准。

## Selectors, Methods, & Implementations

在 Objective-C 的运行时中，selectors, methods, implementations 指代了不同概念，然而我们通常会说在消息发送过程中，这三个概念是可以相互转换的。 下面是苹果 [Objective-C Runtime Reference](https://developer.apple.com/reference/objectivec/1657527-objective_c_runtime#//apple_ref/c/func/method_getImplementation)中的描述：

> * Selector（typedef struct objc_selector *SEL）:在运行时 Selectors 用来代表一个方法的名字。Selector 是一个在运行时被注册（或映射）的C类型字符串。Selector由编译器产生并且在当类被加载进内存时由运行时自动进行名字和实现的映射。
> * Method（typedef struct objc_method *Method）:方法是一个不透明的用来代表一个方法的定义的类型。
> * Implementation（typedef id (*IMP)(id, SEL,...)）:这个数据类型指向一个方法的实现的最开始的地方。该方法为当前CPU架构使用标准的C方法调用来实现。该方法的第一个参数指向调用方法的自身（即内存中类的实例对象，若是调用类方法，该指针则是指向元类对象metaclass）。第二个参数是这个方法的名字selector，该方法的真正参数紧随其后。

理解 selector, method, implementation 这三个概念之间关系的最好方式是：在运行时，类（Class）维护了一个消息分发列表来解决消息的正确发送。每一个消息列表的入口是一个方法（Method），这个方法映射了一对键值对，其中键值是这个方法的名字 selector（SEL），值是指向这个方法实现的函数指针 implementation（IMP）。 Method swizzling 修改了类的消息分发列表使得已经存在的 selector 映射了另一个实现 implementation，同时重命名了原生方法的实现为一个新的 selector。

## Associated Objects(关联对象)

## 参考资料
1.[nshipster](http://nshipster.cn/method-swizzling/)
2.[Swift Runtime动态性分析](https://www.zybuluo.com/pockry/note/329784)




