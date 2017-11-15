---
title: Spring系列_06_自动注入bean
tags:
- Spring
- autowired
- Bean
categories: 
- Spring
---

# 自动注入bean #

在上一节中我们讲了很多bean的配置细节，但所有都是我们手工配置，或手工指定参考bean的方式来对bean进行配置的，在Spring中也可以自动注入bean，自动注入bean的方式有两种：

- byName
- byType
- constructor(当bean中存在多个构造器时，很复杂，不推荐使用，就不演示了)

>写在前面，本文中所使用的代码清单

>Person.java

```java
@Data
public class Person {

    private String name;

    private Integer age;

    private Address address;
}
```

```java
@Data
public class Address {

    private String country;

    private String province;

    private String city;

    private String area;

    private String street;

}
```

```java
public class TestAutowired {

    private ApplicationContext context;

    /**
     * 用于执行每个测试用例前的初始化工作
     */
    @Before
    public void getApplicationContext() {
        context = new ClassPathXmlApplicationContext("applicationContext.xml");
    }

}
```

## 自动注入ByName ##

>applicationContext.xml配置

```xml
<!--address bean one-->
<bean id="address" class="edu.taotao.example.Address">
    <property name="country" value="China"/>
    <property name="province" value="SiChuan"/>
    <property name="area" value="JingRongTown"/>
</bean>

<!--address bean two-->
<bean id="address2" class="edu.taotao.example.Address">
    <property name="country" value="China"/>
    <property name="province" value="BeiJing"/>
    <property name="city" value="ZhongGuanCun"/>
</bean>
<!--person注入bean通过自动注入，注入方式同过名字-->
<bean id="person" class="edu.taotao.example.Person" autowire="byName"/>
```

>测试一

```java
/**
 * 通过名字自动注入
 */
@Test
public void testAutowiredByName(){
    Person person = (Person) context.getBean("person");
    System.out.println("person = " + person);
}
```

>测试结果

![图片无法显示](https://i.imgur.com/LOjA1Mp.png)

>测试二：

修改applicationContext.xml中id为address的bean，修改其id为address1,再次执行测试结果如下：我们会发现并没有Address的bean被注入进来,这就是byName注入的缺点,通过setter方法匹配,在这里就是setAddress,所以,只会在IOC容器里寻到bean的id为address的bean,由于我们改名字的原因是无法注入的.

![图片无法显示](https://i.imgur.com/iiahZ87.png)


## 自动注入ByType ##

>applicationContext.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--address bean one-->
    <bean id="address1" class="edu.taotao.example.Address">
        <property name="country" value="China"/>
        <property name="province" value="SiChuan"/>
        <property name="area" value="JingRongTown"/>
    </bean>

    <!--address bean two-->
    <bean id="address2" class="edu.taotao.example.Address">
        <property name="country" value="China"/>
        <property name="province" value="BeiJing"/>
        <property name="city" value="ZhongGuanCun"/>
    </bean>

    <!--person自动注入address类型的bean，注入方式，通过类型-->
    <bean id="personByType" class="edu.taotao.example.Person"
          p:name="Jim"
          p:age="12"
          autowire="byType"/>


    <bean id="person" class="edu.taotao.example.Person" autowire="byName"/>

</beans>
```

>测试

```java
/**
 * 通过类型自动注入
 */
@Test
public void testAutowiredByType(){
    Person person = (Person) context.getBean("personByType");
    System.out.println("person = " + person);
}
```

>测试结果

![图片无法显示](https://i.imgur.com/DHhHPsP.png)

由图中黄色标记的部分可以看出，我们自动注入失败了，原因是IOC中应该有一个Address的bean，但是找到了两个，address1和address2，这就是通过类型注入的弊端，只允许IOC容器中存在一个类型为注入类型的bean。我们吧address2在IOC容器中去除，再进行测试，测试结果如下：

![图片无法显示](https://i.imgur.com/vnPnLsn.png)


## 总结 ##

bean的自动方式的优缺点

|bean的自动注入方式|优点|缺点|
|----|-----|-----|
|byName|你能保证目标bean的名称和属性名一致，就是优点|1. 如果不存在和setter方法相同的bean id，则无法注入.<br/> 2. 必须将目标bean的名称和属性名设置的完全相同|
|byType|确认IOC容器中只存在注入type只有一个时可以采用|无法确保要注入的bean在IOC容器中只有一个，Spring无法判定，也就无法注入了|
|constructor|-|当bean中存在多个构造器时，这中配置方式很复杂，不推荐使用|

>本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。