## 1.概述

Spring 容器中有两种bean：普通bean和工厂bean。Spring直接使用前者，FactoryBean跟普通Bean不同，其返回的对象不是指定类的一个实例，而是该FactoryBean的getObject方法所返回的对象。

一般情况下，Spring通过反射机制利用<bean>的class属性指定的实现类来实例化bean 。在某些情况下，实例化bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息，配置方式的灵活性是受限的，这时采用编码的方式可能会得到更好的效果。Spring为此提供了一个org.Springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化bean的逻辑。

FactoryBean接口对于Spring框架来说占有重要的地位，Spring  自身就提供了很多FactoryBean的实现。它们隐藏了实例化一些复杂bean的细节，给上层应用带来了便利。从Spring 3.0 开始，  FactoryBean开始支持泛型，即接口声明改为FactoryBean<T> 的形式。

## 2.应用场景

FactoryBean 通常是用来创建比较复杂的bean，一般的bean 直接用xml配置和基于Java配置即可，但如果一个bean的创建过程中涉及到很多其他的bean 和复杂的逻辑，用xml配置比较困难，这时可以考虑用FactoryBean。

## 3.FactoryBean接口

让我们首先看一下*FactoryBean*接口：



```java
public interface FactoryBean {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```

此接口中定义了以下3个方法,前2个方法需要我们去实现：

- *getObject（）*:返回由FactoryBean创建的bean实例，如果isSingleton()返回true，则该实例会放到Spring容器的单实例缓存池中。getObject方法内部由开发者自己去实现对象的创建，然后将创建好的对象返回给Spring容器。
- *getObjectType（）*：返回FactoryBean创建的bean类型。
- *isSingleton（）*：返回由FactoryBean创建的bean实例的作用域是singleton还是prototype，如果返回false，表示由这个FactoryBean创建的对象是多例的，那么我们每次从容器中getBean的时候都会去重新调用FactoryBean中的getObject方法获取一个新的对象。若返回true，表示创建的对象是单例的，那么我们每次从容器中获取这个对象的时候都是同一个对象。

当配置文件中配置的实现类是FactoryBean时，通过getBean()方法返回的不是FactoryBean本身，而是FactoryBean#getObject()方法所返回的对象，相当于FactoryBean#getObject()代理了getBean()方法。如果要获取FactoryBean对象，可以在id前面加一个&符号来获取。

## 4.应用案例

很多开源项目在集成Spring 时都使用到FactoryBean，比如 [MyBatis3](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fmybatis%2Fmybatis-3) 提供 mybatis-spring项目中的 `org.mybatis.spring.SqlSessionFactoryBean`：



```xml-dtd
<bean id="tradeSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="trade" />
        <property name="mapperLocations" value="classpath*:mapper/trade/*Mapper.xml" />
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <property name="typeAliasesPackage" value="com.bytebeats.mybatis3.domain.trade" />
    </bean>
```

`org.mybatis.spring.SqlSessionFactoryBean` 如下：



```java
package org.mybatis.spring;

public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
private static final Log LOGGER = LogFactory.getLog(SqlSessionFactoryBean.class);
        ......
}
```

## 5.FactoryBean实例

### 5.1 实现一个FactoryBean

现在，让我们实现一个示例*FactoryBean*。我们将实现一个*ToolFactory*，它生成*Tool*类型的对象：



```java
public class Tool {
 
    private int id;
 
    // standard constructors, getters and setters
}
```

*ToolFactory*：



```java
public class ToolFactory implements FactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    @Override
    public Tool getObject() throws Exception {
        return new Tool(toolId);
    }
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    public boolean isSingleton() {
        return false;
    }
 
    // standard setters and getters
}
```

如我们所见，*ToolFactory*是一个*FactoryBean*，可以生成*Tool*对象。

### 5.2 FactoryBean与基于XML的配置一起使用

现在让我们看看如何使用*ToolFactory*。

构建基于XML的配置– *factorybean-spring-ctx.xml*：



```xml-dtd
<beans ...>
    <bean id="tool" class="com.baeldung.factorybean.ToolFactory">
        <property name="factoryId" value="9090"/>
        <property name="toolId" value="1"/>
    </bean>
</beans>
```

接下来，我们可以测试*Tool*对象是否正确注入：



```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
    @Autowired
    private Tool tool;
 
    @Test
    public void testConstructWorkerByXml() {
        assertThat(tool.getId(), equalTo(1));
    }
}
```

尽管Spring容器使用*FactoryBean*的*getObject（）*方法的返回值作为Bean，但是您也可以使用*FactoryBean*本身。

**要访问FactoryBean，您只需要在bean名称之前添加“＆”即可。**

让我们尝试获取FactoryBean及其*factoryId*属性：



```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
 
    @Resource(name = "&tool")
    private ToolFactory toolFactory;
 
    @Test
    public void testConstructWorkerByXml() {
        assertThat(toolFactory.getFactoryId(), equalTo(9090));
    }
}
```

### 5.3 FactoryBean与基于Java的配置一起使用

在基于Java的配置中使用*FactoryBean*与基于XML的配置有所不同，您必须显式调用*FactoryBean*的*getObject（）*方法。

让我们将前面小节中的示例转换为基于Java的配置示例：



```java
@Configuration
public class FactoryBeanAppConfig {
  
    @Bean(name = "tool")
    public ToolFactory toolFactory() {
        ToolFactory factory = new ToolFactory();
        factory.setFactoryId(7070);
        factory.setToolId(2);
        return factory;
    }
 
    @Bean
    public Tool tool() throws Exception {
        return toolFactory().getObject();
    }
}
```

然后，我们测试*Tool*对象是否正确注入：



```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = FactoryBeanAppConfig.class)
public class FactoryBeanJavaConfigTest {
 
    @Autowired
    private Tool tool;
  
    @Resource(name = "&tool")
    private ToolFactory toolFactory;
 
    @Test
    public void testConstructWorkerByJava() {
        assertThat(tool.getId(), equalTo(2));
        assertThat(toolFactory.getFactoryId(), equalTo(7070));
    }
}
```

测试结果显示出与以前的基于XML的配置测试相似的效果。

## 6.初始化方式

有时，您需要在设置*FactoryBean*之后但在调用*getObject（）*方法之前执行一些操作，例如属性检查。

您可以通过实现*InitializingBean*接口或使用*@PostConstruct*注解来实现。

## 7. AbstractFactoryBean

Spring为FactoryBean实现提供了一个超类模板（AbstractFactoryBean）。有了这个模板，我们可以更方便地实现一个工厂bean，它创建一个单例或一个原型对象。

让我们实现*SingleToolFactory*和*NonSingleToolFactory*展示如何使用*AbstractFactoryBean*的singleton和prototype类型：

```java
public class SingleToolFactory extends AbstractFactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    protected Tool createInstance() throws Exception {
        return new Tool(toolId);
    }
 
    // standard setters and getters
}
```

非单例实现：

```java
public class NonSingleToolFactory extends AbstractFactoryBean<Tool> {
 
    private int factoryId;
    private int toolId;
 
    public NonSingleToolFactory() {
        setSingleton(false);
    }
 
    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }
 
    @Override
    protected Tool createInstance() throws Exception {
        return new Tool(toolId);
    }
 
    // standard setters and getters
}
```

另外，这些工厂bean的XML配置：

```xml-dtd
<beans ...>
 
    <bean id="singleTool" class="com.baeldung.factorybean.SingleToolFactory">
        <property name="factoryId" value="3001"/>
        <property name="toolId" value="1"/>
    </bean>
 
    <bean id="nonSingleTool" class="com.baeldung.factorybean.NonSingleToolFactory">
        <property name="factoryId" value="3002"/>
        <property name="toolId" value="2"/>
    </bean>
</beans>
```

测试是否按预期注入了*Worker*对象的属性：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-abstract-spring-ctx.xml" })
public class AbstractFactoryBeanTest {
 
    @Resource(name = "singleTool")
    private Tool tool1;
  
    @Resource(name = "singleTool")
    private Tool tool2;
  
    @Resource(name = "nonSingleTool")
    private Tool tool3;
  
    @Resource(name = "nonSingleTool")
    private Tool tool4;
 
    @Test
    public void testSingleToolFactory() {
        assertThat(tool1.getId(), equalTo(1));
        assertTrue(tool1 == tool2);
    }
 
    @Test
    public void testNonSingleToolFactory() {
        assertThat(tool3.getId(), equalTo(2));
        assertThat(tool4.getId(), equalTo(2));
        assertTrue(tool3 != tool4);
    }
}
```

正如我们从测试看出，*SingleToolFactory*生成单实例，而*NonSingleToolFactory*生成原型对象。

请注意，无需在*SingleToolFactory中*设置singleton属性，因为在*AbstractFactory中*，singleton属性的默认值为*true*。

## 8.结论

使用FactoryBean可以封装复杂的构造逻辑，或者使Spring中配置高度可配置的对象更容易，这是一个很好的实践。

因此，在本文中，我们介绍了如何实现FactoryBean的基础知识，如何在基于xml的配置和基于java的配置中使用它，以及FactoryBean的其他一些用法，比如FactoryBean和AbstractFactoryBean的初始化。

