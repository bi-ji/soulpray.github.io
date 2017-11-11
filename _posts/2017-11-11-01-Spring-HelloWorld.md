---
title: 01_Spring系列_HelloWorld
tags:
- Spring
categories: 
- Spring
---

# Spring是什么 #
1. Spring 是一个优秀的开源框架.
1. Spring 为简化企业级应用开发而生. 使用 Spring 可以使简单的 JavaBean 实现以前只有 EJB 才能实现的功能.
1. Spring 是一个 IOC(DI) 和 AOP 容器框架.
# Spring的特点 #
- 轻量级：
	Spring 是非侵入性的 基于 Spring 开发的应用中的对象可以不依赖于 Spring 的 API
- 依赖注入(DI --dependency injection、IOC)
- 面向切面编程(AOP --aspect oriented programming)
- 容器
	Spring 是一个容器, 因为它包含并且管理应用对象的生命周期
- 框架:
	Spring 实现了使用简单的组件配置组合成一个复杂的应用. 在 Spring 中可以使用 XML 和 Java 注解组合这些对象
- 一站式开发
	在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库 （实际上 Spring 自身也提供了展现层的 SpringMVC 和 持久层的 Spring JDBC）
# Spring模块 #
![](https://i.imgur.com/ACCzGkS.png)
# Spring入门之HelloWorld #
## 开发环境 ##
	- IDEA 2017
	- jdk1.8
	- spring4.3.0
	- maven3
## 项目目录结构 ##
![](https://i.imgur.com/qyVB9Ua.png)
# Spring配置Bean #
## 传统方式 ##
代码清单：
清单一:实体对象
```java
public class HelloWorld {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public void helloworld() {
        System.out.println("Hello " + name);
    }
}
```
清单二：使用实体对象
```java
public class Main {
    public static void main(String[] args) {
        // 1:创建实体对象
        HelloWorld helloWorld = new HelloWorld();
        // 2:为属性赋值
        helloWorld.setName("Spring4.X");
        // 3:调用方法
        helloWorld.helloworld();
    }
}
```
测试结果
![](https://i.imgur.com/WDDgP6o.png)
## Spring方式 ##
1. 创建SpringBean的applicationContext.xml,并添加bean实例，交给spring管理
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        创建一个实体对象，交由spring的IOC容器管理，一个bean标签对应一个实体
        id：唯一，推荐类名首字母小写
        class：id对应的实体类
    -->
    <bean id="helloWorld" class="edu.taotao.helloworld.HelloWorld">
        <!--
            设置实体对象的属性名
            name：属性名
            value：属性名的值
        -->
        <property name="name" value="applicationContext Spring4.X"/>
    </bean>
</beans>
```
2. 测试
```java
public class Main {
    public static void main(String[] args) {
        // 1：获取IOC容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 2：获取容器中的bean，有很多中获取方式，这里根据名字获取
        HelloWorld helloWorld = (HelloWorld) context.getBean("helloWorld");
        // 3：调用方法
        helloWorld.helloworld();
    }
}
```
3. 测试结果
![](https://i.imgur.com/TV5TpuR.png)
>入门程序到这里就成功结束了。本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X

