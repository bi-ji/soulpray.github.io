---
title: Spring系列_03_Bean的注入方式
tags:
- Spring
categories: 
- Spring
---

# Bean的注入方式 #
## 项目目录 ##
![](https://i.imgur.com/2sTNTmz.png)
## 方式一：基于xml文件的形式 ##
### 步骤一：新建要配置的bean ###

```java
public class BeanExample {

    private String name;

    public void say() {
        System.out.println("Hello,I am class BeanExample!");
    }
}
```

### 步骤二：在resources下新建applicationContext.xml ###

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        id:IOC容器中bean的唯一标识
        class:IOC利用全类名反射获取bean实例
    -->
    <bean id="xmlBeanExample" class="edu.taotao.example.BeanExample"/>

</beans>
```

### 步骤三：测试 ###

>测试方式一：通过bean的id获取

```java
/**
 * 测试xml配置bean方式，并通过bean的id获取bean实例
 */
@Test
public void testXmlConfigGetBeanById() {
    // 加载xml配置文件，初始化IOC容器，获取IOC容器
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    // 获取IOC容器中的bean实例
    BeanExample example = (BeanExample) context.getBean("xmlBeanExample");
    // 调用Bean实例的方法
    example.say();
}
```

测试结果：
![](https://i.imgur.com/dz1BMyo.png)

>测试方式二:通过bean的类型来获取

```java
@Test
public void testXmlConfigGetBeanByType() {
    // 加载xml配置文件，初始化IOC容器，获取IOC容器
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    // 通过bean的类型获取，已经制定了类型，所以不需要强制转换，
    // 特别注意，通过类型获取时要保证IOC容器中只有一个该类型的Bean实例
    BeanExample example = context.getBean(BeanExample.class);
    // 调用bean实体的方法
    example.say();
}
```

测试结果
![](https://i.imgur.com/gDBFYs4.png)

## 方式二：基于注解的形式 ##

见Spring系列_12_基于注解的bean配置

## 总结 ##
>Spring中Bean的配置就成功结束了。本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。