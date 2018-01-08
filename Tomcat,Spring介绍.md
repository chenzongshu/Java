
https://www.zybuluo.com/mdeditor#823560 

http://blog.csdn.net/qq_22654611/article/details/52606960

spring Web MVC是一种基于Java的实现了Web MVC设计模式的**请求驱动类型**的轻量级Web框架.

# 容器

## servlet容器
负责管理servlet生命周期。

## web容器–tomcat
负责管理和部署web应用，其本身可能具备servlet容器组件；如果没有，一般能将第三方servlet容器作为组件整合进web容器。 

web容器好比”电视机”，servlet容器好比”VCD”。没有VCD你可以看电视，但是有了VCD没有电视机，你从哪看起？ 没有servlet容器，你也可以用web容器直接访问静态页面，比如安装一个apache等，但是如果要显示jsp/servlet，你就要安装一个servlet容器了，但是光有servlet容器是不够的，因为它要被解析成html输出，所以你仍需要一个web容器.大多数servlet容器同时提供了web容器的功能，也就是说大多servelt可以独立运行你的web应用。

## IOC容器–Spring Bean Factory
BeanFactory负责配置、创建、管理Bean，他有一个子接口：ApplicationContext，因此也称之为Spring上下文。Spring容器负责管理Bean与Bean之间的依赖关系。

## Spring容器–Spring Application Context

# Tomcat

## 组成

Tomcat的核心组件就两个Connector和Container（后面还有详细说明），一个Connector+一个Container构成一个Service，Service就是对外提供服务的组件，有了Service组件Tomcat就可以对外提供服务了，但是光有服务还不行，还得有环境让你提供服务才行，所以最外层的Server就为Service提供了生存的土壤。那么这些个组件到底是干嘛用的呢？Connector是一个连接器，主要负责接收请求并把请求交给Container，Container就是一个容器，主要装的是具体处理请求的组件。Service主要是为了关联Container与Connector，一个单独的Container或者一个单独的Connector都不能完整处理一个请求，只有两个结合在一起才能完成一个请求的处理。Server这是负责管理Service集合，从图中我们看到一个Tomcat可以提供多种服务，那么这些Serice就是由Server来管理的，具体的工作包括：对外提供一个接口访问Service，对内维护Service集合，维护Service集合又包括管理Service的生命周期、寻找一个请求的Service、结束一个Service等。

### Connector 组件
是 Tomcat 中两个核心组件（另一个是Container组件）之一，它的主要任务是负责接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 Request 和 Response 对象传给处理这个请求的线程，处理这个请求的线程就是 Container 组件要做的事了。

TOMCAT有两个典型的Connector，一个直接侦听来自browser的http请求，一个侦听来自其它WebServer的请求：

Coyote Http/1.1 Connector 在端口8080处侦听来自客户browser的http请求；
Coyote JK2 Connector 在端口8009处侦听来自其它WebServer(Apache)的servlet/jsp代理请求。

### Container 组件
Container 有四个子组件构成，分别是：Engine、Host、Context、Wrapper，这四个组件不是平行的，而是父子关系，Engine 包含 Host,Host 包含 Context，Context 包含 Wrapper。

Engine下可以配置多个虚拟主机Virtual Host，每个虚拟主机都有一个域名。当Engine获得一个请求时，它把该请求匹配到某个Host上，然后把该请求交给该Host来处理。Engine有一个默认虚拟主机，当请求无法匹配到任何一个Host上的时候，将交给该默认Host来处理。

Host代表一个Virtual Host，虚拟主机。每个虚拟主机下都可以部署(deploy)一个或者多个Web App，每个Web App对应于一个Context，有一个Context path。当Host获得一个请求时，将把该请求匹配到某个Context上，然后把该请求交给该Context来处理。匹配的方法是“最长匹配”，所以一个path==””的Context将成为该Host的默认Context，所有无法和其它Context的路径名匹配的请求都将最终和该默认Context匹配。

Context 代表 Servlet 的 Context。一个Context对应于一个Web Application，一个Web Application由一个或者多个Servlet组成。Context在创建的时候将根据配置文件`$CATALINA_HOME/conf/web.xml`和`$WEBAPP_HOME/WEB-INF/web.xml`载入Servlet类。当Context获得请求时，将在自己的映射表(mapping table)中寻找相匹配的Servlet类，如果找到，则执行该类，获得请求的回应，并返回。简单的 Tomcat 可以没有 Engine 和 Host。Context 最重要的功能就是管理它里面的 Servlet 实例，Servlet 实例在 Context 中是以 Wrapper 出现的

Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。Wrapper是最底层的容器，它没有子容器了，所以调用它的 addChild 将会报错。Wrapper 的实现类是 StandardWrapper，StandardWrapper 还实现了拥有一个 Servlet 初始化信息的 ServletConfig。

## tomcat处理一个http请求的过程

假设来自客户的请求为：
http://localhost:8080/wsota/wsota_index.jsp

1) 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector获得
2) Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应
3) Engine获得请求localhost/wsota/wsota_index.jsp，匹配它所拥有的所有虚拟主机Host
4) Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）
5) localhost Host获得请求/wsota/wsota_index.jsp，匹配它所拥有的所有Context
6) Host匹配到路径为/wsota的Context（如果匹配不到就把该请求交给路径名为””的Context去处理）
7) path=”/wsota”的Context获得请求/wsota_index.jsp，在它的mapping table中寻找对应的servlet
8) Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类
9) 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
10)Context把执行完了之后的HttpServletResponse对象返回给Host
11)Host把HttpServletResponse对象返回给Engine
12)Engine把HttpServletResponse对象返回给Connector
13)Connector把HttpServletResponse对象返回给客户browser

# spring

spring的核心容器实现了Ioc,其目的是提供一种无侵入式的框架。

在Spring中，有2个最基本最重要的包，即`org.springframework.beans`和`org.springframework.context`.在这两个包中实现了无侵入式的框架，代码中大量引用了Java的反射机制，通过动态调用的方式避免了硬编码，为spring的反向控制特性提供了基础。在这2个包中，最重要的类是**`BeanFactory`**和**`ApplicationContext`**

- BeanFactory提供了一种先进的配置机制来管理任何种类的bean。
- ApplicationContext建立在BeanFactory之上，并增加了其他功能，如国际化，获取资源，事件传递等。

## IoC

Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。

在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。如何理解好Ioc呢？理解好Ioc的关键是要明确“谁控制谁，控制什么，为何是反转（有反转就应该有正转了），哪些方面反转了”，那我们来深入分析一下：

- 谁控制谁，控制什么：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；谁控制谁？当然是IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。

- 为何是反转，哪些方面反转了：有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。

![](http://sishuok.com/forum/upload/2012/2/19/a02c1e3154ef4be3f15fb91275a26494__1.JPG)

使用IoC之后：
![](http://sishuok.com/forum/upload/2012/2/19/6fdf1048726cc2edcac4fca685f050ac__2.JPG)

IoC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC容器来创建并注入它所需要的资源了。

IoC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。

### Ioc和DI

DI—Dependency Injection，即“依赖注入”：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

IoC和DI由什么关系呢？其实它们是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC而言，“依赖注入”明确描述了“被注入对象依赖IoC容器配置依赖对象”。

IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。那么DI是如何实现的呢？ Java 1.3之后一个重要特征是反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性，spring就是通过反射来实现注入的。



## controller

在SpringMVC 中，控制器Controller负责处理由DispatcherServlet分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。在SpringMVC中提供了一个非常简便的定义Controller的方法，你无需继承特定的类或实现特定的接口，只需使用`@Controller`标记一个类是Controller，然后使用`@RequestMapping` 和`@RequestParam` 等一些注解用以定义URL请求和Controller方法之间的映射，这样的Controller就能被外界访问到。此外Controller 不会直接依赖于HttpServletRequest 和HttpServletResponse 等HttpServlet 对象，它们可以通过Controller 的方法参数灵活的获取到


## Bean

### 含义

1、Java面向对象，对象有方法和属性，那么就需要对象实例来调用方法和属性（即实例化）

2、凡是有方法或属性的类都需要实例化，这样才能具象化去使用这些方法和属性；
 
3、规律：凡是子类及带有方法或属性的类都要加上注册Bean到Spring IoC的注解；

4、把Bean理解为类的代理或代言人（实际上确实是通过反射、代理来实现的），这样它就能代表类拥有该拥有的东西了

5、我们都在微博上@过某某，对方会优先看到这条信息，并给你反馈，那么在Spring中，你标识一个@符号，那么Spring就会来看看，并且从这里拿到一个Bean或者给出一个Bean

## Bean的实现

1、可以通过配置xml来实现bean；
2、可以使用注解来配置bean（从Spring 3.0开始）；
3、基于Java类配置；

![](http://img.blog.csdn.net/20150420153816435)

### xml中bean的基本概念

```
<bean id="helloWorld" class="com.name.HelloWorld">  
........  
<span style="color:#000000;"> </bean></span>  
```

#### id/name

id:指定在benafactory中管理该bean的唯一的标识。
name可用来唯一标识bean 或给bean起别名。

#### class

class属性指定了bean的来源，即bean的实际路径。注意要指定全路径，而不可只写类名。

#### Singleton的使用

在spring中，bean可被定义为2中部署模式中的一种。singleton和prototype模式。

singloeton:只有一个共享的实例存在，所有对这个bean的请求都会返回这个唯一实例。
prototype:对这个bean的每次请求都会都会创建一个新的bean实例。根据已经存在的bean而clone出来的bean。默认为singleton模式

```
<bean id="student3" class="com.mucfc.beanfactory.Student" scope="prototype">  
```

#### bean的属性

spring中，bean的属性值有2种注入方式。setter注入和构造函数注入。

setter注入是在调用无参的构造函数或无参的静态工厂方法实例化配置文档中定义的bean之后，通过调用bean上的setter方法实现的。

构造函数的依赖注入是通过调用带有很多参数的构造方法实现的，每个参数表示一个对象或者属性。

#### 使用依赖depends-on

```
<bean id="school" class="com.mucfc.beanfactory.School"  
    depends-on="student6">  
    <property name="student" ref="student6" />  
</bean>  
```

### 基于注解的Bean

1、@controller 控制器（注入服务）
2、@service 服务（注入dao）
3、@repository dao（实现dao访问）
4、@component （把普通pojo实例化到spring容器中，相当于配置文件中的<bean id="" class=""/>）

其他:
@DependsOn：定义Bean初始化及销毁时的顺序
@Scope：定义Bean作用域，默认单例
