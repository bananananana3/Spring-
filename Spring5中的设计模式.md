# 1.Spring入门

### Spring简化了应用开发：

1.所有应用程序类都是简单的Pojo类------Spring没有侵入性，在大多数情况下，它不要求扩展框架类或实现框架接口。

2.Spring应用程序可以不需要JavaEE应用程序服务器，它们也可以部署在一个节点上；

3.可以使用事务管理在数据库事务中执行方法，而不依赖于任何第三方事务API;

4.使用Spring,可以将Java方法作为请求处理程序方法或远程方法，如Servlet API的服务方法，但不需要处理Servlet容器的Servlet API。

5.可以将Java方法作为请求处理方法或远程方法，就像ServletAPI的service()方法一样，而不依赖于处理Servlet容器的ServletAPI。Spring可以使用本地的方法作为消息处理方法，而在应用程序中不适用Java消息服务（JMS）API。

6.Spring能够使用本地Java方法作为管理操作，不用在应用程序中使用Java管理拓展（JMX）API.

7.Spring作为应用程序对象的容器。因此，对象不用担心寻找并建立彼此的联系。

8.Spring实例化 Bean并将对象的依赖项注入应用程序，它是Beans的生命周期管理器。





### Spring使用以下策略使Java开发变得更容易测试：

Spring使用Pojo模式进行轻量级和低侵入式企业级应用的开发；

使用依赖注入模式来实现松耦合，实现面向系统接口编程；

使用装饰者模式和代理模式实现通过切面和共同约定来声明式编程；

使用模板方法消除样版代码；

### 依赖注入的好处是相同的，无论使用的是基于XML还是基于Java的配置：

依赖注入促进松耦合，使其可以去掉硬编码，实现P2I的最佳时间，可以在应用程序之外使用工厂模式及其内置的热拔插和可拔插实现。

DI模式促进面向对象编程的组合模式，而不是继承编程。



### 使用Spring通过工厂模式管理Bean

两种不同类型的Spring容器：

1.Bean工厂

2.应用上下文（ApplicationContext）





### Spring容器中Bean的生命周期如下：

（1）加载所有Bean的所有定义，创建有序图；

（2）实例化运行BeanFactoryPostProcessors(可以在这里更新Bean的定义)

（3）实例化每个Bean

(4) Spring将值和引用注入Bean属性中。

（5）如果Bean实现了BeanNameAware接口，就将Bean的ID传递给setBeanName()方法。

（6）如果Bean实现了BeanFactoryAware接口，就将Bean工厂本身的引用传递给setBeanFactory()方法。

（7）如果Bean实现了ApplicationContextAware接口，就将应用上下文本身的引用传递个setApplicationContext（）方法。

（8）BeanPostProcessor是一个接口，Spring允许用自定义的Bean实现它，而Spring容器会调用postProcessBeforeInitialization()方法在初始化修改Bean的实例。

（9）如果Bean实现了InitializingBean接口，Spring会调用afterpropertiesSet()方法来初始化处理或加载申请的应用资源，而这取决于指定的初始化方法。这里有其他方法来实现这一步。例如，可以使用init方法的 < Bean>标签、“@Bean”注解的initMethod属性和“JSR250's@PostConstruct"注解。 

（10）BeanPostProcessor是一个接口，Spring允许自定义的Bean实现它，而Spring容器会调用postProcessAfterInitialization()方法在初始化之后修改Bean的实例。

（11）此时Bean已经准备好了，应用程序可以通过应用上下文的getBean方法访问Bean，并且Bean在应用上下文中保持活动状态，直到通过调用close()方法关闭它。

（12）如果Bean实现了DisposibleBean接口，Spring会调用destoy()方法来销毁处理或清理申请的应用资源。这里有其他方法来实现这一步。例如，可以使用destroy方法的<Bean >标签、“@Bean”注解的destroyMethod属性和“JSR250's@preDestroy"注解。 

### 依赖注入的类型：

基于构造方法的注入。

基于Setter的依赖注入。

```java
基于构造函数的依赖注入

public class UserServiceImpl implents UserService{
private UserDao userDao;


@Autowire
public UserServiceImpl(UserDao userDao){
this.userDao = userDao;
}
}

基于Setter的依赖注入

public class UserServiceImpl implents UserService{
private UserDao userDao;


@Autowire
public serUserDao(UserDao userDao){
this.userDao = userDao;
}
}

基于字段的依赖注入

public class UserServiceImpl implents UserService{
@Autowire
private UserDao userDao;
}
```

#### 基于字段的依赖注入缺点

**对于有final修饰的变量不好使**

Spring的IOC对待属性的注入使用的是set形式，但是final类型的变量在调用class的构造函数的这个过程当中就得初始化完成，这个是**基于字段的依赖注入**做不到的地方．只能使用**基于构造函数的依赖注入**的方式.

**掩盖单一职责的设计思想**

我们都知道在OOP的设计当中有一个**单一职责思想**，如果你采用的是**基于构造函数的依赖注入**的方式来使用Spring的IOC的时候，当你注入的太多的时候，这个构造方法的参数就会很庞大，类似于下面.

当你看到这个类的构造方法那么多参数的时候，你自然而然的会想一下:**这个类是不是违反了单一职责思想?**.但是使用**基于字段的依赖注入**不会让你察觉，你会很沉浸在@Autowire当中





#### 使用构造方法注入模式，有以下优点：

基于构造方法的依赖项注入更适合于强制性依赖关系，并且它生成了一个强依赖关系。

基于构造方法的依赖注入提供了比其他方法更紧凑的代码结构。

支持构造方法参数传递给依赖类的依赖项进行测试。

支持使用不可变对象，并且不破坏信息隐藏原则。（支持final对象的注入）

#### setter注入模式的优点

比构造方法注入更易读

解决了应用程序中循环引用的问题

允许尽可能晚创建待价高昂的资源或服务，并且仅在需要时才注入。

不需要更改构造方法。而是通过公开暴露的属性赋值来传递。

#### setter注入模式的缺点

在setter注入模式中安全性较低，因为它可以被覆盖。

基于setter的依赖注入不能提供与构造方法注入一样紧凑的代码结构。

使用setter注入时要小心，因为他不是必需的依赖项。

### 配置应用程序的元数据的三种方式：

1.基于Java配置得依赖注入模式----使用Java显式配置

@Configuration

@Bean

public TransferService transferService(){

return new TransferServiceImpl(accountRepository(),transferRepository());

}

@Bean

public   AccountRepository accountRepository(){

return new AccountRepository();

}

@Bean

public   TransferRepository transferRepository(){

return new TransferRepository();

}

2.基于注解配置得依赖注入模式------使用Bean发现和自动装配的隐式配置。

3.基于XML配置的依赖注入模式------使用XML显式配置。

@Configuration

@ComponentScan

public class Appconfig(){

}

可以在@ComponentScan

里指定要扫描的包：

@ComponentScan("com.pacet.patterningspring.chapter4.bankapp")

或

@ComponentScan(basePackage="com...")

如果有多个

@ComponentScan(basePackages={"com....","com..."})

Spring允许通过类或接口指定包扫描行为，而不是将包指定为@ComponentScan的basePackages属性的简单字符串值：

@ComponentScan(basePackageClasses={TransfetService.class,AccountRepository.class})

### 用抽象工厂解决依赖关系

如果想为Bean添加“if...else”条件配置，可以这样做；如果使用Java配置，还可以添加一些自定义逻辑。但是在XML配置的情况下，不可能添加“if...then...else"条件。Spring通过使用抽象工厂模式为XML配置中的条件提供了解决方案.使用工厂创建所需的Bean,并在工厂的内部逻辑中实现所需的复杂的Java代码。

Spring 容器中有两种bean：普通bean和工厂bean。Spring直接使用前者，FactoryBean跟普通Bean不同，其返回的对象不是指定类的一个实例，而是该FactoryBean的getObject方法所返回的对象。

一般情况下，Spring通过反射机制利用<bean>的class属性指定的实现类来实例化bean 。在某些情况下，实例化bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息，配置方式的灵活性是受限的，这时采用编码的方式可能会得到更好的效果。Spring为此提供了一个org.Springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化bean的逻辑。

FactoryBean接口对于Spring框架来说占有重要的地位，Spring  自身就提供了很多FactoryBean的实现。它们隐藏了实例化一些复杂bean的细节，给上层应用带来了便利。从Spring 3.0 开始，  FactoryBean开始支持泛型，即接口声明改为FactoryBean<T> 的形式。

FactoryBean 通常是用来创建比较复杂的bean，一般的bean 直接用xml配置和基于Java配置即可，但如果一个bean的创建过程中涉及到很多其他的bean 和复杂的逻辑，用xml配置比较困难，这时可以考虑用FactoryBean。



### Bean的生命周期和使用模式

#### 1.初始化阶段

1.加载Bean定义

在这个不走中，将处理所有的配置文件：

@Configuration类和XML定义文件。对于基于注解的配置，Spring将扫描所有使用了@Componnent注解的类，然后进行Bean定义的加载。而对于基于XML的配置来说，所有XML文件被解析完之后，将Bean的定义添加到BeanFactory类中。每个Bean都在其ID下被建立了索引。Spring提供了多种BeanFactoryPostProcessor,用它来解决运行时所依赖的操作，例如从外部属性文件读取值。在Spring应用中，BeanFacotryPostProcessor可以任意修改Bean的定义。

Spring先加载Bean的定义，然后再调用BeanFactoryPostProcessor修改某些Bean的定义。

例如配置文件AppConfig.java和InfraConfig.java。这些配置信息被加载到了ApplicationContext容器中，并用id作为索引。

![image-20220225165626687](D:\A_编程\spring\spring框架中的设计模式\Spring5中的设计模式.assets\image-20220225165626687.png)

SpringBean根据其属性ID被索引到了BeanFactory中，然后BeanFactory对象被当作参数传递给BeanFactoryPostProcessor的postProcess()方法中。BeanFactoryPostProcessor对象能够修改某些Bean的定义，这取决于开发人员所提供的Bean的配置。现在来看BeanFactoryPostProcessor是如何工作的：

（1）BeanFactoryPostProcessor在实际创建Bean之前，需要处理Bean的定义和Bean的元数据配置信息。

（2）Spring提供了几种有用的BeanFactoryPostProcessor实现，如读取属性文件和注册一个自定义作用域。

（3）可以基于BeanFactoryProcessor接口自己编写实现类。

（4）如果只是在某一个容器中定义BeanFactoryPostProcessor，那么它仅应用于这个容器的Bean定义。

BeanFactoryPostProcessor代码定义如下：

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory var1) throws BeansException;
}
```

BeanFactoryPostProcessor的拓展点案例：读取外部属性文件（database.properties）;

2.初始化Bean实例

在将Bean的定义加载到beanFactory之后，Spring的Ioc容器为应用程序实例画bean，这个流程如下图：

![IMG_20220225_175812](D:\A_编程\spring\spring框架中的设计模式\Spring5中的设计模式.assets\IMG_20220225_175812.jpg)



#### 2.使用阶段

#### 3.销毁阶段



