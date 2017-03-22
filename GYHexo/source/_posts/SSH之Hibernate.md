---
title: Java SSH环境搭建
date:
categories: Java
tags: [SSH] 
description: SSH是 struts+spring+hibernate的一个集成框架，是目前比较流行的一种Web应用程序开源框架。

---

<!--# Java-->

<!--## 基本数据类型 
> boolea 
char 16位
byte 8位
short 16位
int 32位
long 64位
float 32位浮点型
double 64位浮点型-->

# SSH环境搭建

<!--## Myql 数据库建立

* mysql -u root -p（打开数据库）
* show databases; （显示已有数据库）
* create database estoresystem;(创建数据库estoresystem)
* use estoresystem; (使用数据库use estoresystem;)
* source + 路径(直接创建数据库表)-->

> struts-2.3.30 	[下载地址](http://struts.apache.org/download.cgi#struts252)
> 
> spring-framework-4.3.1 [下载地址](http://repo.spring.io/release/org/springframework/spring/)
> 
> hibernate-release-5.2.2 [下载地址](http://hibernate.org/orm/downloads/)
> 


## struts环境搭建

### 创建项目

1. 在eclipse中新建一个web工程,新建工程的时同时勾选web.xml文件如下图:
![Snip20170322_6.png](http://upload-images.jianshu.io/upload_images/2082481-42bd26c82c007760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 下载好strtus文件包并解压缩,将以下jar包复制进入我们的项目lib包中

![Snip20170322_7.png](http://upload-images.jianshu.io/upload_images/2082481-5b1cf62aeef239df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3. 找到lib目录下的web.xml文件并配置(struts版本不同过滤器写法会有差异)

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>MySqlDemo1_Hibernate</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  
  <filter>  
        <filter-name>struts2</filter-name>  
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
    </filter>  
  
    <filter-mapping>  
        <filter-name>struts2</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
</web-app>
```

4. 此处我们可以创建一个action类,一般继承ActionSupport 执行默认方法execute(),此时返回一个字符串(此字符串用于标识并显示对应的jsp页面)

```
package com.entity;

import com.opensymphony.xwork2.ActionSupport;

public class IndexAction extends ActionSupport {

	@Override
	public String execute() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("你是最帅的");
		return "handSomePig";
	}
}

```

5. 配置struts.xml文件

```

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">

    <!-- Add packages here -->
    <package name="com.entity"  extends="struts-default">
    	 <action name="Index" class="com.entity.IndexAction" method="execute">
            <result name="handSomePig">/WEB-INF/jsp/index.jsp</result>
       </action>
    </package>

</struts>

```

> action
> 
> name: 用作客户端访问时的路径相当于Servlet's path
> 
> method:与action类的方法所对应不写默认为execute方法
> 
> resultname:result中的name属性为其父标记所对应方法的返回值，不写默认为success
> 
> 若返回值与name属性值匹配则完成result标记内指定的路径进行跳转到`index.jsp`

6. 运行项目

![Snip20170322_9.png](http://upload-images.jianshu.io/upload_images/2082481-1db75d2efcf2a2a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Snip20170322_10.png](http://upload-images.jianshu.io/upload_images/2082481-e760518c587f636c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示已经显示正常

## spring环境搭建(实现查询数据库表以及简单注入)

1. 加入spring开发所需的jar包(javadoc.jar/sources非必须,如图)

![Snip20170322_11.png](http://upload-images.jianshu.io/upload_images/2082481-7acddc31abade390.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 此外我们还需要加入commons-logging.jar包,用来记录程序运行时的活动的日志记录,该文件在struts2中的lib下可找到,一个struts2中的插件包(struts2-spring-plugin-2.3.30.jar)

* mySql数据库因此还需加入数据库驱动包[mysql-connector-java-5.1.39-bin.jar](https://dev.mysql.com/downloads/connector/j/)

2. 配置struts文件、引入spring

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">

<struts> 
	<!-- 告知Struts2运行时使用Spring来创建对象  --> 
	<constant name="struts.objectFactory" value="spring" />

    <!-- Add packages here -->
    <package name="com.entity"  extends="struts-default">
    <!-- 这里的class并没有引用一个具体的类而是取了一个别名，接下来再由spring给它注入 -->
    	 <action name="Index" class="myIndexAction">
            <result name="success">/WEB-INF/jsp/index.jsp</result>
       </action>
    </package>

    <package name="com.action"  extends="struts-default">
    	 <action name="Index1" class="myIndexAction1" method="FindStudents">
            <result name="success1">/WEB-INF/jsp/login.jsp</result>
       </action>
    </package>

</struts>

```
3. 配置applicationContext.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">


	<bean id="myIndexAction" class="com.entity.IndexAction" scope="prototype">
		<!--<property name="index" ref="myService1"></property>  -->
	</bean>

	
	<bean id="myService1" class="com.service.StudentsServiceImpl" scope="prototype">
		<property name="studentsDao" ref="myDao1"></property>
	</bean>
		
	<bean id="myDao1" class="com.Dao.StudentsDaoImp" scope="prototype">
	<property name="c" ref="myConnection"></property>
	</bean>
	
	<bean id="myConnection" class="com.tools.MyConnection" scope="prototype"></bean>
	
	<bean id="myIndexAction1" class="com.action.FindStudentsAction" scope="prototype">
		<property name="studentsService" ref="myService1"></property> 
	</bean>
	
</beans>
```

spring注入之后耦合度降低

![Snip20170322_12.png](http://upload-images.jianshu.io/upload_images/2082481-a8cf5808f5cd6d60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Hibernate 环境搭建

1. 导入jar包

![Snip20170322_13.png](http://upload-images.jianshu.io/upload_images/2082481-73ab2193b7ac5e2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 需要在src目录下配置一个实体类映射文件这里以(Students.hbm.xml)为例

```
<?xml version="1.0"?> <!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN" "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd"> <!-- Generated 2016-8-29 15:04:12 by Hibernate Tools 3.4.0.CR1 -->
<hibernate-mapping>
	<class name="com.entity.Students" table="STUDENTS">
		<id name="sid" type="int">
			<column name="SID" />
			<generator class="native" />
		</id>
		<property name="name" type="java.lang.String">
			<column name="NAME" />
		</property>
		<property name="gender" type="java.lang.String">
			<column name="GENDER" />
		</property>
		<property name="birthday" type="java.util.Date">
			<column name="BIRTHDAY" />
		</property>
		<property name="address" type="java.lang.String">
			<column name="ADDRESS" />
		</property>
	</class>
</hibernate-mapping>

```

3. 配置连接数据库相关信息的hibernate.cfg.xml文件

```

<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
          "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
          "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>

<session-factory>

  <property name="dialect">org.hibernate.dialect.MySQLDialect</property> 
  <property name="show_sql">true</property>
  <property name="format_sql">true</property>
  <property name="connection.pool_size">5</property>

  <property name="connection.username">root</property> 
  <property name="connection.password">zgy666666</property> 
  <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
  <property name="connection.url">jdbc:mysql://localhost:3306/mysql</property> 

  <mapping resource="Students.hbm.xml"/> 
  
  </session-factory>
</hibernate-configuration>

```


## 附

[易百](http://www.yiibai.com/hibernate/hibernate_query_language.html)

[Demo](https://github.com/GYJava/SSHDemo)





