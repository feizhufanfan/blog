---
title: spring学习--spring核心原理
date: 2025-07-14 14:14:13
categories:
- spring
- spring核心原理
tags:
- java
- spring
---



# 一、Spring容器核心原理
Spring容器的核心职责是 **管理Bean的生命周期** 和 **提供依赖注入(DI)** 能力。其工作原理可概括为以下几个步骤： 
  1. **配置元数据读取**  
     - 容器需要知道如何创建和配置Bean。这些信息来自配置元数据，通常来源于：
       - XML配置文件 (`applicationContext.xml`)
       - Java注解 (`@Component`, `@Autowired`, `@Bean`等)
       - Java配置类 (`@Configuration` + `@Bean`)
     - 容器会读取并解析这些配置元数据。
  2. **实例化Bean**  
     - 根据配置元数据中定义的Bean定义（**BeanDefinnition**）,容器创建Bean的实例。这通常通过Java的反射机制完成（调用构造函数或工厂方法）。   
  3. **依赖注入**  
     - 容器分析Bean之间的依赖关系（通过构造函数参数、**Setter**方法或字段上的`@Autowired`等注解指定）。
     - 容器查找或被依赖的Bean实例。
     - 容器将依赖的Bean实例**注入**到需要它们的Bean中（设置属性、传入构造函数参数等）。这是实现**控制反转（IoC）** 的和兴步骤-对象的依赖关系有容器管理，而不是对象自己创建或查找依赖。
  4. **初始化回调**  
      - 如果Bean实现了特定的生命周期接口（如`InitializingBean`）或在配置中指定了初始化方法（`ini-method`属性或`@PostConstruct`注解），容器会在依赖注入完成后调用这些方法，执行自定义的初始化逻辑。  
  5. **Bean就绪可用**  
     - 此时Bean已被完全配置好，依赖已注入，初始化已完成，可以被应用程序代码使用了（通常通过`getBean()`方法获取，或通过注入方式获得）。  
  6. **注销回调**   
     - 容器关闭时（例如调用`close()`方法），如果Bean实现了特定的销毁接口（如`DisposableBean`）或在配置中制定了销毁方法（`destroy-method`属性或`@PreDestroy`注解），容器会调用这些方法，执行自定义的清理逻辑。   

核心思想总结:Spring容器充当了“Bean工厂”和“依赖装配器”的角色。它接管了传统应用中由对象自身负责创建和查找议案对象的职责，实现了控制反转（IoC）。开发者只需要定义Bean以及依赖关系，容器负责创建，组装和管理它们的整个生命周期。  

# 二、BeanFactory vs ApplicationContext  
`BeanFactory`和`ApplicationContext`都是Spring IoC容器的核心接口，用于管理和访问Bean。`ApplicationContext`是`BeanFactroy`的一个功能更丰富的**子接口**。理解他们的区别至关重要。
  1. **BeanFactory(基础容器接口)**  
     - **定位：** 是Spring IoC容器最基础的接口。它定义了容器最基本的功能：**获取**Bean。
     - **核心功能：**
       - `getBean()`:根据名称或类型获取Bean实例。  
       - `containsBean()`:判断容器是否包含指定名称的Bean。
       - `isSingleton()`/`isPrototype()`:判断Bean的作用域。
       - `getAliases()`:获取Bean的别名。  
       - `getType()`:获取Bean的类型。  
     - **特点** ：  
       - **轻量级** ：只提供核心的IoC功能（Bean定义、加载、实例化、基本的生命周期管理）。
       - **懒加载（Lazy InitiaLization）是默认行为：** 只有当应用程序显示调用`getBean()`请求某个Bean时，容器才会实例化并配置该Bean以及其依赖（除非该Bean是其他非懒加载Bean的依赖）。这有助于减少启动时  的资源消耗。  
       - **需要显示注册：** 通常需要手动创建`BeanFactory`实例（如`XmlBeanFactory`- *注意：这个类Spring 5.1中已被标记为废弃，推荐使用`ApplicationContext`*）并加载配置文件。  
     - **适用场景：** 对资源（如内存）非常敏感的环境（例如移动设备或小型Applet应用），或者不需要`ApplicationContext`提供的高级功能时。**在现代基于Spring Boot的应用程序中，直接使用BeanFactory的情况非常罕见。**  
  2. **ApplicationContext（高级容器接口）**
     - **定位:**构建在**BeanFactory** 基础之上，是功能更**全面**，面向企业应用的容器接口。它继承了**BeanFactory**的所有功能，并添加了大量企业级特性。  
     - **核心增强功能（超越BeanFactory）：**
     - **更丰富的Bean生命周期管理：**
       - 自动检测并调用**BeanPostProcessor**（允许在Bean初始化前后进行自定义处理）。  
       - 自动检测并调用**BeanFactoryPostProcessor**（允许在容器加载Bean定义之后，实例化Bean之前修改Bean定义）。
       - 更便捷地管理`@PostConstruct`和`@PreDestroy`等标准注解的生命周期回调。
     - **国际化（i18n）支持：** 通过`MessageSource`接口轻松处理不同语言的消息。  
     - **资源访问（Resource Access）：** 提供同意的方式访问资源文件（如URL、类路径、文件系统中的资源），通过**ResouceLoader**/**Resource** 接口。
     - **事件发布/监听机制（Event Publication）：**支持基于`ApplicationEvent`和`ApplicationListener`接口的**应用事件** 机制，实现Bean之间的松耦合通信。  
     - **更方便的AOP集成：** 简化了面向切面编程（AOP）的配置和使用。  
     - **集成Web应用（WebApplicationContext）：** 为Web应用提供特定功能（如请求、会话、应用作用域的Bean）。 
     - **加载配置方式的多样性：** 提供多种实现类，支持从类路径（`ClassPathXmlApplicationContext`）、文件系统（`FileSystemXmlApplicationContext`）、注解（`AnnotationConfigApplicationContext`）等多种方式加载配置。  
     - **自动识别配置类：** 支持基于Java注解（如`@Configuration`、`@ComponentScan`）的配置方式。
     - **特定：**
       - **重量级(相对BeanFactory)：** 提供了更多功能，因此启动时加载和初始化的内容更多。  
       - **默认即时加载/预实例化单例Bean(Eager Initialization)：** 在容器启动过程中，它会主动加载配置元数据，并立即实例化所有配置为单例(`singleton`)且非懒加载(`lazy-init="false"`)的Bean及其依赖（除非显式配置为懒加载）。这会导致启动时间稍长，但能尽早发现配置错误，并且首次请求Bean时响应更快。  
       - **适用场景：绝大多数现代Spring应用程序的标准选择。** 它提供了开箱即用的企业级特性，简化了开发，是Spring Boot默认使用的容器类型。  

# 三、**核心区别对比表**  
|**特性**|**BeanFactory**|**ApplicationContext**|
|:---|:---|:---|
|**接口级别**| **基础接口** |**BeanFactory**的子接口，功能更丰富|
| **核心接口** | **基本的Bean生命周期管理、依赖查找** | **所有BeanFactory功能 + 企业级扩展** |
| **Bean实例化默认策略** | **懒加载（Lazy）-按需创建** | **即时加载(Eager)- 启动时创建单例非懒加载Bean** |
| **生命周期管理扩展有限** | **有限** | **完整支持** (`BeanPostProcessor`,` @PostConstruct`等) |
| **国际化（i18n）** | **不支持** | **支持**(MessageSource) |
| **资源访问** | **不支持** | **支持**(ResourceLoader/Resource) |
| **事件机制** | **不支持** | **支持**(事件发布/监听) |
| **AOP集成** | **需要更多手动配置** | **简化集成**|
| **Web应用支持** | **不支持** | **支持**(WebApplicationContext) |
| **注解配置支持** | **有限**| **原生强大支持**(`@Configuration`, `@ComponentScan`) |
| **典型实现类** | **XmlBeanFactory**（已废弃），**DefaultListableBeanFactory** |`ClassPathXmlApplicationContext`, `FileSystemXmlApplicationContext`, `AnnotationConfigApplicationContext`, `AnnotationConfigWebApplicationContext`|
| **启动速度** | **相对较快**（只加载基本配置）| **相对较慢**(加载更多配置并实例化Bean)| 
| **资源占用** | **更低** | **更高** |
| **适用场景** | **资源极度受限环境** | **绝大多数企业级应用和现代Spring应用** |  



  4. **简单代码示例**  

```java
// 使用 ApplicationContext (推荐)
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class); // 基于Java配置类
// ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml"); // 基于XML

MyService myService = context.getBean(MyService.class); // 获取Bean
myService.doSomething();

// 使用 BeanFactory (不常用，且XmlBeanFactory已废弃，此处仅作演示)
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("beans.xml")); // 已废弃!
MyComponent comp = factory.getBean(MyComponent.class);
```


# 四、总结
  - BeanFactory 是Spring IoC容器的**基础规范**，提供最核心的Bean管理和依赖查找功能。它**默认懒加载**，轻量但功能有限。
  - **ApplicationContext** 是**BeanFactory**的**超级接口**，在继承其所有功能的基础上，添加了**全面的企业级支持**（国际化、资源访问、事件、AOP、Web集成、便捷的配置方式等）。它**默认即时加载单例Bean**，启动稍慢但功能强大，能及早发现配置错误。
  - **现代Spring开发中，`ApplicationContext`是绝对的主流和首选。** 除非你有极端的资源限制需求，否则都应该使用`ApplicationContext`（或其特定环境的子接口，如`WebApplicationContext`）。**Spring Boot**自动配置的核心就是基于`AnnotationConfigApplicationContext`或其Web变体构建的。      

    

  

