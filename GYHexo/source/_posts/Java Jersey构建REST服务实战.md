---
title: Java Jersey构建REST服务实战
date:
categories: Java
tags: [REST] 
description: 由于要做移动端和服务端传递数据，对Java接口也不是很熟悉，就采用了jersey开发，另一个REST风格的Web Service开发框架。使用jersey2.0+,现记录实现的过程，给自己和有需要的朋友参考，本文使用Eclipse编写

---

# Java Jersey构建REST服务实战

  
  
## 创建RESTful Web Service服务端

* 在Eclipse中创建一个“dynamic web project”（动态web工程） ，项目名设为 “Java_Jersey”。
* 下载[Jersey](https://jersey.java.net/download.html)
* 添加Jersey里面所有jar问价到lib文件包中。
如下图:
![Snip20170327_2.png](http://upload-images.jianshu.io/upload_images/2082481-7c1c9fb0dc5160a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 创建APIApplication

```
package com.eviac.blog.restws;

import org.codehaus.jackson.jaxrs.JacksonJsonProvider;
import org.glassfish.jersey.filter.LoggingFilter;
import org.glassfish.jersey.server.ResourceConfig;

public class APIApplication extends ResourceConfig{
	
	public APIApplication() {
		  //加载Resource
//		  register(UserInfo.class);
		packages("com.eviac.blog.restws");

		  //注册数据转换器
		  register(JacksonJsonProvider.class);

		  // Logging.
		  register(LoggingFilter.class);
		    }

}

```

## 创建User

```

package com.eviac.blog.restws;

import javax.xml.bind.annotation.XmlRootElement;
@XmlRootElement
public class User {
	 
	 private String name;
	 
	 private String age;
	 
	 private String sex;
	 
	 public String getName() {
	  return name;
	 }
	 public void setName(String name) {
	  this.name = name;
	 }
	 public String getAge() {
	  return age;
	 }
	 public void setAge(String age) {
	  this.age = age;
	 }
	 public String getSex() {
	  return sex;
	 }
	 public void setSex(String sex) {
	  this.sex = sex;
	 }
	}
```


## 创建UserInfo

```

package com.eviac.blog.restws;

import java.util.ArrayList;
import java.util.Dictionary;
import java.util.HashMap;
import java.util.List;

import javax.validation.constraints.Null;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.MediaType;

import org.apache.jasper.tagplugins.jstl.core.Set;

import com.sun.corba.se.spi.orbutil.fsm.Guard;
import com.sun.javafx.collections.MappingChange.Map;
import com.sun.org.apache.bcel.internal.generic.NEW;

//这里@Path定义了类的层次路径。 
//指定了资源类提供服务的URI路径。
@Path("UserInfoService")
public class UserInfo {
	
	// @GET表示方法会处理HTTP GET请求
	@GET
	// 这里@Path定义了类的层次路径。指定了资源类提供服务的URI路径。
	@Path("/name/{i}")
	public String userName(@PathParam("i") String i) {
		 System.out.println("你是最好的");
		String name = i;
		return "<User>" + "<Name>" + name + "</Name>" + "</User>";
	}
	
	@GET
	@Path("/age/{j}") 
	@Produces(MediaType.TEXT_XML)
	public String userAge(@PathParam("j") int j) {
	 
	int age = j;
	return "<User>" + "<Age>" + age + "</Age>" + "</User>";
	}
	
	@GET
	@Path("/getUserJson")
	@Produces(MediaType.APPLICATION_JSON)
	public User getUserJson() {
		System.out.println("出错了");
		User user = new User();
		user.setAge("23");
		user.setName("zgy");
		user.setSex("男");
		
		return user;
	}
	
	@GET
	@Path("/getUserListJson")
	@Produces(MediaType.APPLICATION_JSON)
	public List<User> getUserListJson() {
	
		List<User> users = new ArrayList<User>() ;
		
		User user = new User();
		user.setAge("23");
		user.setName("zgy");
		user.setSex("男");
		users.add(user);
		users.add(user);
		users.add(user);
		
		return users;
	}
	
	//同时支持POST和GET接口
	@GET
	@Path("/getUserInfo")
	@Produces(MediaType.APPLICATION_JSON)
	public HashMap<String, Object> getUserInfo(@QueryParam("name") String name) {
		System.out.println(name);
		HashMap map = new HashMap();
		
		
		
		List<User> users = new ArrayList<User>() ;
		
		User user = new User();
		user.setAge("23");
		user.setName("zgy");
		user.setSex("男");
		users.add(user);
		users.add(user);
		users.add(user);
		map.put("data", users);
		map.put("success", "1");
		
		return map;
	}
	
	@POST
	@Path("/getUserInfo")
	@Produces(MediaType.APPLICATION_JSON)
	public HashMap<String, Object> getUserInfopost(@QueryParam("name") String name) {
		System.out.println(name);
		HashMap map = new HashMap();
		
		
		
		List<User> users = new ArrayList<User>() ;
		
		User user = new User();
		user.setAge("23");
		user.setName("zgy");
		user.setSex("男");
		users.add(user);
		users.add(user);
		users.add(user);
		map.put("data", users);
		map.put("success", "1");
		
		return map;
	}
	
	@POST
	@Path("/getUserInfo1")
//	@PathParam("name")
	@Produces(MediaType.APPLICATION_JSON)
	public HashMap<String, Object> getUserInfo1(@QueryParam("name") String name) {
		System.out.println(name);
		HashMap map = new HashMap();

		if (name == null) {
			map.put("data", new ArrayList<User>() );
			map.put("success", "-100");
			return map;
		}
		
		List<User> users = new ArrayList<User>() ;
		
		User user = new User();
		user.setAge("23");
		user.setName("zgy");
		user.setSex("男");
		users.add(user);
		users.add(user);
		users.add(user);
		map.put("data", users);
		map.put("success", "1");
		
		return map;
	}
	
}


```

## Web.xml配置

```
<servlet> 
<servlet-name>Jersey REST Service</servlet-name> 
<servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class> 
<init-param> 
<param-name>javax.ws.rs.Application</param-name> 
<param-value>com.eviac.blog.restws.APIApplication</param-value> 
</init-param> 
<load-on-startup>1</load-on-startup> 
</servlet> 
<servlet-mapping> 
<servlet-name>Jersey REST Service</servlet-name> 
<url-pattern>/rest/*</url-pattern> 
</servlet-mapping> 
```

## 在浏览器中运行

http://localhost:8081/Java_Jersey/rest/UserInfoService/getUserInfo

输出如下图
![Snip20170327_3.png](http://upload-images.jianshu.io/upload_images/2082481-1e73d5641290aa7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  
  