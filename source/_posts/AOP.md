---
title: AOP
date: 2021-01-21 19:01:49
tags: Spring5
---

![](https://gitee.com/li_da_ben_shi/MyPicture/raw/master/img/20210121190919.jpeg)

<!--more-->

# AOP

## 一、基本概念

面向切面编程，它主要是对业务各个逻辑降低它们之间的耦合度，分离业务。

通俗描述是：不通过修改源代码方式，在主干功能里面添加新功能

（使用的设计模式就是**装饰者模式**）

##  二、底层原理



AOP底层使用的是动态代理。

### （一）有接口，使用JDK动态代理

这个很简单，就是对接口的实现。

这个实现类，就是代理对象

### （二）无接口，使用CGLIB动态代理

这个就是创建类的子类，重写方法或者添加方法，创建增强的逻辑。

而这个子类就是代理对象

（代码略）

## 三、AOP中的一些术语

1. 连接点：类里面有哪些可以被增强就叫做连接点
2. 切入点：实际被真正增强的方法
3. 通知（增强）：实际被增强的逻辑部分。通知分很多种，分为前置通知（方法前执行）、后置通知（方法后执行）、环绕通知（方法前、后执行）、异常通知（方法发生异常执行）、最终通知（方法最终执行（finally））
4. 切面：将通知应用到切入点

## 四、AOP的操作

### （一）准备工作

`Spring` 框架一般都是基于 `AspectJ` 实现 `AOP` 操作

`AspectJ` 不是 `Spring` 组成部分，独立 `AOP` 框架，一般把 `AspectJ` 和 `Spirng` 框架一起使用，进行 `AOP`操作

基于 `AspectJ` 实现 AOP 操作：

 （1）基于 `xml` 配置文件实现 

 （2）基于**注解**方式实现（使用）

然后，写切入点表达式（知道对哪个类里面的哪个方法进行增强）

表达式语法结构：

`execution([权限修饰符] [返回类型] [类全路径] [方法名称]([参数列表]) )`

注意：权限修饰符可以不写

​			返回类型如果追求简单可以写为 * 

​			参数列表也可以写为（..）

如：

​	对`com.atguigu.dao.BookDao` 类里面的 add进行增强 

`execution(* com.atguigu.dao.BookDao.add(..))`

​	对 `com.atguigu.dao.BookDao` 类里面的所有的方法进行增强 

`execution(* com.atguigu.dao.BookDao.* (..))`

​	对 `com.atguigu.dao`包里面所有类，类里面所有方法进行增强

 `execution(* com.atguigu.dao.*.* (..))`

### （二）基于注解的AspectJ

1. 在 spring配置文件中，开启注解扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd

    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.IQIUM.SpringTest05.AspectJ"></context:component-scan>

</beans>
```

2. 使用注解创建 User 和 UserProxy 对象

```java
import org.springframework.stereotype.Component;

@Component
public class User {
    public void add(){
        System.out.println("add......");
    }
}
```

```java
@Component
@Aspect
public class UserProxy {

}
```

3. 在增强类上面添加注解 @Aspect

代码同上

4. 在 spring 配置文件中开启生成代理对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd

    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.IQIUM.SpringTest05.AspectJ"></context:component-scan>
    <!--    开启代理    -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>
```

5. 配置不同类型的通知

在增强类的里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class UserProxy {

    @Before(value = "execution(* com.IQIUM.SpringTest05.AspectJ.User.add(..))")
    public void before() {
        System.out.println("before.....");
    }

    @After("execution(* com.IQIUM.SpringTest05.AspectJ.User.add(..))")
    public void after(){
        System.out.println("after......");
    }

    @Around("execution(* com.IQIUM.SpringTest05.AspectJ.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("Around执之前......");

        //执行被增强的方法
        proceedingJoinPoint.proceed();

        System.out.println("Around执之后......");
    }
}
```

**补充1：相同的切入点抽取**

1. 抽取相同切入点

```java
@Pointcut(value = "execution(* com.IQIUM.SpringTest05.AspectJ.User.add(..))")
public void pointCut(){

}
```

2. 使用

```java
@Before(value = "pointCut()")
public void before() {
    System.out.println("before.....");
}
```

**补充2：有多个增强类多同一个方法进行增强，设置增强类优先级**

在增强类上面添加注解 `@Order(数字类型值)`，**数字类型值越小优先级越高**

```java
@Component
@Aspect
@Order(1)
public class PersonProxy {
    @Pointcut(value = "execution(* com.IQIUM.SpringTest05.AspectJ.User.add(..))")
    public void pointCut() {
    }

    @Before(value = "pointCut()")
    public void before() {
        System.out.println(" 人before.....");
    }
}
```

