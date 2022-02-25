1.在切面类（为切点服务的类）前用@Aspect注释修饰，声明为一个切面类。

　　2.用@Pointcut注释声明一个切点，目的是为了告诉切面，谁是它的服务对象。（此注释修饰的方法的方法体为空，不需要写功能比如 public void say(){};就可以了，方法名可以被候命的具体服务功能所以引用，它可以被理解为切点对象的一个代理对象方法）

　　3.在对应的方法前用对应的通知类型注释修饰，将对应的方法声明称一个切面功能，为了切点而服务

　　4.在spring配置文件中开启aop注释自动代理。如：<aop:aspectj-autoproxy/>

这样讲可能还是很抽象，那么，废话不多说，我们代码说话，代码如下：

pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>file_action</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.13</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.13</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.3.13</version>

        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.3.13</version>
        </dependency>
    </dependencies>

</project>
```

骑士类：（看过上一篇内容的就知道骑士是什么东西了，嘿嘿嘿）

```java
package com.cjh.aop2;

import org.springframework.stereotype.Component;

/**
 * @author Caijh
 *
 * 2017年7月11日 下午3:53:19
 */
@Component("knight")
public class BraveKnight {
    public void saying(){
        System.out.println("我是骑士..（切点方法）");
    }
}
```

切面类：（注释主要在这里体现）

```java
package com.cjh.aop2;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

/**
 * @author Caijh
 * email:codecjh@163.com
 * 2017年7月12日 上午9:31:43
 */
/**
 * 注解方式声明aop
 * 1.用@Aspect注解将类声明为切面(如果用@Component("")注解注释为一个bean对象，那么就要在spring配置文件中开启注解扫描，<context:component-scan base-package="com.cjh.aop2"/>
 *      否则要在spring配置文件中声明一个bean对象)
 * 2.在切面需要实现相应方法的前面加上相应的注释，也就是通知类型。
 * 3.此处有环绕通知，环绕通知方法一定要有ProceedingJoinPoint类型的参数传入，然后执行对应的proceed()方法，环绕才能实现。
 */
@Component("annotationTest")
@Aspect
public class AnnotationTest {
    //定义切点
    @Pointcut("execution(* *.saying(..))")
    public void sayings(){}
    /**
     * 前置通知(注解中的sayings()方法，其实就是上面定义pointcut切点注解所修饰的方法名，那只是个代理对象,不需要写具体方法，
     * 相当于xml声明切面的id名，如下，相当于id="embark",用于供其他通知类型引用)
     * <aop:config>
        <aop:aspect ref="mistrel">
            <!-- 定义切点 -->
            <aop:pointcut expression="execution(* *.saying(..))" id="embark"/>
            <!-- 声明前置通知 (在切点方法被执行前调用) -->
            <aop:before method="beforSay" pointcut-ref="embark"/>
            <!-- 声明后置通知 (在切点方法被执行后调用) -->
            <aop:after method="afterSay" pointcut-ref="embark"/>
        </aop:aspect>
       </aop:config>
     */
    @Before("sayings()")
    public void sayHello(){
        System.out.println("注解类型前置通知");
    }
    //后置通知
    @After("sayings()")
    public void sayGoodbey(){
        System.out.println("注解类型后置通知");
    }
    //环绕通知。注意要有ProceedingJoinPoint参数传入。
    @Around("sayings()")
    public void sayAround(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("注解类型环绕通知..环绕前");
        pjp.proceed();//执行方法
        System.out.println("注解类型环绕通知..环绕后");
    }
}
```

