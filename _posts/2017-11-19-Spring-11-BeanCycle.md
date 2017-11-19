---
title: Spring系列_11_Bean的生命周期
tags:
- Spring
- 生命周期
- Bean
categories: 
- Spring
---

# Bean的生命周期 #

Spring IOC容器可以管理Bean的生命周期，Spring允许在Bean生命周期的特定点执行特定的任务。
<br/>
在Spring IOC容器中对Bean的生命周期的管理分为两种

- 普通生命周期
- 带后置处理器的Bean的生命周期

<br/>Spring中Bean的生命周期里，通过在声明bean的时候设置init-method和destroy-method属性，为Bean指定初始化和销毁的方法。

>写在前面，本节代码清单

>Person.java

```java
@Data
public class Person {

    private String name;

    public Person(){
        System.out.println("person's constructor......");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("person's setName() " + name);
        this.name = name;
    }

    /**
     * person的初始化方法
     */
    public void initMetod(){
        System.out.println("person's initMethod......");
    }

    /**
     * person的销毁方法
     */
    public void destroyMethod(){
        System.out.println("person's destroyMethod .......");
    }
}
```

>MyBeanPostProcessor.java(Bean的后置处理器)

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    // 在Bean的init-method方法调用之前执行此方法
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization: " + bean + " " + beanName);
        // 对bean做其他操作，也可以对指定beanName的bean进行处理，也可以对指定类型的bean进行处理，总之，你可以在这里为所欲为
        return bean;
    }
    // 在Bean的init-method方法调用之后执行此方法
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization: " + bean + " " + beanName);
        // 对bean做其他操作，也可以对指定beanName的bean进行处理，也可以对指定类型的bean进行处理，总之，你可以在这里为所欲为
        return bean;
    }
}
```

>TestBeanCycle.java

```java
public class TestBeanCycle {
    
    private ClassPathXmlApplicationContext context;
    
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
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

>项目目录结构

![](https://i.imgur.com/dglO00m.png)

## 普通生命周期 ##

### Bean的普通生命周期分为以下几个过程 ###

1. 通过构造器或工厂方法创建 Bean 实例
2. 为 Bean 的属性设置值和对其他 Bean 的引用
3. 调用 Bean 的初始化方法
4. Bean 可以使用了
5. 当容器关闭时, 调用 Bean 的销毁方法


>applicationContext.xml

```xml
<!--创建Person Bean，通过init-method指定初始化方法，通过destroy-method指定销毁方法-->
<bean id="person" class="edu.taotao.example.Person"
      init-method="initMetod"
      destroy-method="destroyMethod">
    <property name="name" value="Tom"/>
</bean>
```

>测试

```java
/**
 * 测试Bean的生命周期
 */
@Test
public void testNormalBeanCycle(){
    Person person = (Person) context.getBean("person");
    System.out.println("person = " + person);
    context.destroy();
}
```

>测试结果

![](https://i.imgur.com/SpeGPfK.png)

通过图片中所标注的1-5的序号和之前的bean的生命周期进行对比，发现完全符合。

## 带后置处理器的Bean的生命周期 ##

### 什么是Bean的后置处理器 ###

Bean 后置处理器允许在调用初始化方法前后对 Bean 进行额外的处理.<br/>
Bean 后置处理器对 IOC 容器里的所有 Bean 实例逐一处理, 而非单一实例. 其典型应用是: 检查 Bean 属性的正确性或根据特定的标准更改 Bean 的属性.<br/>
对Bean 后置处理器而言, 需要实现BeanPostProcessor接口. 在初始化方法被调用前后, Spring 将把每个 Bean 实例分别传递给上述接口的以下两个方法:

	public Object postProcessBeforeInitialization(Object bean, String beanName) 

	public Object postProcessAfterInitialization(Object bean, String beanName)

### 带后置处理器的Bean的生命周期过程 ###

1. 通过构造器或工厂方法创建 Bean 实例
2. 为 Bean 的属性设置值和对其他 Bean 的引用
3. 将 Bean 实例传递给 Bean 后置处理器的 postProcessBeforeInitialization 方法
4. 调用 Bean 的初始化方法
5. 将 Bean 实例传递给 Bean 后置处理器的 postProcessAfterInitialization方法
6. Bean 可以使用了
7. 当容器关闭时, 调用 Bean 的销毁方法

>applicationContext.xml

```xml
<!--注册Bean的后置处理器-->
<bean id="myBeanPostProcessor" class="edu.taotao.example.MyBeanPostProcessor"/>
```

>测试

```java
/**
 * 测试Bean的生命周期
 */
@Test
public void testNormalBeanCycle(){
    Person person = (Person) context.getBean("person");
    System.out.println("person = " + person);
    context.destroy();
}
```

>测试结果

![](https://i.imgur.com/yIqcdWa.png)

通过图片中所标注的1-7的序号和带后置处理器的bean的生命周期进行对比，发现完全符合。

## 总结 ##

关于在Spring中Bean的生命周期就介绍到这里了。

>本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。