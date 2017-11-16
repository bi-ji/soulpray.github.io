---
title: Spring系列_08_Bean的作用域
tags:
- Spring
- scope
- Bean
categories: 
- Spring
---

# Bean的作用域 #

本节将为大家介绍Bean的作用域，在Spring中Bean一共有四种作用域，其中常用的有两种

- singleton(常用)
- prototype(常用)
- request
- session

>写在前面，本节代码清单

>Person.java

```java
@Data
public class Person {

    private String name;

	public Person() {
        System.out.println("Person Constructor");
    }
}
```

>TestBeanScope.java

```java
public class TestBeanScope {

    private ApplicationContext context;

    @Before
    public void getApplicationContext(){
        context = new ClassPathXmlApplicationContext("bean-scope.xml");
    }

}
```

>bean-scope.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd">



</beans>
```

## singleton作用域 ##

singleton，见名知意，这种方式注入的bean是单例的，并且伴随着IOC容器的初始化而初始化，下面就将单例性，和伴随IOC容器的初始化而初始化做验证

- 验证随IOC容器一起创建

>bean-scope.xml

```xml
<!--创建一个单例的person,通过设置scope属性为singleton-->
<bean id="singletonPerson" class="edu.taotao.example.Person" scope="singleton"/>
```
>测试

我们写一个空的测试方法，并运行可以看到输出Person的构造器中的输出语句，可以验证scope为singleton的bean会随Spring IOC容器一起创建。

```java
@Test
public void testSingletonBean(){
    
}
```

>测试结果

![](https://i.imgur.com/QpTePz1.png)

- 验证单例性

>bean-scope.xml

使用1中的bean-scope.xml

>测试

在1的基础上进行添加代码，用来验证bean的单例性

```java
/**
 * 测试bean的scope为singleton
 */
@Test
public void testSingletonBean(){
    Person personOne = (Person) context.getBean("singletonPerson");
    Person personTwo = (Person) context.getBean("singletonPerson");
    System.out.println("(personOne == personTwo) is " + (personOne == personTwo));
}
```	

>测试结果

![](https://i.imgur.com/VKFtkTP.png)

通过测试结果也看以看出，从IOC容器中取出了两个Person的实例，只调用了一次Person的构造器，而且两个Person的引用指向同一个地址，说明了bean是单例的。

## prototype作用域 ##

prototype：原型作用域，不会随着IOC容器的创建而创建，而会在每次从IOC容器中取出该bean类型的实例时创建一个新的原型给你。

- 测试不会随IOC容器一起创建

>bean-scope.xml

在测试prototype时记得要把在singleton中配置的bean**注释**掉，不然会引发验证错误，因为单例的会随着IOC一起创建。

```xml
<!--创建一个原型的person bean，通过设置scope属性为prototype-->
<bean id="prototypePerson" class="edu.taotao.example.Person" scope="prototype"/>
```

>测试

```java
/**
 * 测试bean的scope为prototype
 */
@Test
public void testPrototypeBean(){

}
```

>测试结果

![](https://i.imgur.com/yDjPuwT.png)

我们可以看到，控制台什么也没输出，说明Person Bean并没有随着IOC容器的创建而创建。

- 测试其原型性

>bean-scope.xml

同上

>测试

在测试1的基础上进行追加

```java
/**
 * 测试bean的scope为prototype
 */
@Test
public void testPrototypeBean(){
    Person personOne = (Person) context.getBean("prototypePerson");
    Person personTwo = (Person) context.getBean("prototypePerson");
    System.out.println("(personOne == personTwo) is " + (personOne == personTwo));
}
```

>测试结果

![](https://i.imgur.com/ov2njVu.png)

通过截图中的黄色框中内容可以看到，Person的构造方法被调用了两次，而且personOne对象和personTwo对象的引用并不相等，说明是两个对象。

## 说明 ##

由于request和session scope的Bean并不常用，这里不再介绍。

>本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。