---
title: Spring系列_09_外部属性文件
tags:
- Spring
- properties
- Bean
categories: 
- Spring
---

# 外部属性文件 #

本节将为大家介绍如何在Spring使用外部属性文件，使用外部属性文件的好处就是可以将配置的职责进行分明，在更改配置的时候能能够更快的定位到配置文件中。比如常用的数据库链接的配置，文件路径等信息。Spring 提供了一个 PropertyPlaceholderConfigurer 的 BeanFactory 后置处理器, 这个处理器允许用户将 Bean 配置的部分内容外移到属性文件中. 可以在 Bean 配置文件里使用形式为 ${var} 的变量, PropertyPlaceholderConfigurer 从属性文件里加载属性, 并使用这些属性来替换变量.Spring 还允许在属性文件中使用 ${propName}，以实现属性之间的相互引用


>写在前面，本节代码清单

>DataSource.java（模拟jdbc数据源注入）

```java
@Data
public class DataSource {

    private String driver;
    private String url;
    private String username;
    private String password;
}
```

>TestExternalPropertiesFile.java

```java
public class TestExternalPropertiesFile {

    private ApplicationContext context;

    @Before
    public void getApplicationContext(){
        context = new ClassPathXmlApplicationContext("applicationContext.xml");
    }

}
```

>applicationContext.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-2.5.xsd">

</beans>
```

这里一定要引入使用PropertyPlaceholderConfigurer的命名空间

	xmlns:context="http://www.springframework.org/schema/context"
	http://www.springframework.org/schema/context
   	http://www.springframework.org/schema/context/spring-context-2.5.xsd

>数据库配置文件：jdbc.properties

```xml
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.driver=com.mysql.jdbc.Driver
jdbc.username=root
jdbc.password=root
```

>项目目录结构

![](https://i.imgur.com/lRZFBxy.png)

## 外部属性文件配置 ##

>applicationContext.xml

```xml
<!--加载jdbc.properties配置文件-->
<context:property-placeholder location="jdbc.properties"/>

<!--注入dataSource的bean-->
<bean id="dataSource" class="edu.taotao.example.DataSource">
    <!--使用${key}的方式注入值，其中key为properties中的键-->
    <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="password" value="${jdbc.username}"/>
    <property name="username" value="${jdbc.password}"/>
</bean>
```

>测试

```java
/**
 * 测试外部配置文件注入
 */
@Test
public void testExternalPropertiesFile(){
    DataSource dataSource = (DataSource) context.getBean("dataSource");
    System.out.println("dataSource = " + dataSource);
}
```

>测试结果

![](https://i.imgur.com/NoMY68r.png)

## 总结 ##

关于在Spring中集成外部属性配置文件的方式就介绍到这里了。

>本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。