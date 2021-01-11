# 核心技术

> 5.3.2

# 1.Ioc容器

##  1.1. Spring Ioc容器和Beans介绍

本章介绍了控制反转（IoC）原理的Spring框架实现。 IoC也称为依赖注入（DI）。 在此过程中，对象仅通过构造函数参数，工厂方法的参数或在构造后或从工厂方法返回后在对象实例上设置的属性来定义其依赖项（即与它们一起使用的其他对象） 。 然后，容器在创建bean时注入那些依赖项。 此过程从根本上讲是bean本身通过使用类的直接构造或诸如[服务定位器模式](#服务定位器模式)之类的机制来控制其依赖项的实例或位置的逆过程（因此称为控制的反转）。

`org.springframework.beans`和`org.springframework.context`包是Spring Framework的IoC容器的基础。`BeanFactory`接口提供了一种能够管理任何类型对象的高级配置机制。`ApplicationContext`是`BeanFactory`的子接口。 它增加了：

- 与Spring AOP功能轻松集成
- 消息资源处理（用于国际化）
- 事件发布
- 应用层特定的上下文，例如Web应用程序中使用的WebApplicationContext

简而言之，BeanFactory提供了配置框架和基本的功能，而ApplicationContext添加了更多企业特定的功能。 ApplicationContext是BeanFactory的完整超集，在本章中仅在Spring的IoC容器描述中使用。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中众多对象中的一个。bean及其之间的依赖关系反映在容器使用的配置元数据中。

## 1.2. Container介绍

org.springframework.context.ApplicationContext接口代表Spring IoC容器，并负责实例化，配置和组装Bean。 容器通过读取配置元数据获取有关要实例化，配置和组装哪些对象的说明。 配置元数据以XML，Java注解或Java代码表示。 它使您能够表达组成应用程序的对象以及这些对象之间的丰富相互依赖关系。

Spring提供了ApplicationContext接口的几种实现。 在独立应用程序中，通常创建`ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`的实例。 尽管XML是定义配置元数据的传统格式，但是可以通过提供少量XML配置来声明性地启用对其他元数据格式的支持，从而指示容器将Java注解或代码用作元数据格式。

在大多数应用场景中，不需要显式用户代码即可实例化一个Spring IoC容器的一个或多个实例。 例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（约）样板Web描述符XML通常就足够了。 如果您使用Spring Tools for Eclipse（由Eclipse驱动的开发环境），则只需单击几下鼠标或击键即可轻松创建此样板配置。

下图显示了Spring的工作原理的高级视图。 您的应用程序类与配置元数据结合在一起，以便在创建和初始化ApplicationContext之后，您将拥有一个完全配置且可执行的系统或应用程序。

![container magic](https://docs.spring.io/spring-framework/docs/current/reference/html/images/container-magic.png)

### 1.2.1. 元数据配置

如上图所示，Spring IoC容器使用一种形式的配置元数据。 此配置元数据表示您作为应用程序开发人员如何告诉Spring容器实例化，配置和组装应用程序中的对象。传统上，配置元数据以简单直观的XML格式提供，这是本章大部分用来传达Spring IoC容器的关键概念和功能的格式。

> 基于XML的元数据不是配置元数据的唯一允许形式。 Spring IoC容器本身与实际写入此配置元数据的格式完全脱钩。 如今，许多开发人员为他们的Spring应用程序选择基于Java代码的配置。

- 基于注解的配置：Spring 2.5引入了对基于注解的配置元数据的支持。
- 基于Java的配置：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。 因此，您可以使用Java而不是XML文件来定义应用程序类外部的bean。 要使用这些新功能，请参见`@ Configuration`，`@Bean`，`@Import`和`@DependsOn`注解。

Spring配置由容器必须管理的至少一个（通常是一个以上）bean定义组成。 基于XML的配置元数据将这些bean配置为顶级`<beans />`元素内的`<bean />`元素。 Java配置通常在`@Configuration`类中使用`@Bean`注释的方法。

这些bean定义对应于组成应用程序的实际对象。 通常，您定义服务层对象，数据访问对象（DAO），表示对象（例如Struts Action实例），基础结构对象（例如Hibernate SessionFactories，JMS队列）等等。 通常，不会在容器中配置细粒度的域对象，因为DAO和业务逻辑通常负责创建和加载域对象。 但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。 请参阅使用AspectJ通过Spring依赖注入域对象。

以下示例显示了基于XML的配置元数据的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

- `id`属性是一个标识单个bean定义的字符串。它的值指代该对象的引用。

- `class`属性定义bean的类型并使用完全限定名。

### 1.2.2. 容器初始化

提供给ApplicationContext构造函数的一个或多个位置路径是资源字符串，这些资源字符串使容器可以从各种外部资源（例如本地文件系统，Java CLASSPATH等）加载配置元数据。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由PetStoreServiceImpl类和两个JpaAccountDao和JpaItemDao类型的数据访问对象组成（基于JPA对象关系映射标准）。 属性名称元素引用JavaBean属性的名称，而ref元素引用另一个bean定义的名称。 id和ref元素之间的这种联系表达了协作对象之间的依赖性。

#### 构造基于XML的元数据

使bean定义跨越多个XML文件可能很有用。 通常，每个单独的XML配置文件都代表体系结构中的逻辑层或模块。您可以使用应用程序上下文构造函数从所有这些XML片段中加载bean定义。 如上一节中所示，此构造函数具有多个Resource位置。 或者，使用`<import />`元素的一个或多个实例从另一个文件加载bean定义。 以下示例显示了如何执行此操作：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义是从三个文件加载的：services.xml，messageSource.xml和themeSource.xml。 所有位置路径都相对于执行导入的定义文件，因此，services.xml必须与执行导入的文件位于同一目录或类路径位置，而messageSource.xml和themeSource.xml必须位于该位置下方的资源位置 。 如您所见，斜杠被忽略。 但是，*鉴于这些路径是相对的，最好不要使用任何斜线*。 根据Spring Schema，导入的文件的内容（包括顶级`<beans />`元素）必须是有效的XML bean定义。

> 可以但不建议使用相对路径“ ../”引用父目录中的文件。 这样做会创建对当前应用程序外部文件的依赖。 特别是，不建议对classpath：URL（例如，classpath：../ services.xml）使用此引用，在URL中，运行时解析过程会选择“最近”的classpath根目录，然后查看其父目录。 类路径配置的更改可能导致选择其他错误的目录。您始终可以使用绝对资源路径而不是相对路径：例如，file：C：/config/services.xml或classpath：/config/services.xml。 但是请注意，您正在将应用程序的配置耦合到特定的绝对路径。 通常，最好为这样的绝对路径保留一个间接寻址，例如，通过在运行时针对JVM系统属性解析的“ $ {…}”占位符。

命名空间本身提供了import指令特性。除了普通bean定义之外，Spring提供的XML名称空间的选择中还有其他配置特性，例如`context`和`util`命名空间。

#### The Groovy Bean Definition DSL(领域特定语言)

作为外部化配置元数据的另一个示例，Bean定义也可以在Spring的Groovy Bean定义DSL中表达，如Grails框架所示。 通常，这种配置位于“ .groovy”文件中，其结构如以下示例所示：

```groovy
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这种配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。 它还允许通过`importBeans`指令导入XML bean定义文件。

### 1.2.3. 使用容器

`ApplicationContext`是高级的工厂接口，该工厂能够维护不同bean及其依赖关系的注册表。 通过使用方法`T getBean(String name，Class <T> requiredType)`，可以检索bean的实例。

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

在Groovy配置中，使用看起来非常相似。它有一个不同的上下文实现类，该类支持groovy(但也理解XML bean定义)。下面的示例显示了Groovy配置

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是结合reader委托使用的GenericApplicationContext，例如，使用XML文件的XmlBeanDefinitionReader，如下面的示例所示

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

还可以为Groovy文件使用GroovyBeanDefinitionReader，如下面的示例所示

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以在相同的ApplicationContext上混合和匹配这样的读取器委派，从不同的配置源读取bean定义.

然后可以使用getBean检索bean的实例。 ApplicationContext接口还有其他几种检索bean的方法，但是理想情况下，您的应用程序代码永远不要使用它们。 实际上，您的应用程序代码应该根本不调用getBean（）方法，因此完全不依赖于Spring API。 例如，Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖项注入，使您可以通过元数据（例如自动装配注释）声明对特定Bean的依赖项。

## 1.3. Bean介绍

在容器本身内，bean定义表示为BeanDefinition对象，其中包含（除其他信息外）以下元数据：

- 包限定的类名：通常，定义了Bean的实际实现类。

- Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。

- 引用该bean完成其工作所需的其他bean。 这些引用也称为协作者或依赖项。

- 要在新创建的对象中设置的其他配置设置，例如，池的大小限制或在管理连接池的bean中要使用的连接数。

该元数据转换为一组组成每个bean定义的属性。下表描述了这些属性

| Property                 | Explained in…                                                |
| :----------------------- | :----------------------------------------------------------- |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

除了包含有关如何创建特定bean的信息的bean定义之外，*ApplicationContext实现还允许注册在容器外部（由用户）创建的现有对象*。 这是通过通过getBeanFactory（）方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory DefaultListableBeanFactory实现。 DefaultListableBeanFactory通过registerSingleton（..）和registerBeanDefinition（..）方法支持此注册。 但是，典型的应用程序只能与通过常规bean定义元数据定义的bean一起使用。

> Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自省步骤期间正确地判断它们。虽然在某种程度上支持覆盖现有元数据和现有的单例实例，但在运行时注册新bean(与对工厂的实时访问并发)不受官方支持，可能会导致并发访问异常、bean容器中的不一致状态，或者两者都有。

### 1.3.1. Beans命名

每个bean具有一个或多个标识符。 这些标识符在承载Bean的容器内必须唯一。 一个bean通常只有一个标识符。 但是，如果需要多个，则可以将多余的别名视为别名。

在基于XML的配置元数据中，可以使用id属性和/或name属性来指定Bean标识符。 id属性可让您精确指定一个id。 按照惯例，这些名称是字母数字（“ myBean”，“ someService”等），但它们也可以包含特殊字符。 如果要为bean引入其他别名，还可以在name属性中指定它们，并用逗号（`，`），分号（`;`）或空格分隔。 作为历史记录，在Spring 3.1之前的版本中，id属性定义为`xsd：ID`类型，该类型限制了可能的字符。 从3.1开始，它被定义为`xsd：string`类型。 请注意，bean ID唯一性仍由容器强制执行，尽管不再由XML解析器执行。

您*不需要提供bean的名称或ID。 如果未明确提供名称或ID，则容器将为该bean生成一个唯一的名称。 但是，如果要按名称引用该bean，则通过使用ref元素或服务定位器样式查找，必须提供一个名称。 不提供名称的动机与使用内部bean和自动装配协作类有关*。

> 约定是在命名bean时使用标准Java约定来命名实例字段名。也就是说，bean名称以小写字母开头，并从那里采用驼峰格式。这些名称的例子包括accountManager、accountService、userDao、loginController等等。一致地命名bean可以使配置更易于阅读和理解。另外，如果您使用Spring AOP，在将通知(`advice`)应用到一组与名称相关的bean时，它会很有用。
>
> 通过在类路径中进行组件扫描，Spring会按照前面描述的规则为未命名的组件生成Bean名称：本质上，采用简单的类名称并将其初始字符转换为小写。 但是，在特殊情况下，如果有多个字符并且第一个和第二个字符均为大写字母，则会保留原始大小写。 这些规则与`java.beans.Introspector.decapitalize`（Spring使用的）定义的规则相同。

#### Bean别名

在bean定义本身中，可以使用由`id`属性指定的最多一个名称和`name`属性中任意数量的其他名称的组合，为bean提供多个名称。 这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如，通过使用特定于该组件本身的bean名称，让应用程序中的每个组件都引用一个公共依赖项。

但是，在实际定义bean的地方指定所有别名并不够。 有时需要为在别处定义的bean引入别名。 这在大型系统中通常是这种情况，在大型系统中，配置在每个子系统之间分配，每个子系统都有自己的对象定义集。 在基于XML的配置元数据中，可以使用`<alias />`元素来完成此操作。 以下示例显示了如何执行此操作：

```xml
<alias name="fromName" alias="toName"/>
```

例如，子系统A的配置元数据可以通过子系统A-dataSource的名称引用数据源。 子系统B的配置元数据可以通过子系统B-dataSource的名称引用数据源。 组成使用这两个子系统的主应用程序时，主应用程序通过myApp-dataSource的名称引用数据源。 要使所有三个名称都引用相同的对象，可以将以下别名定义添加到配置元数据中：

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

现在，每个组件和主应用程序都可以通过唯一的名称引用数据源，并且保证不会与任何其他定义(有效地创建一个名称空间)冲突，但它们引用的是同一个bean。

> 如果使用Javaconfiguration，则@Bean注解可用于提供别名。

### 1.3.2. 实例化Beans

bean定义本质上是创建一个或多个对象的方法。当被请求时，容器查看指定bean的方法，并使用该bean定义封装的配置元数据创建(或获取)实际对象。

如果使用基于XML的配置元数据，则在`<bean />`元素的class属性中指定要实例化的对象的类型（或类）。 这个类属性（在内部是BeanDefinition实例的Class属性）通常是必需的。 （可以通过以下两种方式之一使用Class属性：

- 通常，在容器本身通过反射性地调用其构造函数直接创建Bean的情况下，指定要构造的Bean类，这在某种程度上等同于使用new运算符的Java代码。

- 要指定包含用于创建对象的静态工厂方法的实际类，在不太常见的情况下，容器将在类上调用静态工厂方法以创建Bean。 从静态工厂方法的调用返回的对象类型可以是同一类，也可以是完全不同的另一类。

> **如果要为静态内部类配置Bean定义，则必须使用内部类的二进制名称**。例如，如果您在com.example包中有一个名为SomeThing的类，并且此SomeThing类具有一个名为OtherThing的静态内部类，则bean定义上的class属性的值为com.example.SomeThing $ OtherThing。请注意，名称中使用$字符将内部类的类名与外部类名分开。

#### 构造器实例化

当通过构造方法创建一个bean时，所有普通类都可以被Spring使用并兼容。 也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码。 只需指定bean类就足够了。 但是，根据您用于该特定bean的IoC的类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您要管理的任何类。 它不仅限于管理真正的JavaBean。 大多数Spring用户更喜欢实际的JavaBean，它仅具有默认（无参数）构造函数，并具有根据容器中的属性构造的适当setter和getter。 您还可以在容器中具有更多非bean规范的类。 例如，如果您需要使用绝对不符合JavaBean规范的旧式连接池，则Spring也可以对其进行管理。

#### 静态工厂实例化

定义使用静态工厂方法创建的bean时，请使用class属性指定包含静态工厂方法的类，并使用名为factory-method的属性指定工厂方法本身的名称。 您应该能够调用此方法（使用可选参数，如稍后所述）并返回一个活动对象，该对象随后将被视为已通过构造函数创建。 这种bean定义的一种用法是在旧版代码中调用静态工厂。

以下bean定义指定通过调用工厂方法来创建bean。 该定义不指定返回对象的类型（类），而仅指定包含工厂方法的类。 在此示例中，createInstance（）方法必须是静态方法。 以下示例显示如何指定工厂方法：

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了可与前面的bean定义一起使用的类：

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

#### 实例工厂实例化

类似于通过静态工厂方法进行实例化，使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean。 要使用此机制，请将class属性保留为空，并在factory-bean属性中，在当前（或祖先）容器中指定包含要创建该对象的实例方法的bean的名称。 使用factory-method属性设置工厂方法本身的名称。 以下示例显示了如何配置此类Bean：

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含一个以上的工厂方法，如以下示例所示：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。 

> 在Spring文档中，“ factory bean”是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。 相比之下，`FactoryBean`（注意大小写）是指特定于Spring的FactoryBean实现类。

#### 确定Bean的运行时类型

确定特定bean的运行时类型并非易事。 Bean元数据定义中的指定类只是初始类引用，可能与声明的工厂方法结合使用，或者是FactoryBean类，这可能导致Bean的运行时类型不同，或者在实例级工厂方法（通过指定的factory-bean名称解析）的情况下完全不进行设置 。 此外，AOP代理可以使用基于接口的代理包装bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。

*找出特定bean的实际运行时类型的推荐方法是对指定bean名称的BeanFactory.getType调用*。 这考虑了上述所有情况，并返回了针对相同bean名称的BeanFactory.getBean调用将返回的对象的类型。

## 1.4. 依赖Dependencies

典型的企业应用程序不包含单个对象(或Spring术语中的bean)。即使是最简单的应用程序也有几个对象一起工作，以表示最终用户认为一致的应用程序。下一节将解释如何从定义大量独立的bean定义过渡到一个完全实现的应用程序，在该应用程序中，对象相互协作以实现目标。

### 1.4.1. 依赖注入

依赖注入（DI）是一个过程，在这个过程中，对象仅通过构造函数参数、工厂方法的参数，或者在对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖项(即与它们一起工作的其他对象)。 然后，容器在创建bean时注入那些依赖项。 从根本上讲，此过程是通过使用类的直接构造或服务定位器模式来控制bean自身依赖的实例或位置的bean创建的逆过程（因此称为Control Inversion-控制反转）

使用DI原则后，代码更加清晰，当提供对象的依赖关系时，解耦会更加有效。对象不查找它的依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这些依赖关系允许在单元测试中使用stub或mock 实现。

*依赖注入有两种主要的变体:基于构造函数的依赖注入和基于setter的依赖注入。*

#### 构造器注入

基于构造函数的DI是通过容器调用具有多个参数的构造函数来完成的，每个参数表示一个依赖项。 调用带有特定参数的静态工厂方法来构造Bean几乎是等效的，并且本次讨论将构造函数和静态工厂方法的参数视为类似。 以下示例显示了只能通过构造函数注入进行依赖项注入的类：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
	/*这里的MovieFinder需由Ioc容器管理；若有多个构造器的话，需要注意是否正确使用，可参考
      https://blog.csdn.net/qq_41737716/article/details/85596817*/
    // business logic that actually uses the injected MovieFinder is omitted...
}
```

注意，该类没有什么特别的。 它是一个POJO，不依赖于特定于容器的接口，基类或注解。

**构造器参数解析**

构造函数参数解析匹配通过使用参数的类型进行。 如果Bean定义的构造函数参数中没有潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。 考虑以下类：

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设ThingTwo和ThingThree类没有通过继承关联，则不存在潜在的歧义。 因此，以下配置可以正常运行，并且您无需在`<constructor-arg />`元素中显式指定构造函数参数索引或类型。

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以发生匹配（与前面的示例一样）。 当使用简单类型（例如<value> true </ value>）时，Spring无法确定值的类型，因此在没有指示的情况下无法按类型进行匹配。 考虑以下类别：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

**构造器参数类型匹配**

在上述情况下，如果通过使用`type`属性显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。 如下例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**构造器参数索引**

您可以使用index属性来显式指定构造函数参数的索引，如以下示例所示：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义之外，指定索引还可以解决构造函数有两个相同类型的参数的歧义。

> 索引从0开始。

**构造器参数名**

还可以使用构造函数参数名来消除值的歧义，如下面的示例所示

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，要使其能够开箱即用，您的代码必须在编译时启用debug标志，以便Spring可以从构造函数查找参数名。如果您不能或不希望用debug标志编译您的代码，您可以使用@ConstructorProperties JDK注解来显式地命名您的构造函数参数。示例类如下所示

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#### setter注入

基于setter的DI是由容器在调用无参数构造函数或无参数静态工厂方法来实例化bean之后再调用bean上的setter方法来实现的。

下面的示例显示了一个只能通过使用纯setter注入来注入依赖项的类。这个类是传统的Java。它是一个不依赖于容器特定接口、基类或注解的POJO。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext`支持它管理的bean的基于构造函数和基于setter的DI。 在已经通过构造函数方法注入了某些依赖项之后，它还支持基于setter的DI。 您可以以`BeanDefinition`的形式配置依赖项，并与`PropertyEditor`实例结合使用，以将属性从一种格式转换为另一种格式。 但是，大多数Spring用户并不直接（即以编程方式）使用这些类，而是使用XML bean定义，带注解的组件（即以@ Component，@ Controller等进行注解的类）或@Bean方法来处理这些类。 基于Java的@Configuration类。 然后将它们在内部转换为BeanDefinition实例，并用于加载整个Spring IoC容器实例。

> 由于可以混合使用基于构造函数的DI和基于setter的DI，因此将构造函数用于强制性依赖项并将setter方法或配置方法用于可选的依赖项是一个很好的经验。 注意，在setter方法上使用@Required批注可以使该属性成为必需的依赖项。 但是，最好使用带有参数的程序验证的构造函数注入。
>
> Spring团队通常提倡构造函数注入，因为它使您可以将应用程序组件实现为不可变对象，并确保所需的依赖项不为null。 此外，注入构造函数的组件始终以完全初始化的状态返回到客户端（调用）代码。 附带说明一下，构造器包含大量的参数是不好的代码，这表明该类可能承担了太多的职责，应进行重构以更好地解决关注点分离问题。
>
> Setter注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。 否则，必须在代码使用依赖项的任何地方执行非空检查。 setter注入的一个好处是，setter方法使该类的对象在以后可以重新配置或重新注入。 因此，通过JMX MBean进行管理是用于setter注入的典型用例。
>
> 使用对特定类最有意义的DI。 有时，在处理您没有源代码的第三方类时，将为您做出选择。 例如，如果第三方类未公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。

#### 依赖解析过程

容器按如下方式执行bean依赖项解析：

- 使用描述所有bean的配置元数据创建和初始化ApplicationContext。 可以通过XML，Java代码或注解指定配置元数据。

- 对于每个bean，它的依赖关系都以属性、构造函数参数或静态工厂方法的参数的形式表示(如果您使用该方法而不是普通的构造函数)。当实际创建bean时，将向bean提供这些依赖项。

- 每个属性或构造函数参数都是要设置的实参，或者是对容器中另一个bean的引用。

- 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，比如int、long、string、boolean等等。

在创建容器时，Spring容器会验证每个bean的配置。 但是，在实际创建Bean之前，不会设置Bean属性本身。 创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean。 作用域在Bean作用域中定义。 否则，仅在请求时才创建Bean。 创建和分配bean的依赖关系及其依赖关系（依此类推）时，创建bean可能会导致创建一个bean图。 请注意，这些依赖项之间的解析不匹配可能会在后期出现，即在第一次创建受影响的bean时。

> 如果主要使用构造函数注入，则可能会创建无法解析的循环依赖方案。
>
> 例如：A类通过构造函数注入需要B类的实例，而B类通过构造函数注入需要A类的实例。 如果您为将类A和B相互注入而配置了bean，则Spring IoC容器会在运行时检测到此循环引用，并抛出`BeanCurrentlyInCreationException`。
>
> 一种可能的解决方案是编辑某些类的源代码，这些类的源代码由setter而不是构造函数来配置。 或者，避免构造函数注入，而仅使用setter注入。 换句话说，尽管不建议这样做，但是您可以使用setter注入配置循环依赖项。
>
> 与典型情况（没有循环依赖关系）不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。

通常，您可以信任Spring做正确的事。 它在容器加载时检测配置问题，例如对不存在的Bean的引用和循环依赖项。 在实际创建bean时，Spring设置属性并尽可能晚地解决依赖关系。 这意味着，如果创建对象或其依赖项之一有问题，则正确加载了Spring的容器以后可以在您请求对象时生成异常-例如，bean会因丢失或无效属性而抛出异常。这可能会延迟某些配置问题的可见性，这就是为什么默认情况下ApplicationContext实现会预先实例化单例bean的原因。 在实际需要这些bean之前需要花一些前期时间和内存来创建它们，您会在创建ApplicationContext时发现配置问题，而不是稍后。 您仍然可以覆盖此默认行为，以便单例bean延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，则在将一个或多个协作Bean注入到从属Bean中时，每个协作Bean在注入到从属Bean中之前都已完全配置。 这意味着，如果bean A依赖于bean B，则Spring IoC容器会在对bean A调用setter方法之前完全配置beanB。换句话说，被实例化的bean（如果它不是预先实例化的单例） ），它的依赖项被设置，并调用了相关的生命周期方法（例如已配置的init方法或InitializingBean回调方法）。

#### 依赖注入实例

下面的示例为基于setter的DI使用基于xml的配置元数据。Spring XML配置文件的一小部分指定了一些bean定义，如下所示

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，将setter声明为与XML文件中指定的属性匹配。下面的示例使用基于构造函数的DI

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

bean定义中指定的构造函数参数用作ExampleBean构造函数的参数。

现在考虑该示例的一个变体，在该变体中，不是使用构造函数，而是告诉Spring调用静态工厂方法以返回对象的实例：

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

静态工厂方法的参数由`<constructor-arg />`元素提供，与实际使用构造函数的情况完全相同。 factory方法返回的类的类型不必与包含静态工厂方法的类的类型相同（尽管在此示例中是）。 实例（非静态）工厂方法可以以基本上相同的方式使用（除了使用factory-bean属性代替class属性之外），因此在此不讨论这些细节。



### 1.4.2.  依赖配置详解





































# 附录

## 服务定位器模式

> 服务定位器模式 服务定位器模式（Service Locator Pattern）用在我们想使用 JNDI 查询定位各种服务的时候。考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能

服务定位器模式（Service Locator Pattern）用在我们想使用 JNDI 查询定位各种服务的时候。考虑到为某个服务查找 JNDI 的代价很高，服务定位器模式充分利用了缓存技术。在首次请求某个服务时，服务定位器在 JNDI 中查找服务，并缓存该服务对象。当再次请求相同的服务时，服务定位器会在它的缓存中查找，这样可以在很大程度上提高应用程序的性能。以下是这种设计模式的实体。

- **服务（Service）** - 实际处理请求的服务。对这种服务的引用可以在 JNDI 服务器中查找到。

- **Context / 初始的 Context** - JNDI Context 带有对要查找的服务的引用。
- **服务定位器（Service Locator）** - 服务定位器是通过 JNDI 查找和缓存服务来获取服务的单点接触。
- **缓存（Cache）** - 缓存存储服务的引用，以便复用它们。
- **客户端（Client）** - Client 是通过 ServiceLocator 调用服务的对象。

![servicelocator_pattern_uml_diagram.jpg](http://img.php.cn/upload/image/614/338/876/1486194513420223.jpg)