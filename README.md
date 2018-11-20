1. Spring整体架构

Spring框架是一种分层架构，它包含了一系列的功能，大概由20种模块组成。 这些模块分为核心容器(Core Container), 数据访问/集成(Data Access/Integration), Web, AOP, 工具(Instrumentation), 消息(Messaging), 测试用例(Test).

这里写图片描述

1.1 核心容器(Core Container)

包含模块spring-core, spring-beans, spring-context, spring-context-support,spring-expression.

    spring-core 主要包含Spring框架基本的核心工具类
    spring-beans 包含访问配置文件、创建和管理bean以及进行IoC/DI操作的相关类. BeanFactory
    spring-context 构建与Core和Beans之上，继承了Beans的特性，扩展添加了国际化、时间传播、资源加载和对Context的创建和支持。ApplicationContext
    spring-expression提供 一个强大的表达式语言用于在运行时查询和操作对象，该语言支持设置/获取属性值，属性的分配，方法的调用，访问数组上下文、容器和索引器、逻辑和算是运算符、命名变量以及从Spring的容器中根据名称检索对象

1.2 AOP和Instrumentation

包含模块spring-aop, spring-aspects, spring-instrument, spring-instrument-tomcat

    spring-aop 提供了一个AOP联盟标准的面向方面编程的实现，它允许你定义方法拦截器与切入点，从而将逻辑代码与实现函数进行分离。
    spring-aspects 提供了与AspectJ的集成
    spring-instrument 提供了类工具的支持与classloader的实现，以便在特定的应用服务上使用。
    spring-instrument-tomcat 包含了spring对于Tomcat的代理

1.3 消息(Messaging)

spring framework 4 包含了spring-messaging模块，其中使用了来自于spring integration项目的关键抽象，如Message, MessageChannel, MessageHandler等，他们可以作为基于消息的应用服务的基础。该模块还包含了一组可将消息映射到方法的注解，类似于spring-mvc的编程模型.

1.4 数据访问/集成(Data Access/ Integration)

包含spring-jdbc, spring-tx, spring-orm, spring-oxm, spring-jms.

    spring-jdbc 提供了JDBC抽象层，消除了冗长的JDBC编码和解析数据库厂商特有的错误代码.
    spring-tx 为实现了特定接口的类提供了可编程的声明式事务管理支持，对所有的POJOs都适用
    spring-orm 提供了对象相关映射(ORM)集成，包含JPA, JDO, Hibernate，使用spring-orm模块可以将这些框架与spring提供的特性结合在一起使用，比如事务管理.
    spring-oxm 提供了对Object/Xml Mapping实现的抽象，包括JAXB,Castor, XMLBeans, JiBX以及XStream.
    spring-jms 包含了一些生产和消费消息的特性，从spring Framework 4.1开始，提供了与spring-messaging集成.

1.5 Web

包含spring-web, spring-webmvc, spring-websocket, spring-webmvc-portlet

    spring-web提供了基于面向web集成的特性，如多文件上传功能、通过servlet listener初始化IoC容器与面向web的ApplicationContext，它还包含了HTTP客户端与Spring远程支持的web相关的部分.
    spring-webmvc（又名web-servlet）包含了Spring对于Web应用的MVC与REST实现，Spring MVC框架提供了领域模型代码和Web表单之间的分离，并集成了Spring框架的所有其他特性.
    spring-webmvc-portlet（又名web-portlet）提供了基于Portlet环境使用MVC的实现.

1.6 Test

spring-test模块通过Junit或TestNG对spring的组件提供了单元测试和集成测试
2. 核心技术
2.1 IoC容器
2.1.1 IoC介绍

IoC也成为DI(dependency injection), 它是对象定义其依赖关系的过程. 一般对象通过构造器参数、工厂方法参数或者构造(工厂方法返回实例)之后set相应的属性完成依赖配置.
容器在创建Bean的时候注入这些依赖，这个过程从根本上来说是反转，因此叫做IoC, 它通过直接构建类来控制其初始化或依赖类的位置。这种机制类似服务定位器设计模式。

Spring Framework的IoC容器的基础包是org.springframework.beans与org.springframe.context. 其中接口BeanFactory提供框架配置和基本功能，
其子接口ApplicationContext增加了更多的企业级功能，如国际化资源处理、事件发布以及应用层特定的上下文(如用于web应用的WebApplicationContext).ApplicationContext是BeanFactory完整的超集.
2.1.2 容器概述

ApplicationContext接口相当于负责bean的初始化、配置和组装的IoC容器.
Spring为ApplicationContext提供了一些开箱即用的实现, 独立的应用可以使用ClassPathXmlApplicationContext或者FileSystemXmlApplicationContext，web应用在web.xml配置监听，提供xml位置和org.springframework.web.context.ContextLoaderListener即可初始化WebApplicationContextIoC容器.
2.1.2.1 配置元数据

配置元数据配置了应用中实例的实例化、配置以及组装的规则，SpringIoC容器通过此配置进行管理Bean. 配置元数据有以下几种方式：

    基于XML配置： 清晰明了，简单易用
    基于Java代码配置：无xml,通过@Configuration来声明配置、对象实例化与依赖关系
    基于Java注解配置：少量的XML(<context:annotation-config/>)，通过注解声明实例化类与依赖关系

后续的分析基于XML配置, 与Java代码和注解大体上的机制是一样
2.1.2.2 实例化容器

实例化容器非常简单，只需要提供本地配置路径或者根据ApplicationContext的构造器提供相应的资源(Spring的另一个重要抽象)即可.

ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

    1

此处先给出类图， 稍后再做具体分析.

这里写图片描述
2.1.3 Bean概述

spring IoC容器管理了多个根据配置元数据创建的bean. 在容器内部，这些Bean可以描述为BeanDefinition对象，它主要包含这些元数据:

    全类名：通常为Bean的实际实现类
    Bean的行为配置元素，这种状态描述了bean在容器中的行为，如作用域、生命周期回调等等.
    当前bean依赖bean的引用: 依赖项
    新创建对象中的其他配置设置，如池对象bean的池大小限制等.

容器除了通过配置元数据管理bean之外， 还可以通过ApplicationContext的方法getBeanFactory()获取BeanFactory的实现DefaultListableBeanFactory来注册自己的bean, 但通过不建议.

Bean的实例化

可以将bean的定义想象为创建对象的菜谱，容器在请求时查看对应bean的配方(配置元数据)，并使用该配方创建实际对象.

使用XML配置元数据的时候需要指定bean的class属性，class属性强制作为BeanDefinition的getBeanClassName()值，但这个属性不一定是运行时的最终类名(静态工厂方法或者类继承).

使用Class属性有种方式：

    通常，当容器本身通过反射调用其构造函数直接创建bean的时候需要制定Class，这与new ...()操作类似
    指定包含静态工厂方法的实际类，该方法用于创建对象.

<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
<bean id="clientService" factory-bean="serviceLocator" factory-method="createClientServiceInstance"/>

    1
    2

2.1.4 依赖
2.1.4.1 Dependency Injection

使用DI使得代码更加简洁，依赖解耦. 实际对象不需要知道依赖的类的具体实现或依赖项的位置。这更易于测试，尤其是基于接口进行依赖时.

依赖注入方式如下：

    基于构造器的依赖注入

    <beans>
      <bean id="foo" class="x.y.Foo">
          <constructor-arg ref="bar"/>
          <constructor-arg ref="baz"/>
      </bean>

      <bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg type="int" value="7500000"/>
        <constructor-arg type="java.lang.String" value="42"/>
      </bean>

      <bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg index="0" value="7500000"/>
        <constructor-arg index="1" value="42"/>
      </bean>

      <bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg name="years" value="7500000"/>
        <constructor-arg name="ultimateAnswer" value="42"/>
      </bean>
    </beans>
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13
        14
        15
        16
        17
        18
        19
        20
        21
        22

    基于Setter的依赖注入
    ApplicationContext支持基于构造器注入和基于set注入，也支持通过构造器注入后再通过set方法注入.

        使用构造器注入还是使用setter注入？
        你可以混合使用两者，构造函数用于强制依赖项，setter方法用于可选依赖项(set方法上加@Required为强制依赖项)
        Spring团队提倡构造函数注入.
        set注入的一个好处是对象可以重新配置注入.

    依赖解析过程

容器执行的解析如下：

    使用配置元数据(xml、java、annotation)来创建和初始化ApplicationContext.
    对于每个bean, 它的依赖关系可能是属性、构造参数、静态工厂方法。 这些依赖将会在bean实际创建时提供.
    每个属性或者构造器参数都被设置一个实际定义的值或者引用另一个bean
    每个属性或者构造器参数的值都会被转换成该属性或构造器的实际类型.

spring在创建容器的时候会验证每个bean的配置，但是在实际创建bean之前不会设置bean本身的属性. 单例作用域的bean在容器创建的时候会被预初始化. 其他的时候bean是在请求的时候才被创建。
创建一个Bean的时候回创建这个bean的依赖以及依赖的依赖.所以解析一些依赖项的时候出现不匹配问题可能出现的比较晚，比如第一次创建受影响的bean的时候.

注意 通过构造器注入可能会导致循环引用，如ClassA构造器需要ClassB, 而ClassB的构造器需要ClassA。此时使用set注入可避免此种情况,但尽量避免出现循环引用.
2.1.4.2 依赖和配置细节

    直接值（基本类型，字符串等）：可通过<property>元素的value属性指定，也可以使用p-namesoace(xmlns:p="http://www.springframework.org/schema/p")指定.
    idref元素：<idref bean="" />元素是一种简单的误差检验方式，通过id引用其他的bean.使用idref标签允许容器在部署的时候检测引用的bean是否存在而不是容器部署完成才发现异常.
    引用其他的bean:<ref>引用其他的bean, 在引用之前将强制设置所需属性.
    内部bean: <property> <bean ...> <property/>
    集合： <property>内可以配置<props> <list> <map> <set>

2.1.4.3 使用depends-on

如果一个bean依赖另一个bean，通常作为会作为一个属性. 然而，有时候依赖之间没有明显的关系,如一个类初始化之前需要另一个初始化。
depends-on属性就明确的声明了初始化当前bean之前应该有那个bean被初始化.<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
2.1.4.4 延迟加载bean

默认ApplicationContext在初始化流程部分是立即创建配置的所有单例bean.通常这种预先初始化的方式是比较指的肯定的，因为这样在配置或建立环境的时候能立即发现错误。
但有些时候你可能不想让单例的bean预先初始化，这个时候就可以使用lazy-init属性来告诉IOC容器当第一次请求这个bean时再创建.
当一个不是lazy init的单例bean依赖一个lazy init的bean时， ApplicationContext在启动的时候会创建lazy init bean.
2.1.5 自动装配

Spring容器可以自动装配bean之间的关系. 自动装备具有以下优势:

    显著减少需要指定的属性或构造器参数
    自动装配可以随着对象的演化而更新配置，例如需要向类中添加新的依赖项时无需修改任何配置就可以满足依赖项.

基于XML的配置可以通过<bean autowire=''>的属性指定自动装配，它有4种取值：

    no: 不进行自动装配，需要显式的指定引用的对象
    byName: 通过属性名称自动装配. Spring在容器中寻找与属性名称相同的bean进行装配.
    byType: 如果在容器中只有一个相应类型的Bean时自动装配, 如果有多个同类的bean则会抛出异常. 如果没有匹配的bean,则属性不会被设置
    constructor: 类似于byType, 但适用于构造器参数，如果容器中没有此类型的bean, 则会引发致命的错误.

自动装配的限制和缺点

如果项目中始终使用自动装配那是极好的. 如果通常的时候不使用，突然有那么一两个bean使用，那么会造成很多阅读或使用上的疑惑. 自动装配的限制和缺点：

    在property或constructor-arg指定的依赖总司会覆盖自动装配的设置. 不能自动装配简单属性，如基本类型、字符串(这是设计上的限制)。
    自动装配不如显式装配(通过xml等方式)明确.尽管spring会极力的避免猜错以防止出现意外的结果，但spring管理的对象之间的关系不会被记录下来.
    从Spring容器生成文档的工具中可能无法获取到装配的信息.
    容器中的多个bean定义通过指定类型匹配装配时，如果没有唯一的bean可用，则会抛出异常.

在后续的使用场景中，可以选择一下的选项：

    使用显式装配替代自动装配
    使用基于注解的配置实现更细粒度的控制
    通过设置<bean autowire-candidate="false">避免使用自动装配

2.1.6 方法注入

在大多数应用场景中，容器中大多数bean都是单例的. 当一个单例bean使用另一个单例bean，或者一个非单例bean需要组装另一个非单例bean时，通常将其作为属性来处理彼此的依赖。
那么，一个问题本抛出来了：不同的bean的生命周期不同，假设单例bean A需要使用另一个非单例Bean B时，A只会被创建一次也就只有一次设置属性的机会。容器不能为A每次都提供一个新的B。

解决这个问题有很多种方法，spring提供了一种高级特性–Method Injection.

    Lookup method injection（查找方法注入）
    查找方法注入是容器在容器托管bean上重写方法的能力，以便为容器中的另一个命名bean返回查找结果。查找通常涉及一个原型bean.
    SpringFramework通过使用CGLIB库来动态生成重写此方法的子类的字节码来实现的

            为了能使动态生成的子类能工作，当前类不能是final并且方法也不能是final修饰
            单元测试时具有抽象方法的类时需要自己实现此抽象方法
            扫描为组件的类需要具体的方法
            一个关键的限制查找方法不能使用工厂方法. 因为此时容器不负责创建实例，因此不能动态生成子类.

    客户端类的格式：<public|protected> [abstract] <return-type> theMethodName(no-arguments);

    public abstract class CommandManager {

    public Object process(Object commandState) {
        // 获取适合command接口的实例
        Command command = createCommand();
        // 设置command状态（业务逻辑实现）
        command.setState(commandState);
        return command.execute();
    }

    // 获取command实例的方法声明， spring通过cglib动态实现此类
    protected abstract Command createCommand();
    }
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13

    此处的createCommand()方法为抽象方法，动态生成的类实现此方法，否则动态生成的类重写此方法.

    此例子的XML配置：

    <!-- 原型bean -->
    <bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype"></bean>

    <!-- 通过方法注入原型实例 -->
    <bean id="commandManager" class="fiona.apple.CommandManager">
        <lookup-method name="createCommand" bean="myCommand"/>
    </bean>
        1
        2
        3
        4
        5
        6
        7

    Java注解配置：

    public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
    }
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11

    任意方法替换

实现rg.springframework.beans.factory.support.MethodReplacer定义新的方法.

public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        String input = (String) args[0];
        ...
        return ...;
    }
}

<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16

2.2 Bean作用域

bean的作用域主要用来控制单个配置产生的bean实例的数量以及产生的bean实例的作用范围。

spring支持7中作用域，其中5种仅在web的ApplicationContext下起作用.

    singletion: 默认作用域，在每个Spring容器中只有一个定义的bean实例
    prototype: 一个bean定义有多个实例
    request: 一个定义的bean的实例的生命周期为一次http request。仅在web下生效
    session: 一个定义的bean的实例的生命周期为一次http session。仅在web下生效
    globalSession: 一个定义的bean的实例的生命周期为一次global http request。仅在Portlet下生效
    application: 一个定义的bean的实例的生命周期为ServletContext，仅在web下生效
    websocket: 一个定义的bean的实例的生命周期为WebSocket

    spring 3.0 之后，线程作用域也是可用的了. SimpleThreadScope, 通过ThreadLocal实现

    singleton scope

当对象为singleton时，IoC容器创建好bean的实例时，会将其缓存，后续所有的请求和引用都返回缓存的对象.

Spring的单例与单例设计模式有所不同的是：单例模式通过在类内部进行硬编码限制返回一个对象，而Spring通过容器缓存来进行单个对象实例的创建于管理.

    prototype scope

当对象为prototype时，IOC容器会为每次的请求（无论是将bean注入到其他对象中还是通过getBean()方法）都创建bean新的实例将其返回。

singleton的实例适合无状态的bean, 而prototype的实例适合有状态的bean.

spring并没有管理prototype bean的完整的生命周期，容器只对bean进行了实例化、配置以及组装，将其交给客户端之后，spring并没有进一步记录这个实例。
因此，在prototype作用域下实例的销毁生命周期方法并没有被调用. 因而，客户端需要负责释放prototype作用域实例占用的资源. 当然也可以自定义bean的post-processor来交由容器释放.

    Request, session, global session, application, and WebSocket scopes

这些类型的作用域只有在web下生效，如XmlWebApplicationContext，如果使用ClassPathXmlApplicationContext会抛出IllegalStateException异常。

    作用域bean作为依赖

如果想将sope为session的bean注入到生命周期更长的实例中(如scope为singleton的bean), 此时如果用以下方式注入：

    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session"></bean>

    <bean id="userService" class="com.foo.SimpleUserService">
        <property name="userPreferences" ref="userPreferences"/>
    </bean>

    1
    2
    3
    4
    5

由于SimpleUserService的scope是singleton仅会被容器初始化一次，所以userService持有的userPreferences对象无法达到预期，每次使用同一个UserPreferences;

为了解决此问题，可以使用<aop:scoped-proxy/>标签，如：

    <!-- HTTP session 作用域的bean暴露一个代理 -->
    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <!-- 指示容器代理此bean -->
        <aop:scoped-proxy/>
    </bean>

    <!-- 单例作用域的bean注入上边的bean -->
    <bean id="userService" class="com.foo.SimpleUserService">
        <!-- 实际引用userPreferences的代理bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11

当bean包含<aop:scoped-proxy/>容器会创建一个和此bean完全相同接口的代理（基于CGLIB的代理），持有此代理引用的bean在每次调用时，先调用代理的方法，代理会从其scope中获取真正的对象。

    选择创建的代理类型
        基于CGLIB:默认spring容器基于cglib创建代理类，此种代理类只代理public的方法.而非public并不会委托给真实的类.
        基于JDK接口: 此种方式意味着代理的bean必须实现至少一个接口，在bean引用中也都是通过接口进行注入的.

2.3 自定义作用域

可以通过实现org.springframework.beans.factory.config.Scope接口自定义作用域或者重写已存在的作用域(singleton与prototype不能被重写)
2.4 自定义bean的特性
2.4.1 生命周期回调

    bean初始化与销毁回调

可以通过实现InitializingBean和DisposableBean接口参与bean的生命周期交互，容器在初始化bean的时候会调用InitializingBean.afterPropertiesSet(),
在销毁bean的时候会调用DisposableBean.destroy()方法。也可以通过JSR-250定义的注解@PostConstruct与@PreDestroy达到相同的作用.

在Spring内部使用BeanPostProcessor接口实现类来处理回调接口的.如果需要自定义特性或者干预bean生命周期的行为，可以自己实现BeanPostProcessor接口.

除了初始化和销毁回调之外，Spring管理对象还实现了Lifecycle接口，以便这些对象在容器自身的生命周期的驱动下参与启动和关闭的过程.

如果对象的生命周期方法配置了多个，那么如果相同周期内方法名相同，则方法只会调用一次，如果相同周期内方法名不同，则调用顺序如下：
- 初始化调用顺序
- 1. @PostConstruction
- 2. InitializingBean.afterPropertiesSet
- 3. 自定义配置init()
- 销毁调用顺序
- 1. @PreDestroy
- 2. DisposableBean.destroy()
- 3. destroy()

    IoC容器启动与关闭回调

Lifecycle接口定义了基本的生命周期方法:void start(); void stop(); boolean isRunning();

spring管理的任意对象可以实现此接口,然后当ApplicationContext自身接收到启动或停止的信号时,会在其上下文中查找所有的Lifecycle的实现. 这个过程委托给LifecycleProcessor.
LifecycleProcessor本身是Lifecycle的扩展接口，增加了上下文刷新和关闭的响应方法。。

需要注意的是Lifecycle接口只是一个普通的启动/停止通知的契约，但并不意味着在上下文刷新时自动启动。考虑实现SmartLifecycle在指定bean自动启动时进行细粒度的控制.

如果对象之间有依赖关系，这个时候启动或关闭的顺序就非常重要了, 而SmartLifecycle接口继承了Phased接口的方法getPhase(), 此方法会返回一个Integer.MAX_VALUE到Integer.MIN_VALUE范围的阶段值。 当启动的时候，阶段值最低的先启动；当关闭的时候，阶段值最高的先关闭。而实现了Lifecycle而没有实现SmartLifecycle接口的对象默认的阶段值为0

如何在非web应用中优雅关机？

ConfigurableApplicationContext接口定义了一个registerShutdownHook()方法，此方法的默认实现是在JVM上注册一个关闭钩子(shutdown hook),以实现优雅关机。

public void registerShutdownHook() {
        if (this.shutdownHook == null) {
            this.shutdownHook = new Thread() {
                @Override
                public void run() {
                    synchronized (startupShutdownMonitor) {
                        doClose(); // 具体关闭逻辑
                    }
                }
            };
            Runtime.getRuntime().addShutdownHook(this.shutdownHook);
        }
    }

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13

2.4.2 ApplicationContextAware and BeanNameAware

当ApplicationContext创建实现了ApplicationContextAware接口的类时，会调用void setApplicationContext(ApplicationContext applicationContext)提供ApplicationContext的引用。

当ApplicationContext创建实现了BeanNameAware接口的类时，会调用void setBeanName(String name)提供bean在BeanFactory的唯一Id;

这样在需要获取ApplicationContext的时候或者获取bean的时候不需要编写冗余的代码，但随之而来的是违反了IoC的风格，将bean与spring耦合在了一起.
2.4.3 其他感知接口

除了ApplicationContextAware与BeanNameAware接口外，Spring还提供了一系列的感知接口，如下：

    ApplicationContextAware: ApplicationContext，void setApplicationContext(ApplicationContext applicationContext)
    ApplicationEventPublisherAware: ApplicationContext事件发布, void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher)
    BeanClassLoaderAware: 加载bean类的类加载器, void setBeanClassLoader(ClassLoader classLoader)
    BeanFactoryAware: BeanFactory, void setBeanFactory(BeanFactory beanFactory)
    BeanNameAware: void setBeanName(String name);
    BootstrapContextAware: 容器运行的资源适配器BootstrapContext,仅在JAC(J2EE连接器)下使用,void setBootstrapContext(BootstrapContext bootstrapContext);
    LoadTimeWeaverAware: 类定义加载流程时定义weaver, void setLoadTimeWeaver(LoadTimeWeaver loadTimeWeaver)
    MessageSourceAware: 为解析信息（国际化或参数化）配置策略，void setMessageSource(MessageSource messageSource)
    NotificationPublisherAware: Spring JMX 通知发布者， void setNotificationPublisher(NotificationPublisher notificationPublisher)
    PortletConfigAware: 当前运行容器的PortletConfig, void setPortletConfig(PortletConfig portletConfig);
    PortletContextAware: 当前运行容器的PortletContext, void setPortletContext(PortletContext portletContext)
    ResourceLoaderAware: 配置用于访问低级资源的加载器， void setResourceLoader(ResourceLoader resourceLoader)
    ServletConfigAware: 当前运行容器的ServletConfig, void setServletConfig(ServletConfig servletConfig)
    ServletContextAware: 当前运行容器的ServletContext, void setServletContext(ServletContext servletContext)

使用以上的接口会破坏IOC的风格，因此，建议将实现以上接口的bean作为基础设施
2.5 容器扩展点
2.5.1 使用BeanPostProcessor自定义bean

如果想在容器完成bean的初始化、配置和实例化之前加入一些自己的逻辑，那么可以通过插入一些BeanPostProcessor接口实现类来达到目的。

可以有多个BeanPostProcessor的实例，并且可以通过实现Ordered接口设置其顺序.

当一个BeanPostProcessor实例被注册到容器中后，对于容器中创建的每个bean在其初始化之前之后都会进行回调.一些AOP的基础设施类也是通过实现为BeanPostProcessor来提供代理逻辑.

如果在bean的配置元数据的时候，bean本身实现了BeanPostProcessor，spring容器会自动将其注册到容器中以便后续调用。

使用回调接口或者结合注解自定义BeanPostProcessor是IoC容器通常的扩展方式. 例如Spring的RequiredAnnotationBeanPostProcessor就是BeanPostProcessor的实现，它的作用是确保Java Bean被注解标记的属性是一个可以依赖注入的值.
2.5.2 使用BeanFactoryPostProcessor自定义配置元

BeanFactoryPostProcessor与BeanPostProcessor重要的不同之处是：BeanFactoryPostProcessor操作的是bean的配置元数据，Spring允许BeanFactoryPostProcessor在容器实例化bean之前读取并修改配置元数据.

BeanFactoryPostProcessor可以配置多个，也可以通过实现Ordered接口设置顺序, 但不能将BeanFactoryPostProcessor配置为延迟加载，否则不起作用.

在Spring容器中定义的BeanFactoryPostProcessor可以自动被ApplicationContext执行，Spring中包含了一些农民人的实现，如PropertyOverrideConfigure与PropertyPlaceholderConfigure.
2.5.3 使用FactoryBean自定义实例化逻辑

FactoryBean接口是IoC实例化逻辑的配置点. 如果有复杂的实例化逻辑，可以创建自己的FactoryBean来实现，并将其插入到容器中. FactoryBean有三个方法：

public interface FactoryBean<T> {
  T getObject() throws Exception;
  Class<?> getObjectType();
  boolean isSingleton();
}

    1
    2
    3
    4
    5

    getObject() 返回当前工厂创建的对象实例，这个实例有可能是共享的
    getObjectType() 返回getObject()实例的类型，如果预先不知道则返回null
    isSingleton() 返回的对象实例是否单例

Spring中大量使用了(超过50个地方)FactoryBean接口，如果想获取产生bean的实际FactoryBean,可在通过在beanName之前加&获取，如:ApplicaionContext.getBean("&beanName").
2.6 运行环境抽象

Environment是集成在容器应用运行环境的抽象，有两个关键模型：profiles与properties.

profile是一个具有名称的逻辑组，它的作用是对bean定义分组，然后在IoC容器启动的时候根据激活的profile注册对应组的bean定义. Bean可以通过XML或者注解被分配到profile上。
而Environment决定了哪些profile当前是被激活的，哪些profile是需要被默认激活的.

properties对于大多数应用来说是一个比较重要的角色，它可能有多个来源：properties文件、JVM系统参数、系统环境变量、JNDI、servlet上下文参数、Map等等.
Enviroment提供了很方便的用于用户进行配置和解析这些属性的接口服务。
2.6.1 bean definition profiles

bean definition profiles是核心容器允许在不同的环境中加载不同bean的一种机制。不同用户对于环境的理解千差万别，一般的使用案例有：

    开发的时候使用内存数据库，QA或者生产环境上使用JNDI的数据源
    在部署到正式坏境中才注册监控设施
    为客户A和客户B注册不同的自定义bean实现

注解定义profile:

    @Bean("dataSource")
    @Profile("development")
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    1
    2
    3
    4
    5
    6
    7
    8
    9

XML定义profile:

<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11

激活profile:

AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();

    1
    2
    3
    4

2.6.2 PropertySource抽象

Spring的Environment抽象提供了对属性资源配置可进行层次查询操作. 在获取一个属性时，Environment回去PropertySource中查找.

PropertySource是一个简单的键值对抽象，StandardEnvironment默认会配置两种PropertySource:JVM参数与系统环境变量：

public class StandardEnvironment extends AbstractEnvironment {

    public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
    public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";

    protected void customizePropertySources(MutablePropertySources propertySources) {
        propertySources.addLast(new MapPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
        propertySources.addLast(new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
    }
}

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10

    StandardEnvironment为单体应用的定义了默认属性资源， StandardServletEnvironment为web应用定义了默认属性资源(除默认资源外，还有servlet配置与servlet上下文属性)

属性值搜索是分层的，默认情况下jvm系统参数优先于操作系统环境变量. 需要注意的是属性值不会被合并，只会优先于它的配置覆盖.

属性分级查找例子：StandardServletEnvironment中属性查找顺序：

        ServletConfig参数 
        ServletContext参数
        JNDI环境变量
        JVM系统参数(通过’-D’命令行添加的参数)
        操作系统环境变量

MutablePropertySources暴露了API用于添加或者修改属性资源以及其查找顺序.

@PropertySource注解可用于标记类需要使用到的属性资源.
2.6 类加载期织入

LoadTimeWeaver用于在JVM加载类的时候动态改变类.

    在java中，从织入切面的方式上来看，一般由三种织入方式：编译期织入，类加载期织入, 运行期织入。
    编译期织入是指在类编译期，采用特殊编译器将切面织入到勒种，如AspectJ通过maven或ant来实现.
    类加载期织入是指在类字节码加载到JVM的时候，采用特殊的类加载器织入切面
    运行期织入是指在运行时通过CGLIB工具或者动态代理的方式织入切面.

启用方式：java注解@EnableLoadTimeWeaving或者xml配置<context:load-time-weaver/>.

JPA中使用到了此种方式，LocalContainerEntityManagerFactoryBean.
2.7 ApplicationContext的附加功能

org.springframework.beans.factory包提供了管理与操作bean的基础功能； org.springframework.context提供了继承自BeanFactory的ApplicationContext接口，它除了扩展其他接口外还提供了更多面向框架编程风格的功能.

为了增强BeanFavtory功能以更面向框架的风格，org.springframework.context还增加了以下功能：

    通过MessageSource接口访问i18n风格的国际化资源
    通过ResourceLoader接口访问来自于URL或者文件资源
    通过实现ApplicationListener监听ApplicationEventPublisher接口发布的事件
    加载多层次上下文，允许每一层聚焦于自己的功能.

2.7.1 标准事件与自定义事件

ApplicationContext提供了ApplicationEvent类与ApplicationListener接口来处理事件. 如果Bean实现了ApplicationListener接口，那么每次ApplicationZEvent被ApplicationContext发布时,这个bean会被通知到, 大体上类似于观察者设计模式.

    从4.2的版本开始，事件得到了显著的改进，提供更好的基于注解模型发布事件。bean不需要继承ApplicationEvent了.

Spring提供的标准事件：

    ContextRefreshedEvent: 当ApplicationContext被初始化或者被刷新时被通知。即所有的bean被加载，post-processor bean被检验以及被激活，单例bean已被预先创建，ApplicationContext已经准备好可以使用了. 只要ApplicationContext没有被关闭，就可以触发多次刷新.
    ContextStartedEvent: 当ApplicationContext被启动的时候发布. 被启动以为这所有Lifecyclebean都接收到显式的启动信号。
    ContextStoppedEvent: 当ApplicationContext被停止时发布。被停止意味着所有Lifecyclebean接收到显式的停止信号。
    ContextClosedEvent: 当ApplicationContext被关闭时发布，被关闭意味着所有单例bean被销毁，上下文到达生命的终点，不会被刷新或者被重启
    RequestHandledEvent: web特有事件，当请求完成之后事件被发布

可以通过@EventListener注解监听事件，@Async注解可以异步监听, @Order可以设置触发顺序.
3. 资源(Resources)

Java本身的java.net.URL类为处理各种前缀URL提供了一些标准，但不幸的是不能够满足对低级资源的访问.

Spring的Resource接口用于抽象访问低级资源的能力.
3.1 资源接口

public interface Resource extends InputStreamSource {
    boolean exists();
    boolean isReadable();
    boolean isOpen();
    URL getURL() throws IOException;
    URI getURI() throws IOException;
    File getFile() throws IOException;
    long contentLength() throws IOException;
    long lastModified() throws IOException;
    Resource createRelative(String relativePath) throws IOException;
    String getFilename();
    String getDescription();
    InputStream getInputStream() throws IOException; // 来源于InputStreamSource接口
}

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14

这个接口有一些重要的方法：

    getInputStream(): 定位并打开资源，返回一个InputStream，调用者负责关闭输入流
    exists()返回boolean值指明此资源是否真实的存在
    isOpen()返回boolean值指明是否打开了一个流的句柄。一个InputStream不能被使用多次，使用完必须关闭以避免资源泄露.
    getDescription(): 返回资源描述，用于资源出错时输出. 通常是资源名称或资源URL.

Spring中广泛的使用了Resource抽象，当需要资源时，Resource作为方法的签名参数类型，还要一些Spring的API（如ApplicaitonContext的实现类）,提供一个简单的字符串，由上下来适配一个Resource,或者通过制定特殊的字符串前缀来指定Resource的实现类.

Reource在不仅Spring中大量使用，自己的代码中也可以使用Resource来替代URL(即使不使用Spring的其他部分).
3.2 内置的资源实现

    UrlResource: 包装了java.net.URL，可以通过URL访问任何对象，例如文件、http、ftp等，所有的URL都有一个标准的字符串表达方式，前缀代表URL类型，如file:代表文件路径，http:代表http协议的资源, ftp:代表访问FTP资源
    ClassPathResource: 这个类表明应该从classpath加载资源. 使用线程上下文的类加载器、给定的类加载器或者更给定的类加载资源
    FileSystemResource: 处理java.io.File的一个Resource实现
    ServletContextResource: ServletContext资源的实现，用于加载web应用根目录的相对路径下的资源
    InputStreamResource: InputStream资源的实现，仅仅只用于没有指定Resource的实现的时候. 与其他资源实现相比，这个描述的是已经打开的资源。不适用于多次读取流时使用.
    ByteArrayResource: 给定的字节数组的资源实现，可以从给定的字节数组中创建一个ByteArratInputStream

3.3 资源加载器(ResourceLoader)

所有的ApplicationContext都实现了ResourceLoader接口，因此ApplicationConetxt可以获取Resource实例。当调用getReource()方法时，如果路径没有指定前缀，那将会获得一个适应于上下文的Resource. 如ClassPathXmlApplicationContext获得ClassPathResource，WebApplicationContext获取的ServletContextResource等的。
3.4 ResourceLoaderAware接口

实现ResourceLoaderAware并将其注册到ApplicationContext后，应用上下文就会执行setResourceLoader(ResourceLoader)方法，提供ResourceLoader引用。

之前也讲过，ApplicationConetxt实现了ResourceLoader，所以它本身就是一个ResourceLoader，所以也可以实现ApplicationContextAware获取到ApplicationContext将其转换为ResourceLoader,但通常不建议这样做，因为很多时候仅仅只是获取ResouceLoader而不是获取整个ApplicationContext

以上为根据官方文档整理Spring最核心的概念，但并不是全部核心概念(除此之外还包含数据绑定/验证、SpEl、**AOP**)，在后续分析的时候会重新对其进行分析。
--------------------- 
作者：胖蚂蚁_alleyz 
来源：CSDN 
原文：https://blog.csdn.net/u010209217/article/details/80617310 
版权声明：本文为博主原创文章，转载请附上博文链接！
