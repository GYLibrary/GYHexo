---
title: Mac设置开机启动项
date: 2017-12-7 19:55:16
categories: Mac
tags: [Mac终端] 
description: 本文记录Mac开机自启动以及定时任务的添加
---

#  Mac设置开机启动项
>  由于最近使用到开机自启动以及定时任务的开启,在采坑的同事记录一下Mac自启动的以及定时任务开启的正确姿势

# Mac 开机自启动以及定时任务设置方式
* 1. 使用登录项添加可执行的脚本(此处不做介绍，注意添加可执行权限即可(chmod 777 ..sh))
*  2.  launchctl加载plist文件
*  3.  crontab添加可执行文件以及脚本

##  launchctl加载plist文件

Mac开机启动大部分会使用**launchctl**加载文件,launchctl 通过 **plist** 属性列表（Property List）配置。

plist文件位置以及相关权限

```
~/Library/LaunchAgents 由用户自己定义的任务项
/Library/LaunchAgents 由管理员为用户定义的任务项
/Library/LaunchDaemons 由管理员定义的守护进程任务项
/System/Library/LaunchAgents 由Mac OS X为用户定义的任务项
/System/Library/LaunchDaemons 由Mac OS X定义的守护进程任务项
```
LaunchDaemons和LaunchAgents的区别？

```
LaunchDaemons是用户未登陆前就启动的服务,即开机即可启动
LaunchAgents使用户登陆后启动的服务
```

具体plist设置[苹果官方教材](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man5/launchd.plist.5.html#//apple_ref/doc/man/5/launchd.plist),以下示例:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>cn.GY.tasklogout</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Applications/ATSign.app/Contents/MacOS/ATSign</string>
	</array>
	<key>StartCalendarInterval</key>
	<dict>
              <key>Minute</key>
              <integer>30</integer>
              <key>Hour</key>
              <integer>9</integer>
              <key>Day</key>
            <integer>1</integer>
            <key>Month</key>
            <integer>5</integer>
            <!-- 0和7都指星期天 -->
            <key>Weekday</key>
            <integer>0</integer>
		**<key>Minute</key>
		<string>01</string>
		<key>Hour</key>
		<string>18</string>**
	</dict>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>
```

> **注意**: 在**--**区间的设置时间点的plist的key,此处必须为integer,而非string,刚开始使用string会造成app一直无限自启动,后来查文档发现是integer,才解决了这个问题。

### 部分键值说明

* Label
   服务的名称,不可重复,可与当前plist文件名一致
* Program
  需要启动的程序(可不设置ProgramArguments)
* ProgramArguments
   -startup开机启动
* StartCalendarInterval
  此处设置具体时分秒以及工作日等相关信息的设置,注意key值类型相照应,如果不设置会默认任意时间点启动
### Mac通过launchctl加载plist的相关命令
* 检查plist语法是否正确

```
plutil ~/Library/LaunchAgents/demo.plist
```

* 加载配置文件,使配置文件生效

```
launchctl load ~/Library/LaunchAgents/demo.plist
```
* 取消当前配置文件的进程

```
launchctl unload ~/Library/LaunchAgents/demo.plist
```
* 查看当前你服务是否加入

```
launchctl list 所有服务
launchctl list | grep demo 过滤后的服务
```
* 如果加载服务后,再次修改该服务plist文件,可通过取消当前配置在加载当前配置的方式修改服务

```
launchctl unload
launchctl load
```

## crontab执行脚本文件等

编辑自定义自己的任务

```
 crontab -e
 13 15 * * *  /usr/local/bin/python2.7  
 /Users/macprohz/Desktop/Python/WebAppDemo/GY.py
```
> 添加编辑wq保存即可, 以上代表意义:在15:30分启动python脚本GY.py

时间格式

```
 *      *      *     *      *   command
 分     时      日    月     周    命令
```
> 第1列表示分钟1～59 每分钟用或者 /1表示 
第2列表示小时1～23（0表示0点） 
第3列表示日期1～31 
第4列表示月份1～12 
第5列标识号星期0～6（0表示星期天） 
5个*表示每分钟 
*表示每分钟/时/日/月/周 
*/n表示每隔n分钟/时/日/月整/周 
每个时间位多个数值用逗号隔开：* * * * 0,1,2,3,4,5就表示除了周六以外的每一分钟 
```
     “*”代表取值范围内的数字,

     “/”代表”每”,

     “-”代表从某个数字到某个数字,

     “,”分开几个离散的数字
```

crontab的一些终端命令
```
ps aux | grep cron :查看服务是否已经运行用 
crontab -e:编辑当前用户crontab任务，保存退出后自动加到crontab列表中执行 
crontab -l :查看当前用户所有crontab 列表 
crontab -r :删除当前用户所有crontab 列表
```

> 总结:此文仅为记录Mac下的启动方式,防止个人用到时又要无休止的goole,大部分资源还是很早之前的,特此记录(GiantForJade)。

参考文献
[launchctl](http://hanks.pub/2016/03/28/mac-launchctl/)
[crontab](http://blog.csdn.net/py_tester/article/details/78272006)










