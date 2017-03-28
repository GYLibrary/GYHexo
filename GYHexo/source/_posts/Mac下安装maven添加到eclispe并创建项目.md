---
title: Mac下安装maven添加到eclispe并创建maven项目
date:
categories: Java
tags: [Maven] 
description: 学习并记录安装maven的全过程

---



# Mac OS X 安装Maven

*  下载 [Maven](https://maven.apache.org/download.cgi), 并解压到某个目录。例如/Users/sh15bg0110/Desktop/JavaService/apache-maven-3.3.9
*  打开终端,输入以下命令,设置Maven classpath

	```
$ vi ~/.bash_profile

	```

* 添加下列两行代码，之后保存并退出:

	```
	M2_HOME=/Users/sh15bg0110/Desktop/JavaService/apache-maven-3.3.9/
	export M2_HOME
	```
	![1.png](http://upload-images.jianshu.io/upload_images/2082481-59e38f7368a1b4ba.png)

此处注意路径设置

* 输入命令以使bash_profile生效

	```
	$ source ~/.bash_profile
	```
	
* 输入mvn -v查看Maven是否安装成功

	```
	Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /Users/sh15bg0110/Desktop/JavaService/apache-maven-3.3.9
Java version: 1.8.0_101, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.12.2", arch: "x86_64", family: "mac"
	```
# 配置Maven的库
![2.png](http://upload-images.jianshu.io/upload_images/2082481-66204a1ecd52423a.png)

![3.png](http://upload-images.jianshu.io/upload_images/2082481-dc496c6b181ba50c.png)

