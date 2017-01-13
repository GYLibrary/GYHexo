### associatedtype
关联类型:代指变量
associatedtype 用于protocol中.
associatedtype类型是protoc
ol中32代指一个确定类型并要求改类型实现指定方法


Swift 标准库中有许多 protocols，其中很多看起来貌似很抽象，并且感觉并没有什么卵用，RawRepresentable 就是其中之一，也

``` py
public protocol RawRepresentable {
    associatedtype RawValue

    public init?(rawValue: Self.RawValue)

    public var rawValue: Self.RawValue { get }
}
```

[CustomStringConvertible协议](http://swiftcafe.io/2016/01/15/printable/)

[internal(set)](http://www.devtalking.com/articles/swift-access-control/)
[苹果](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html)
[Tips](http://swifter.tips/property-access/)

[RunLoop](http://artjing.com/2016/05/20/RUNLOOP/)

AFNetworking的做法
开启一个不死线程(线程保活)

```
AFURLConnectionOperation
+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });

    return _networkRequestThread;
}
```


```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];

        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
```
