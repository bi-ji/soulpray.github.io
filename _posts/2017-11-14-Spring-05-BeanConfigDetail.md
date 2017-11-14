---
title: Spring系列_04_bean的配置细节
tags:
- Spring
- List
- Map
- Bean
categories: 
- Spring
---

# Bean的配置细节 #

在上一节中，我们主要是对bean进行了简单的配置，而在java中，对象众多，比如常用的List，Array，Set，Map，组合等多种数据类型在Spring中都是可以配置的，下面将介绍在Spring中如何配置下述数据
- Array
- List
- Set
- Map
- Properties
- util命名空间的使用
- p命名空间的使用
- 字面值
- 级联属性
- null值

>写在前面，本文所使用的bean将在下述代码清单中

>Phone.java

```java
@Data
@ToString
public class Phone {

    private String brand;

    private double price;

    private ArrayList<String> nums;

    private List<Contract> contacts;

    private Map<String, Object> gameList;

    private Set<String> photos;

    private String[] array;

}
```

>Contract.java

```java
@Data
@ToString
public class Contact {

    private String name;

    private int age;

    private String company;

}
```

>DataSource.java

```java
@Data
@ToString
public class DataSource {

    private Properties properties;

}
```

>TestDIDetail.java

```java
public class TestDIDetail {

    private ApplicationContext context;

    /**
     * 以后只用写测试就行了，@Before将会在测试执行用例执行前执行
     */
    @Before
    public void getApplicationContext(){
        context = new ClassPathXmlApplicationContext("applicationContext.xml");
    }
}
```

## Spring配置Array ##

>applicationContext.xml配置

```xml
<bean id="testArray" class="edu.taotao.example.Phone">
    <property name="array">
        <array>
            <value>elementOne</value>
            <value>elementTwo</value>
            <value>elementThree</value>
        </array>
    </property>
</bean>
```

>测试

```java
/**
 * 测试Array
 */
@Test
public void testArray(){
    Phone phone = (Phone) context.getBean("testArray");
    System.out.println(Arrays.asList(phone.getArray()));
}
```

>测试结果

![](https://i.imgur.com/rAF85AF.png)

## Spring配置List ##

>applicationContext.xml中配置

```xml
<!--测试List-->
<bean id="testList" class="edu.taotao.example.Phone">
    <!--Person中的contacts属性为List的值-->
    <property name="contacts">
        <list>
            <!--引用外部bean-->
            <ref bean="maleContact"/>
            <!--引用外部bean-->
            <ref bean="femaleContact"/>
            <!--内部bean，无法被外部引用-->
            <bean class="edu.taotao.example.Contact">
                <property name="name"value="unknow"/>
                <property name="age" value="12"/>
                <property name="company" value="unkonwCompany"/>
            </bean>
        </list>
    </property>
</bean>

<!--男性联系人-->
<bean id="maleContact" class="edu.taotao.example.Contact">
    <property name="name" value="male"/>
    <property name="age">
        <value>24</value>
    </property>
    <property name="company" value="maleCompany"/>
</bean>

<!--女性联系人-->
<bean id="femaleContact" class="edu.taotao.example.Contact">
    <property name="name" value="female"/>
    <property name="age">
        <value>20</value>
    </property>
    <property name="company" value="femaleCompany"/>
</bean>
```

>测试

```java
/**
 * 测试List注入
 */
@Test
public void testList(){
    Phone phone = (Phone) context.getBean("testList");
    System.out.println("phone.getContacts() = " + phone.getContacts());
}
```

>测试结果

![](https://i.imgur.com/LbM0AVX.png)

## Spring配置Set ##

>applicationContext.xml中配置

```xml
<!--测试Set-->
<bean id="testSet" class="edu.taotao.example.Phone">
    <property name="photos">
        <!--通过set标签注入值-->
        <set>
            <value>110.jpg</value>
            <value>110.jpg</value>
            <value>120.jpg</value>
            <value>119.jpg</value>
            <value>120.png</value>
        </set>
    </property>
</bean>
```

>测试

```java
/**
 * 测试Set
 */
@Test
public void testSet(){
    Phone phone= (Phone) context.getBean("testSet");
    System.out.println("phone.getPhotos() = " + phone.getPhotos());
}
```

>测试结果

![](https://i.imgur.com/nq4Y2eD.png)

## Spring配置Map ##

>applicationContext.xml中配置

```xml
<!--测试Map-->
<bean id="testMap" class="edu.taotao.example.Phone">
    <property name="gameList">
        <map>
            <!--key 和 value可以使用bean参照-->
            <entry key="lookPhoto" value="photo"/>
            <entry key="lookContact" value-ref="maleContact"/>
            <entry key="picture" value-ref="testSet"/>
        </map>
    </property>
</bean>
```

>测试

```java
/**
 * 测试Map
 */
@Test
public void testMap(){
    Phone phone = (Phone) context.getBean("testMap");
    System.out.println("phone.getGameList() = " + phone.getGameList());
}
```

>测试结果

![](https://i.imgur.com/xqGd8gQ.png)

## Spring配置Properties ##

>applicationContext.xml中配置

```xml
<!--测试Properties-->
<bean id="testProperties" class="edu.taotao.example.DataSource">
    <property name="properties">
        <props>
            <!--属性的键值对-->
            <prop key="url">jdbc:mysql:///test</prop>
            <prop key="driverClass">com.mysql.jdbc.Driver</prop>
            <prop key="username">root</prop>
            <prop key="password">123456</prop>
        </props>
    </property>
</bean>
```

>测试

```java
/**
 * 测试Properties
 */
@Test
public void testProperties(){
    DataSource dataSource = context.getBean(DataSource.class);
    System.out.println(dataSource);
}
```

>测试结果

![](https://i.imgur.com/iltYVDC.png)

## util命名空间的使用  ##
试想这样一个场景，对于一个公司来说，每个人的手机里都会有公司的电话号码，那么难道在每个人的手机的联系人中都添加这个联系人吗，这样不仅代码冗余，而且一旦公司的电话号码发生了变化，还要去改每个人的手机号，显然是不可取的，util命名空间可以用来解决这个问题，只要每个人的电话和公司电话号码，建立参考即可，util命名空间介绍

- 使用基本的集合标签定义集合时, 不能将集合作为独立的 Bean 定义, 导致其他 Bean 无法引用该集合, 所以无法在不同 Bean 之间共享集合.
- 可以使用 util schema 里的集合标签定义独立的集合 Bean. 需要注意的是, 必须在 <beans> 根元素里添加 util schema 定义：

>xmlns:util="http://www.springframework.org/schema/util
>http://www.springframework.org/schema/util
>http://www.springframework.org/schema/util/spring-util.xsd

这里只对List做一个简单的介绍，其实就是将之前在bean里面配置的Array，List，Set，Map独立拉取出来，然后在Bean和这些结构之间建立参考即可
### util:list的使用 ###

>applicationContext.xml配置

```xml
<!--utillist命名空间测试-->
<util:list id="commonList" value-type="edu.taotao.example.Contact">
    <ref bean="maleContact"/>
    <ref bean="femaleContact"/>
    <bean class="edu.taotao.example.Contact">
        <property name="company" value="company"/>
        <property name="name" value="myCompany"/>
        <property name="age" value="1"/>
    </bean>
</util:list>

<!--测试utillist-->
<bean id="testUtillist" class="edu.taotao.example.Phone">
    <property name="contacts" ref="commonList"/>
</bean>
```

>测试

```java
/**
 * utillist命名空间测试
 */
@Test
public void testUtillist(){
    Phone phone = (Phone) context.getBean("testUtillist");
    System.out.println("phone.getContacts() = " + phone.getContacts());
}
```

>测试结果

![](https://i.imgur.com/rgH9eGT.png)

## p命名空间的使用  ##
- 为了简化 XML 文件的配置，越来越多的 XML 文件采用属性而非子元素配置信息。
- Spring 从 2.5 版本开始引入了一个新的 p 命名空间，可以通过 <bean> 元素属性的方式配置 Bean 的属性。
- 使用 p 命名空间后，基于 XML 的配置方式将进一步简化

>applicationContext.xml配置

```xml
<!--测试p命名空间-->
<bean id="testP" class="edu.taotao.example.Contact" p:age="12" p:name="testP" p:company="testPCompany"/>
```

>测试

```java
/**
 * 测试p命名空间
 */
@Test
public void testP(){
    Contact contact = (Contact) context.getBean("testP");
    System.out.println("contact = " + contact);
}
```

>测试结果

![](https://i.imgur.com/leO5K8j.png)

## 字面值 ##

- 字面值：主要是用来处理特殊符号的，比如配置一个字符串<hello>，可用字符串表示的值，可以通过 <value> 元素标签或 value 属性进行注入。
- 基本数据类型及其封装类、String 等类型都可以采取字面值注入的方式
- 若字面值中包含特殊字符，可以使用 <![CDATA[]]> 把字面值包裹起来。

>applicationContext.xml配置


```xml
<!--测试字面值-->
<bean id="testCDATA" class="edu.taotao.example.Contact">
    <property name="age" value="24"/>
    <property name="name">
        <value><![CDATA[<Paul>]]></value>
    </property>
    <property name="company">
        <value><![CDATA[<<CDATA Company>>]]></value>
    </property>
</bean>
```

>测试

```java
/**
 * 测试字面值
 */
@Test
public void testCDATA(){
    Contact contact = (Contact) context.getBean("testCDATA");
    System.out.println("contact = " + contact);
}
```

>测试结果

![](https://i.imgur.com/nEgclhz.png)

## 级联属性 ##

>applicationContext.xml配置

```xml
<!--测试级联属性-->
<bean id="testCasecadeProperty" class="edu.taotao.example.Phone">
    <!--联系人-->
    <property name="favorite" ref="femaleContact"/>
    <!--级联给联系人赋值-->
    <property name="favorite.name" value="angel"/>
    <property name="favorite.company" value="movie"/>
</bean>
```

>测试

```java
/**
 * 测试级联属性
 */
@Test
public void testCasecadeProperty(){
    Phone phone = (Phone) context.getBean("testCasecadeProperty");
    System.out.println(phone.getFavorite());
}
```

>测试结果

![](https://i.imgur.com/vleIeBL.png)

## null值 ##

>applicationContext.xml配置

```xml
<!--测试null值-->
<bean id="testNull" class="edu.taotao.example.Phone">
    <!--联系人,这是联系人的属性是有值的，设置为null值-->
    <property name="favorite" ref="femaleContact"/>
    <!--级联给联系人赋值为空值-->
    <property name="favorite.name">
        <null/>
    </property>
    <property name="favorite.company">
        <null/>
    </property>
</bean>
```

>测试

```java
/**
 * 测试空值
 */
@Test
public void testNull(){
    Phone phone = (Phone) context.getBean("testNull");
    System.out.println(phone.getFavorite());
}
```

>测试结果

![](https://i.imgur.com/rbpKzil.png)

## 总结 ##
>Spring中的bean的配置细节到这就成功结束了。本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。