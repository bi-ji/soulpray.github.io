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

- 当在组件类上使用了特定的注解之后，还需要在Spring中配置扫描包的位置，即使用<context:component-scan>,其有两个属性
	- base-package 属性指定一个需要扫描的基类包，Spring 容器将会扫描这个基类包里及其子包中的所有类. 
	- 当需要扫描多个包时, 可以使用逗号分隔.
	- 如果仅希望扫描特定的类而非基包下的所有类，可使用 resource-pattern 属性过滤特定的类，
		- <context:include-filter> 子节点表示要包含的目标类
		- <context:exclude-filter> 子节点表示要排除在外的目标类
		- <context:component-scan> 下可以拥有若干个 <context:include-filter> 和<context:exclude-filter> 子节点
		- <context:include-filter> 和 <context:exclude-filter> 子节点支持多种类型的过滤表达式

过滤表达式

|类别|示例|说明|
|----|-----|-----|
|annotation|qwe|qwe|
|assinable|qwe|qwe|
|aspectj|qwe|qwe|
|regex|qwe|qwe|
|custom|qwe|qwe|



## 总结 ##
>Spring中的构造器依赖注入到这就成功结束了。本教程基于Spring4.3.0,代码地址：[https://github.com/soulpray/SpringFamily.git](https://github.com/soulpray/SpringFamily.git)，目录中的Spring4.X。