---
title: spring学习--依赖注入
date: 2025-07-23 11:20:33
categories:
- spring
- 依赖注入
tags:
- java
- spring
---


# Spring核心基础--依赖注入（构造器注入、Setter注入、字段注入）
## 一、依赖注入（Dependency Injection, DI）概述
依赖注入是Spring框架实现**控制反转（IoC）** 的主要技术手段。它的核心思想是：
*   **传统方式：** 对象自己负责创建或查找它所依赖的对象（主动控制）。对象主动创建依赖（强耦合）
*   **代码示例**
    ```java
    // 传统方式：对象自己创建依赖
    class Service {
        private Repository repo = new DatabaseRepository(); // 直接实例化
    }

    ```
*   **依赖注入：** 对象的依赖关系由外部容器（如Spring）在创建对象时**注入**（被动接受）。对象只需要声明它需要什么依赖，而不需要关心如何获取它们。依赖由外部容器提供（解耦）
*   **代码示例**
    ```java
    // DI方式：依赖从外部注入
    class Service {
        private Repository repo;
        // 通过构造器/Setter/字段接收依赖
    }

    ```
### 依赖注入的好处
1.  **解耦（Decoupling）：** 对象不需要知道其依赖的具体实现或如何创建它们，只依赖于接口或抽象。这降低了类之间的耦合度。
2.  **可测试性（Testability）：** 可以轻松地用模拟对象（Mock）替换实际依赖，便于进行单元测试。
3.  **可维护性（Maintainability）：** 配置集中管理，修改依赖关系通常只需要改动配置（注解或XML），而不需要修改使用这些依赖的类代码。
4.  **可重用性（Reusability）：** 组件更易于在不同环境中重用，因为它们不负责管理自己的依赖。

--- 

## 二、三种依赖注入方式详解与比较
Spring支持三种主要的依赖注入方式：
### 1. 构造器注入（Constructor Injection）
*   **原理：** 通过类的**构造函数**来注入依赖。Spring容器在创建Bean时，会调用带有需要注入依赖参数的构造函数。
*   **实现方式：**
    *   **XML配置：** 使用`<constructor-arg>`元素。
    *   **Java注解：** 在构造方法上使用`@Autowired`（Spring 4.3以后，如果类只有一个构造器，可以省略`@Autowired`）。
    *   **Java配置类：** 在`@Bean`方法中显式调用构造函数并传入参数。
*   **代码示例：**
    ```java
    public class OrderService {
        private final PaymentProcessor paymentProcessor; // 推荐使用final确保不变性
        private final InventoryService inventoryService;
    
        // 构造器注入（Spring 4.3+ 单个构造器可省略@Autowired）
        @Autowired
        public OrderService(PaymentProcessor paymentProcessor, InventoryService inventoryService) {
            this.paymentProcessor = paymentProcessor;
            this.inventoryService = inventoryService;
        }
    
        public void processOrder(Order order) {
            // 使用paymentProcessor和inventoryService处理订单
        }
    }
    ```
*   **优点：**
    *   **不可变性（Immutability）：** 可以将依赖字段声明为`final`，确保它们在对象创建后被正确初始化且不可变。这增强了线程安全性和对象状态的稳定性。
    *   **完全初始化的状态：** 对象一旦创建，其所有必需的依赖都已就绪，保证了对象在使用的任何时候都处于一个完整、一致的状态（避免了部分注入导致的状态不一致）。
    *   **明确依赖：** 构造函数清晰地声明了创建对象所必需的所有依赖项。这使代码意图更明确，也方便在单元测试中手动构造对象（无需容器）。
    *   **避免循环依赖：** 如果使用构造器注入，Spring在启动时就能检测到循环依赖（A依赖B，B又依赖A）并抛出`BeanCurrentlyInCreationException`，迫使你重构设计，避免潜在的运行时问题。
    *   **与框架无关：** 构造器注入不依赖特定的注解（除了可选的`@Autowired`），是更纯粹的Java代码。
*   **缺点：**
    *   如果依赖项很多，构造函数参数列表会很长，可能影响可读性（这时可能是类职责过多的信号，应考虑重构）。
### 2. Setter 注入（Setter Injection）
*   **原理：** 通过类的**Setter方法**来注入依赖。Spring容器在调用无参构造函数（或工厂方法）创建Bean实例后，再调用相应的Setter方法来设置依赖。
*   **实现方式：**
    *   **XML配置：** 使用`<property>`元素。
    *   **Java注解：** 在Setter方法上使用`@Autowired`。
*   **代码示例：**
    ```java
    public class CustomerService {
        private CustomerRepository customerRepository;
        private EmailService emailService;
    
        // Setter注入（通常方法名为setXxx）
        @Autowired
        public void setCustomerRepository(CustomerRepository customerRepository) {
            this.customerRepository = customerRepository;
        }
    
        @Autowired
        public void setEmailService(EmailService emailService) {
            this.emailService = emailService;
        }
    
        public void registerCustomer(Customer customer) {
            // 使用customerRepository和emailService
        }
    }
    ```
*   **优点：**
    *   **灵活性：** 可以在对象创建后**动态地**改变依赖（虽然在实际应用中很少需要这样做）。适合可选的依赖或配置可能变化的依赖。
    *   **可读性（对于可选依赖）：** 对于非必需的依赖，Setter方法可以更清晰地表示它们是可选的（构造器注入要求所有依赖在创建时提供）。
    *   **解决循环依赖：** Spring可以通过先实例化对象（调用无参构造），再注入依赖的方式处理某些构造器注入无法解决的循环依赖（但这通常是设计有问题的征兆）。
*   **缺点：**
    *   **对象状态可能不完整：** 在Setter方法被调用之前，对象可能处于依赖缺失的状态（部分注入）。如果其他方法在依赖未设置时被调用，会导致`NullPointerException`。
    *   **可变性：** 依赖字段通常不能设为`final`，意味着它们可能在对象生命周期中被改变（除非Setter方法有保护逻辑），这降低了线程安全性和状态稳定性。
    *   **不如构造器注入明确：** 一个类可能有多个Setter方法，哪些依赖是必需的，哪些是可选的，不如构造器注入一目了然。     
### 3. 字段注入（Field Injection）
*   **原理：** 直接在类的**字段（成员变量）** 上使用注解（如`@Autowired`）进行注入。Spring容器通过反射机制直接设置字段的值，不需要构造器或Setter方法。
*   **实现方式：**
    *   **Java注解：** 在字段上直接使用`@Autowired`（最常见）或`@Resource`(JSR-250)。
*   **代码示例：**
    ```java
    public class ProductService {
        @Autowired // 直接注入到字段
        private ProductRepository productRepository;
        @Autowired
        private DiscountCalculator discountCalculator;
    
        public Product getProductById(String id) {
            // 使用productRepository和discountCalculator
        }
    }
    ```
*   **优点：**
    *   **简洁：** 代码量最少，没有样板代码（没有构造器或Setter方法）。看起来非常简洁。
*   **缺点：**
    *   **违反封装性：** 字段通常是`private`的，但注入绕过了正常的访问控制（通过反射直接设置私有字段），破坏了类的封装性。
    *   **不可变性和final：** 由于注入发生在对象构造之后（通过反射），你**不能**将注入的字段声明为`final`（否则编译错误），失去了不可变性带来的好处。
    *   **隐藏依赖：** 依赖关系完全隐藏在类内部。从类的外部看（只看公共API如构造器和方法），无法知道这个类依赖哪些组件。这使得单元测试变得困难（必须使用Spring容器或反射来注入依赖，无法通过构造器或Setter手动注入Mock）。
    *   **与容器强耦合：** 字段注入高度依赖于Spring容器。如果你想在非Spring环境中复用这个类（例如，在普通Java应用中手动`new`一个实例），将无法初始化这些依赖字段，因为它们不会被自动注入。
    *   **难以进行依赖检查：** 对于必需的依赖，如果容器找不到合适的Bean来注入，错误只会在运行时（实际使用该字段时）才暴露（可能抛出`NullPointerException`）。而构造器注入在启动时就能发现缺失依赖。
   
<div class="markdown-table-wrapper"><table><thead><tr><th>特性</th><th>构造器注入</th><th>Setter注入</th><th>字段注入</th></tr></thead><tbody><tr><td><strong>实现方式</strong></td><td>通过构造函数参数</td><td>通过Setter方法</td><td>直接在字段上添加注解</td></tr><tr><td><strong>代码示例</strong></td><td><code>public A(B b) { this.b=b; }</code></td><td><code>public void setB(B b) {...}</code></td><td><code>@Autowired private B b;</code></td></tr><tr><td><strong>不可变性</strong></td><td>✅ (字段可声明final)</td><td>❌</td><td>❌</td></tr><tr><td><strong>对象完整性</strong></td><td>✅ (创建即完整状态)</td><td>❌ (可能处于部分初始化状态)</td><td>❌</td></tr><tr><td><strong>循环依赖检测</strong></td><td>✅ (启动时报错)</td><td>⚠️ (Spring可处理)</td><td>⚠️ (Spring可处理)</td></tr><tr><td><strong>可测试性</strong></td><td>✅ (无需框架即可测试)</td><td>✅</td><td>❌ (需反射或Spring容器)</td></tr><tr><td><strong>Null安全性</strong></td><td>✅ (强制依赖非空)</td><td>❌ (可能漏注入)</td><td>❌</td></tr><tr><td><strong>推荐度</strong></td><td>⭐⭐⭐⭐ (Spring官方推荐)</td><td>⭐⭐⭐</td><td>⭐ (不推荐)</td></tr></tbody></table></div>
## 三、详细解析与代码示例
### 1.构造器注入（Constructor Injection）  
```java
@Component
public class OrderService {
    private final PaymentGateway paymentGateway; // final确保不变性
    private final InventoryService inventoryService;

    // 构造器注入（Spring 4.3+ 可省略@Autowired）
    @Autowired
    public OrderService(PaymentGateway paymentGateway, 
                        InventoryService inventoryService) {
        this.paymentGateway = paymentGateway;
        this.inventoryService = inventoryService;
    }
}
```
*   **优点**
    *   **强制依赖**：确保对象创建时所有必需依赖已就绪
    *   **线程安全**：依赖字段可声明为final
    *   **避免NPE**：启动时立即发现缺失依赖
    *   **清晰可见**：通过构造函数明确依赖关系
*   **缺点**
    *   参数较多时影响可读性（可能是代码坏味道）
### 2.Setter注入（Setter Injection）  
```java

@Component
public class UserService {
    private UserRepository userRepository;
    private EmailService emailService;

    // Setter注入（可选依赖）
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // 另一个Setter
    @Autowired(required = false) // 非必需依赖
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}

```

*   **优点**：
    *   **灵活性**：支持可选依赖（required=false）
    *   **可重新注入**：允许运行时更换实现
    *   **解决循环依赖**：Spring特有解决方案
*   **缺点**：
    *   对象可能处于不完整状态
    *   需手动检查依赖是否初始化
    *   线程安全问题（非final字段）
### 3.字段注入（Field Injection）  
```java
@Component
public class ProductService {
    @Autowired  // 不推荐！
    private ProductRepository productRepository;
    
    @Autowired
    private DiscountCalculator discountCalculator;
}
```
*   **优点**：
    *   代码简洁（无样板代码）
*   **致命缺点**：
    *   **破坏封装性**：反射直接修改私有字段
    *   **不可测试**：无法脱离Spring容器实例化对象
    *   **隐藏依赖**：从类外部无法感知依赖需求
    *   **循环依赖风险**：设计问题被掩盖
    *   **无法声明final**：失去不变性保证
## 四、最佳实践推荐
1.  **首选构造器注入**（Spring官方推荐）
    *   强制依赖
    *   核心业务组件
    *   需要不可变性的场景
    ```java
    @Service
    @RequiredArgsConstructor // Lombok自动生成构造器
    public class OrderProcessor {
        private final PaymentService paymentService;
        private final InventoryService inventoryService;
    }
    ```
2.  **谨慎使用Setter注入**
    *   可选依赖（如`@Autowired(required=false)`）
    *   需要动态更换实现的场景
    *   遗留代码兼容
3.  **避免字段注入**
    *    新项目禁止使用
    *    旧项目逐步重构替换
4.  **特殊场景处理**
    ```java
    // 混合使用示例
    @Service
    public class HybridExample {
        private final MandatoryService mandatory; // 必需依赖
        
        private OptionalService optional; // 可选依赖
        
        @Autowired
        public HybridExample(MandatoryService mandatory) {
            this.mandatory = mandatory;
        }
        
        @Autowired(required = false)
        public void setOptionalService(OptionalService optional) {
            this.optional = optional;
        }
    }
    ```    
## 五、最佳实践建议
| 特性               | 构造器注入 (Constructor Injection)         | Setter注入 (Setter Injection)              | 字段注入 (Field Injection)               |
| :----------------- | :---------------------------------------- | :---------------------------------------- | :--------------------------------------- |
| **代码简洁性**     | 中等 (需要构造函数)                       | 中等 (需要Setter方法)                     | **最高** (直接在字段上加注解)            |
| **可读性/明确性**  | **最高** (必需依赖一目了然)               | 中等 (依赖是否必需需看Setter方法名/注释) | 最低 (依赖关系隐藏)                      |
| **不可变性**       | **支持** (可声明final字段)                | 不支持 (字段通常非final)                 | 不支持 (字段不能是final)                |
| **对象状态完整性** | **最高** (创建后即完整)                   | 较低 (可能部分注入)                      | 较低 (可能部分注入)                     |
| **可测试性**       | **最好** (易于手动传入Mock依赖)           | 好 (可通过Setter传入Mock)                | 差 (需要反射或Spring容器)               |
| **封装性**         | 好 (通过构造函数参数)                     | 好 (通过Setter方法)                      | **差** (直接反射访问私有字段)           |
| **框架耦合度**     | 低 (构造函数是标准Java)                   | 中等 (Setter是标准Java，但注解是Spring)   | **高** (完全依赖注解和反射)            |
| **循环依赖处理**   | 启动时失败 (强制设计改进)                 | 可能解决 (但建议避免循环依赖)            | 可能解决 (但建议避免循环依赖)           |
| **适用场景**       | **强推荐用于必需的、不变的依赖**          | 可选依赖、需要动态重新绑定的依赖          | **不推荐，尽量少用**                   |

## 六、总结
*   **构造器注入**：保证对象完整性和不变性，是Spring官方推荐的首选方式
*   **Setter注入**：适用于可选依赖或需要动态配置的场景
*   **字段注入**：应避免使用，因其破坏封装性、降低可测试性
遵循这些原则可构建更健壮、可测试和可维护的Spring应用。实际项目中，结合Lombok的@RequiredArgsConstructor可让构造器注入更简洁高效。