---
title: spring依赖注入原理
date: 2019-1-17 09:45:58
tag: Spring详解
category: Java
---
### 前言：

- JavaBean： 即POJO的同义词

- Spring通过应用上下文（ Application Context） 装载bean的定义并把它们组装起来。 **Spring应用上下文全权负责对象的创建和组装**。 Spring自带
了多种应用上下文的实现， 它们之间主要的区别仅仅在于如何加载配置。
- spring通过**依赖注入和面向接口**实现松耦合
- 传统的做法：每个对象负责管理与自己相互协作的对象的引用。
- spring的做法：在类中定义依赖关系，然后通过配置，在运行时实例化（new）这些依赖。即：spring干的活就是根据定义好的依赖关系，使用相应的bean构造方法，将构造好的bean（实例化的对象）注入描述依赖的地方。

### spring装配bean的三种方式：
##### 一、 在XML中进行显式配置。

1. **创建Bean**：

    > <bean id="beanName" class="com.***.***"> 
    补充：内部调用该class的无参构造函数创建对象

2. **装配Bean**：
    - 构造器注入：
        - 引用类型注入：
            > <constructor-arg ref="beanName"> 
        - 字面量注入：
            > <constructor-arg value="someValue">
        - 集合注入：
            ```
            <constructor-arg>
                <list>
                    <value>****</value>
                    <value>****</value>
                    ... ...
                </list>
            ==========================
            或者
                <set>
                    <value>****</value>
                    <value>****</value>
                    ... ...
                    </set>
            </constructor-arg>
            ```

    - setter方法属性注入：
        - 注入引用类型：
            > <property name="beanName" ref="beanId" />
            补充：通过setter方法注入的
        - 注入字面量：
            > <property name="" value="" />
        - 注入集合：
            ```
            <property name="">
                <list>
                    <value>...</value>
                </list>
            </property>
            ```
           
##### 二、在Java中进行显式配置。
1. **创建Bean**：给一个返回对象的方法加上@Bean注解

2. **装配Bean**：
    - 在需要其他Bean的时候引用创建那个Bean的方法
    
    - 使用@AutoWired注解，可以放在任何能使用@AutoWired注解的地方
        
    > 补充：（自动单例：看起来， CompactDisc是通过调用sgtPeppers()得到的，但情况
并非完全如此。因为sgtPeppers()方法上添加了@Bean注解，
Spring将会拦截所有对它的调用，并确保直接返回该方法所创建的
bean，而不是每次都对其进行实际的调用。）

##### 三、隐式的bean发现机制和自动装配（注解+组件扫描+自动装配）
1. **创建bean**：容器会自动把扫描到的带有注解的类实例化
2. **装配bean**：@AutoWired注解，容器自动装配

### 关于注入方式，即Bean的注入地点：共有三种
1. 构造器注入；
2. 设值注入（setter方式注入）；
3. Feild方式注入（字段方式注入）。

### 使用哪种方式？

1. 强依赖使用构造器注入
2. 弱依赖使用setter注入
3. 不会动态修改某个bean本身的时候，使用Feild注入。