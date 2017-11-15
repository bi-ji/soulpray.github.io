---
title: Spring系列_07_Bean之间的关系
tags:
- Spring
- relation
- Bean
categories: 
- Spring
---

# Bean之间的关系 #

在上一节中我们讲了很多bean自动注入方式，这节我们会将Bean之间的关系

- 继承
- 依赖

>写在前面，本文中所使用的代码清单

>Shape.java

```java
@Data
public class Shape {

    private String color;

    private Double area;

    private Double width;

    private Double height;

	private Pen pen;    
}
```

>TestBeanRelation.java

```java
public class TestBeanRelation {

    private ApplicationContext context;

    @Before
    public void getApplicationContext(){
        context = new ClassPathXmlApplicationContext("applicationContext.xml");
    }
    
}
```

## 继承 ##

- Spring 允许继承 bean 的配置, 被继承的 bean 称为父 bean. 继承这个父 Bean 的 Bean 称为子 Bean
- 子 Bean 从父 Bean 中继承配置, 包括 Bean 的属性配置
- 子 Bean 也可以覆盖从父 Bean 继承过来的配置
- 父 Bean 可以作为配置模板, 也可以作为 Bean 实例. 若只想把父 Bean 作为模板, 可以设置 <bean> 的abstract 属性为 true, 这样 Spring 将不会实例化这个 Bean
- 并不是 <bean> 元素里的所有属性都会被继承. 比如: autowire, abstract 等.
- 也可以忽略父 Bean 的 class 属性, 让子 Bean 指定自己的类, 而共享相同的属性配置. 但此时 abstract 必须设为 true

### 子Bean继承父Bean的配置 ###

>applicationContenxt.xml配置

```xml
<!--创建一个正方形-->
<bean id="square" class="edu.taotao.example.Shape">
    <!--注入值到属性中-->
    <property name="height" value="10"/>
    <property name="width" value="10"/>
    <property name="area" value="100"/>
    <property name="color" value="red"/>
</bean>

<!--创建一个上面正方形的拷贝，执行parent为上述bean的id即可-->
<bean id="squareCopy" parent="square"/>
```

>测试

```java
@Test
public void testBeanCopy(){
    Shape shape = (Shape) context.getBean("square");
    System.out.println("shape = " + shape);
    Shape shapeCopy = (Shape) context.getBean("squareCopy");
    System.out.println("shapeCopy = " + shapeCopy);
}
```

>测试结果

![](https://i.imgur.com/VwxEbNA.png)

### 子Bean覆盖父Bean的配置 ###

>applicationContenxt.xml配置

```xml
<!--创建一个上面正方形的拷贝，执行parent为上述bean的id即可，然后覆盖其属性,加倍宽和高-->
<bean id="squareOverride" parent="square">
    <!--这里覆盖了宽和高，面积，并没有覆盖颜色-->
    <property name="height" value="20"/>
    <property name="width" value="20"/>
    <property name="area" value="400"/>
</bean>
```

>测试

```java
@Test
public void testBeanOverrideProperty(){
    Shape shape = (Shape) context.getBean("square");
    System.out.println("shape = " + shape);
    Shape shapeOverride = (Shape) context.getBean("squareOverride");
    System.out.println("shapeOverride = " + shapeOverride);
}
```

>测试结果

![](https://i.imgur.com/ihV1KEj.png)

### 模板Bean ###

>applicationContenxt.xml配置

```xml
<!--抽象bean,做模版bean-->
<bean id="abstractTriangle" class="edu.taotao.example.Shape" abstract="true">
    <property name="height" value="4"/>
    <property name="width" value="3"/>
    <property name="area" value="6"/>
    <property name="color" value="blue"/>
</bean>

<!--创建模版抽象bean的实例bean-->
<bean id="templateBeanInstance" class="edu.taotao.example.Shape" parent="abstractTriangle"/>
```

>测试

```java
/**
 * 测试实例化抽象bean
 */
@Test
public void testAbstractParentBean(){
    Shape templateInstance = (Shape) context.getBean("templateBeanInstance");
    System.out.println("templateInstance = " + templateInstance);
    Shape shape = (Shape) context.getBean("abstractTriangle");
    System.out.println("shape = " + shape);
}
```

>测试结果

![无法显示图片](https://i.imgur.com/yjYjvK6.png)

由测试结果可以看到，被指定abstract属性为true的bean，无法实例化，而有模版创建的bean可以实例化。

## 依赖 ## 

Spring 允许用户通过 depends-on 属性设定 Bean 前置依赖的Bean，前置依赖的 Bean 会在本 Bean 实例化之前创建好
如果前置依赖于多个 Bean，则可以通过逗号，空格或的方式配置 Bean 的名称

### 被依赖的bean未被创建 ###

>applicationContenxt.xml配置

```xml
<!--测试bean的依赖，depend-on中填写bean的id-->
<bean id="dependPenShape" class="edu.taotao.example.Shape" depends-on="pen">
    <property name="color" value="yellow"/>
    <property name="width" value="20"/>
    <property name="height" value="20"/>
    <property name="area" value="400"/>
</bean>
```

>测试

```java
/**
 * 测试bean的依赖关系
 */
@Test
public void testBeanDependsOn(){
    Shape dependPenShape = (Shape) context.getBean("dependPenShape");
    System.out.println("dependPenShape = " + dependPenShape);
}
```

>测试结果

![](https://i.imgur.com/QRYIi1S.png)

由图中框的错误信息可看出，IOC容器中并没有pen这个bean

### 被依赖的bean被创建 ###

>applicationContenxt.xml配置

```xml
<!--测试bean的依赖，depend-on中填写bean的id-->
<bean id="dependPenShape" class="edu.taotao.example.Shape" depends-on="pen">
    <property name="color" value="yellow"/>
    <property name="width" value="20"/>
    <property name="height" value="20"/>
    <property name="area" value="400"/>
</bean>

<!--被依赖的Pen bean-->
<bean id="pen" class="edu.taotao.example.Pen"/>
```

>测试

```java
@Test
public void testBeanDependsOn(){
    Shape dependPenShape = (Shape) context.getBean("dependPenShape");
    System.out.println("dependPenShape = " + dependPenShape);
}
```

>测试结果

![](https://i.imgur.com/OVESD1t.png)

由测试结果可知，在IOC容器中有Pen类型且id为pen的情况下，depends-on才会生效。

## 总结 ##

>本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。