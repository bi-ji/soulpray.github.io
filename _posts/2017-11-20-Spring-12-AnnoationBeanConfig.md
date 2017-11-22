---
title: Spring系列_12_基于注解的bean配置
tags:
- Spring
- annoation
categories: 
- Spring
---

# 基于注解配置Bean #

## 什么是组件扫描 ##

说到基于注解的配置，就不得不提到component scanning（组件扫描）：Spring能够从classpath(类编译后的路径)下自动扫描，侦测和实例化具有特定注解的组件，这些组件包括使用下列注解标识的。

- @Component:基本注解，标识了一个受Spring管理的组件
- @Respository:标识持久层组件
- @Service:标识服务层（业务层组件）
- @Controller:标识展现层组件

对于扫描到的组件，<font color=red>Spring有默认的命名策略：</font>使用非限定类名，然后第一个字母小写，也可以在注解中通过value属性值标识组件的名称。

## 组件扫描的配置 ##

- 当在组件类上使用了特定的注解之后，还需要在Spring中配置扫描包的位置，即使用&lt;context:component-scan>,其有两个属性
	- base-package 属性指定一个需要扫描的基类包，Spring 容器将会扫描这个基类包里及其子包中的所有类. 
	- 当需要扫描多个包时, 可以使用逗号分隔.
	- 如果仅希望扫描特定的类而非基包下的所有类，可使用 resource-pattern 属性过滤特定的类，
		- &lt;context:include-filter> 子节点表示要包含的目标类
		- &lt;context:exclude-filter> 子节点表示要排除在外的目标类
		- &lt;context:component-scan> 下可以拥有若干个 &lt;context:include-filter> 和&lt;context:exclude-filter> 子节点
		- &lt;context:include-filter> 和 &lt;context:exclude-filter> 子节点支持多种类型的过滤表达式

过滤表达式

|类别|示例|说明|
|----|-----|-----|
|annotation|a.b.c.XxxAnnotation|所有标注了XxxAnnotation的类。该类型采用目标类是否标注了某个注解进行过滤|
|assinable|ab.c.XxxService|所有集成或扩展XxxService的类。该类型采用目标类是哦负集成或扩展某个特定类进行过滤|
|aspectj|a.b..*Service+|所有类名以Service结束的类及继承或扩展它们的类。该类型采用AspejctJ表达式进行过滤|
|regex|a\b\c\..*|所有a.b.c包下的类。该类型采用正则表达式根据类的类名进行过滤|
|custom|a.b.XxxTypeFilter|采用XxxTypeFilter通过代码的方式定义过滤规则。该类必须实现org.springframework.core.type.TypeFilter接口|

## 本文代码清单 ##

>项目目录

![](https://i.imgur.com/3C0ooHl.png)

>User.java

```java
@Data
public class User {

    private Long id;

    private String name;

}
```

>UserController.java

```java
//如果没有指定value属性，则在Spring中bean的名字默认是首字母小写
@Controller
public class UserController {

    @Autowired
    private IUserService userService;

    /**
     * 保存用户
     * @param user
     */
    public void saveUser(User user){
        System.out.println("UserController --> saveUser");
        userService.saveUser(user);
    }
}
```
>IUserService.java

```java
public interface IUserService {

    /**
     * 添加用户
     * @param user 用户
     * @return 用户信息
     */
    Long saveUser(User user);
}
```

>UserServiceImpl.java

```java
@Service
public class UserServiceImpl implements IUserService{

    @Autowired
    private UserRepository userRepository;

    /**
     * 添加用户
     *
     * @param user 用户
     * @return 用户信息
     */
    @Override
    public Long saveUser(User user) {
        System.out.println("UserService --> saveUser");
        return userRepository.saveUser(user);
    }
}
```

>UserRepository.java

```java
@Repository
public class UserRepository {

    /**
     * 保存用户信息
     * @param users 用户
     * @return 用户id
     */
    public Long saveUser(User users){
        System.out.println("UserRepository --> saveUser");
     return 1L;
    }

}
```

>applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.1.xsd ">

</beans>
```

>TestApplication.java

```java
public class TestApplication {

    private ApplicationContext context;

    /**
     * 初始化IOC容器
     */
    @Before
    public void getApplicationContext(){
        context = new ClassPathXmlApplicationContext("applicationContext.xml");
    }
}
```

## 扫描所有注解标识的类 ##

>配置applicationContext.xml

本测试将会测试注入所有的被Spring注解标识的类，在建立好上述的测试环境之后，便可在applicationContext.xml中添加组件扫描，需要引入context命名空间

```xml
<!--组件扫描，将会扫描 base-package 定义的包及其子包下的组件-->
<context:component-scan base-package="edu.taotao.example"/>
```

>测试

```java
/**
 * 测试全部自动注入
 */
@Test
public void testAnnoAutowired() {
    UserController userController = (UserController) context.getBean("userController");
    User user = new User();
    userController.saveUser(user);
}
```

>测试结果

![](https://i.imgur.com/5C9TNOe.png)

## 注解说明 ##

在Spring中，可以通过三种方式来装配注解配置的bean

- @Autowired 注解自动装配具有兼容类型的单个 Bean属性
	- 	构造器, 普通字段(即使是非 public), 一切具有参数的方法都可以应用@Authwired 注解
	- 	默认情况下, 所有使用 @Authwired 注解的属性都需要被设置. 当 Spring 找不到匹配的 Bean 装配属性时, 会抛出异常, 若某一属性允许不被设置, 可以设置 @Authwired 注解的 required 属性为 false
	- 	默认情况下, 当 IOC 容器里存在多个类型兼容的 Bean 时, 通过类型的自动装配将无法工作. 此时可以在 @Qualifier 注解里提供 Bean 的名称. Spring 允许对方法的入参标注 @Qualifiter 已指定注入 Bean 的名称
	- 	 @Autowired 注解也可以应用在数组类型的属性上, 此时 Spring 将会把所有匹配的 Bean 进行自动装配.
	- 	@Autowired 注解也可以应用在集合属性上, 此时 Spring 读取该集合的类型信息, 然后自动装配所有与之兼容的 Bean. 
	- 	@Autowired 注解用在 java.util.Map 上时, 若该 Map 的键值为 String, 那么 Spring 将自动装配与之 Map 值类型兼容的 Bean, 此时 Bean 的名称作为键值
- @Resource 注解要求提供一个 Bean 名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为 Bean 的名称
- @Inject 和 @Autowired 注解一样也是按类型匹配注入的 Bean， 但没有 reqired 属性
- 建议使用 @Autowired 注解

## 排除指定类型的注解 ##

>配置applicationContext.xml

这里将会排除@Repository注解，所以会报错，报找不到bean useRepository

```xml
<!--组件扫描，将会扫描 base-package 定义的包及其子包下的组件-->
<context:component-scan base-package="edu.taotao.example">
    <!--将会排除包下的所有名为Repository的注解-->
    <context:exclude-filter type="annotation"
    	expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

>测试

```java

/**
 * 测试排除@Repositroy注解
 */
@Test
public void testExcludeRepository(){
    UserController userController = (UserController) context.getBean("userController");
    User user = new User();
    userController.saveUser(user);
}
```

>测试结果

![](https://i.imgur.com/j3O4EX7.png)

由图中黄色框框中所示，确实是按照我们预期的那样。没有找到UseRepository bean

## 等价的xml配置 ##

要先去除所有的注解，并添加setter方法，将会等价于下列xml配置

>为UserServiceImpl.java添加setter方法


```java
//@Service
public class UserServiceImpl implements IUserService{

//    @Autowired
    private UserRepository userRepository;

    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    /**
     * 添加用户
     *
     * @param user 用户
     * @return 用户信息
     */
    @Override
    public Long saveUser(User user) {
        System.out.println("UserService --> saveUser");
        return userRepository.saveUser(user);
    }
}
```

>为UserRepository.java去除注解

```java
//@Repository
public class UserRepository {

    /**
     * 保存用户信息
     * @param users 用户
     * @return 用户id
     */
    public Long saveUser(User users){
        System.out.println("UserRepository --> saveUser");
     return 1L;
    }

}
```

>为UserController.java去除注解，添加setter方法

```java
//@Controller
public class UserController {

//    @Autowired
    private IUserService userService;

    public void setUserService(IUserService userService) {
        this.userService = userService;
    }

    /**
     * 保存用户
     * @param user
     */
    public void saveUser(User user){
        System.out.println("UserController --> saveUser");
        userService.saveUser(user);
    }
}
```
>配置applicationContext.xml

```xml
<!--等价xml-->
<!--配置userRepository-->
<bean id="userRepository" class="edu.taotao.example.dao.UserRepository"/>
<!--配置userService-->
<bean id="userService" class="edu.taotao.example.service.UserServiceImpl">
    <!--注入userRepository-->
    <property name="userRepository" ref="userRepository"/>
</bean>
<!--配置userController-->
<bean id="userController" class="edu.taotao.example.controller.UserController">
    <!--注入userService-->
    <property name="userService" ref="userService"/>
</bean>
```

>测试

```java
/**
 * 测试等价xml
 */
@Test
public void testEqualXmlConfig() {
    UserController userController = (UserController) context.getBean("userController");
    User user = new User();
    userController.saveUser(user);
}
```

>测试结果

![](https://i.imgur.com/QXAhGby.png)

## 总结 ##
>Spring中的基于注解的注入到这就成功结束了。

>本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。