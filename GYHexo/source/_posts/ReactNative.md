---
title: ReactNative
tags: [ReactNative] 
description: React Native使你能够在Javascript和React的基础上获得完全一致的开发体验，构建世界一流的原生APP。
---

# ReactNative

##  端口占用 @providesModule declaration with the same name accross two different files.

> 解决方案:
> 
> 1.命令 lsof -i tcp:port  （port替换成端口号，比如8081）可以查看该端口被什么程序占用，并显示PID，方便KILL
> 
> 2.kill -9 pid 
> 
> 3.重启Xcode