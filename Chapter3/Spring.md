# [Spring 基础](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-基础)

## [1. 什么是 Spring 框架?](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#什么是-spring-框架)

Spring 是一款开源的轻量级 Java 开发框架，支持 IoC（Inversion of Control:控制反转） 和 AOP(Aspect-Oriented Programming:面向切面编程)、可以很方便地对数据库进行访问、可以很方便地集成第三方组件（电子邮件，任务，调度，缓存等等）、对单元测试支持比较好、支持 RESTful Java 应用程序的开发。

## [2. Spring 包含的模块有哪些？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-包含的模块有哪些)

[Core Container](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#core-container)

- **spring-core**：Spring 框架基本的核心工具类。
- **spring-beans**：提供对 bean 的创建、配置和管理等功能的支持。
- **spring-context**：提供对国际化、事件传播、资源加载等功能的支持。
- **spring-expression**：提供对表达式语言（Spring Expression Language） SpEL 的支持，只依赖于 core 模块，不依赖于其他模块，可以单独使用。

[AOP](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#aop)

[Data Access/Integration](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#data-access-integration)

[Spring Web](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-web)

[Spring Test](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-test)

## [3. Spring,Spring MVC,Spring Boot 之间什么关系?](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-spring-mvc-spring-boot-之间什么关系)

Spring MVC 是 Spring 中的一个很重要的模块，基于 Spring 的 IoC 容器，主要赋予 Spring 快速构建 MVC 架构的 Web 程序的能力。MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

Spring Boot 旨在简化 Spring 开发（减少配置文件，开箱即用！）。

## [4. Spring 框架中用到了哪些设计模式？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-框架中用到了哪些设计模式)

- **工厂设计模式** : Spring 使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。

# [Spring IoC](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-ioc)

## [1. 谈谈自己对于 Spring IoC 的了解](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#谈谈自己对于-spring-ioc-的了解)

IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。

**为什么叫控制反转？**

- **控制**：指的是对象创建（实例化、管理）的权力
- **反转**：控制权交给外部环境（Spring 框架、IoC 容器）

在实际项目中一个 Service 类可能依赖了很多其他的类，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

## [2. IoC 和 DI 有区别吗？](https://javaguide.cn/system-design/framework/spring/ioc-and-aop.html#ioc-和-di-有区别吗)

IoC 最常见以及最合理的实现方式叫做依赖注入（Dependency Injection，简称 DI）。

## [2. 什么是 Spring Bean？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#什么是-spring-bean)

Bean 代指的就是那些被 IoC 容器所管理的对象。

## [3. 将一个类声明为 Bean 的注解有哪些?](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#将一个类声明为-bean-的注解有哪些)

- `@Component`：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。

## [4. @Component 和 @Bean 的区别是什么？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#component-和-bean-的区别是什么)

- `@Component` 注解作用于类，而`@Bean`注解作用于方法。
- `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

下面这个例子是通过 `@Component` 无法实现的。

~~~Java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
~~~

## [5. 注入 Bean 的注解有哪些？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#注入-bean-的注解有哪些)

Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以用于注入 Bean。

## [6. @Autowired 和 @Resource 的区别是什么？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#autowired-和-resource-的区别是什么)

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配，类型不行的话按名称），`@Resource`默认注入方式为 `byName`（根据名称进行匹配，名称不行按类型）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 `name` 属性来显式指定名称。
- `@Autowired` 支持在构造函数、方法、字段和参数上使用。`@Resource` 主要用于字段和方法上的注入，不支持在构造函数或参数上使用。

~~~java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Autowired
private SmsService smsServiceImpl1;
// 正确注入  SmsServiceImpl1 对象对应的 bean
// smsServiceImpl1 就是我们上面所说的名称
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;

----
 // 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;
// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
~~~

## [7. 注入 Bean 的方式有哪些（依赖注入）？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#注入-bean-的方式有哪些)

构造函数注入示例：

~~~java
@Service
public class UserService {
		//依赖项
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
~~~

Setter 注入示例：

~~~java
@Service
public class UserService {

    private UserRepository userRepository;

    // 在 Spring 4.3 及以后的版本，特定情况下 @Autowired 可以省略不写
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
~~~

Field 注入示例：

~~~java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    //...
}
~~~

## [8. 构造函数注入还是 Setter 注入？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#构造函数注入还是-setter-注入)

**Spring 官方推荐构造函数注入**，这种注入方式的优势如下：

1. 依赖完整性：确保所有必需依赖在对象创建时就被注入，避免了空指针异常的风险。
2. 不可变性：有助于创建不可变对象，提高了线程安全性。

构造函数注入适合处理**必需的依赖项**，而 **Setter 注入** 则更适合**可选的依赖项**，这些依赖项可以有默认值或在对象生命周期中动态设置。虽然 `@Autowired` 可以用于 Setter 方法来处理必需的依赖项，但构造函数注入仍然是更好的选择。

## [9. Bean 的作用域有哪些?如何配置？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#bean-的作用域有哪些)

- **singleton** : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean **默认都是单例的**，是对单例设计模式的应用。
- **prototype** : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例。
- **request** （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
- **session** （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
- **application/global-session** （仅 Web 应用可用）：每个 Web 应用在启动时创建一个 Bean（应用 Bean），该 bean 仅在当前应用启动时间内有效。
- **websocket** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。

xml方式

~~~xml
<bean id="..." class="..." scope="singleton"></bean>
~~~

注解方式：

~~~java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
~~~

## [10. Bean 是线程安全的吗？如果不安全如何解决？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#bean-是线程安全的吗)

singleton 作用域下，IoC 容器中只有唯一的 bean 实例，可能会存在资源竞争问题（取决于 Bean 是否有状态）。如果这个 bean 是有状态的话，那就存在线程安全问题（**有状态 Bean 是指包含可变的成员变量的对象**）。

~~~java
// 定义了一个购物车类，其中包含一个保存用户的购物车里商品的 List
@Component
public class ShoppingCart {
    private List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }
}
~~~

不过，大部分 Bean 实际都是无状态（**没有定义可变的成员变量**）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。

prototype 作用域下，每次获取都会创建一个新的 bean 实例，不存在资源竞争问题，所以不存在线程安全问题。



对于有状态单例 Bean 的线程安全问题，常见的三种解决办法是：

1. **避免可变成员变量**: 尽量设计 Bean 为无状态。
2. **使用`ThreadLocal`**: 将可变成员变量保存在 `ThreadLocal` 中，确保线程独立。
3. **使用同步机制**: 利用 `synchronized` 或 `ReentrantLock` 来进行同步控制，确保线程安全。

## [11. Bean 的生命周期了解么?](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#bean-的生命周期了解么)

1. **创建 Bean 的实例**：Bean 容器首先会找到配置文件中的 Bean 定义，然后使用 Java 反射 API 来创建 Bean 的实例。

2. **Bean 属性赋值/填充**：为 Bean 设置相关属性和依赖，例如`@Autowired` 等注解注入的对象、`@Value` 注入的值、`setter`方法或构造函数注入依赖和值、`@Resource`注入的各种资源。

3. Bean 初始化

   - 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
   - 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
   - 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。
   - 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
   - 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
   - 如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
   - 如果 Bean 在配置文件中的定义包含 `init-method` 属性，执行指定的方法。
   - 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法。

4. 销毁 Bean

   ：销毁并不是说要立马把 Bean 给销毁掉，而是把 Bean 的销毁方法先记录下来，将来需要销毁 Bean 或者销毁容器的时候，就调用这些方法去释放 Bean 所持有的资源。

   - 如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
   - 如果 Bean 在配置文件中的定义包含 `destroy-method` 属性，执行指定的 Bean 销毁方法。或者，也可以直接通过`@PreDestroy` 注解标记 Bean 销毁之前执行的方法。

   

1. 整体上可以简单分为四步：实例化 —> 属性赋值 —> 初始化 —> 销毁。
2. 初始化这一步涉及到的步骤比较多，包含 `Aware` 接口的依赖注入、`BeanPostProcessor` 在初始化前后的处理以及 `InitializingBean` 和 `init-method` 的初始化操作。
3. 销毁这一步会注册相关销毁回调接口，最后通过`DisposableBean` 和 `destory-method` 进行销毁。

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501312124824.png)

# [Spring AOP](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-aop)

## [1. 谈谈自己对于 AOP 的了解](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#谈谈自己对于-aop-的了解)

AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP 就是基于动态代理的。

| 术语              |                             含义                             |
| :---------------- | :----------------------------------------------------------: |
| 目标(Target)      |                         被通知的对象                         |
| 代理(Proxy)       |             向目标对象应用通知之后创建的代理对象             |
| 连接点(JoinPoint) |         目标对象的所属类中，定义的所有方法均为连接点         |
| 切入点(Pointcut)  | 被切面拦截 / 增强的连接点（**切入点一定是连接点，连接点不一定是切入点**）。切点可以通过注解、正则表达式、逻辑运算等方式来定义 |
| 通知(Advice)      | 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情  |
| 切面(Aspect)      | 对横切关注点进行封装的类，一个切面是一个类，等于切入点(Pointcut)+通知(Advice) |
| Weaving(织入)     |       将通知应用到目标对象，进而生成代理对象的过程动作       |

~~~java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    // @Before 通知，在目标方法执行前执行
    @Before("execution(* com.example.demo.service.*.*(..))")
    public void beforeAdvice(JoinPoint joinPoint) {
        System.out.println("Before method: " + joinPoint.getSignature().getName());
    }

    // @After 通知，无论目标方法是否抛出异常都会执行
    @After("execution(* com.example.demo.service.*.*(..))")
    public void afterAdvice(JoinPoint joinPoint) {
        System.out.println("After method: " + joinPoint.getSignature().getName());
    }

    // @AfterReturning 通知，只有目标方法正常返回时才会执行
    @AfterReturning(pointcut = "execution(* com.example.demo.service.*.*(..))", returning = "result")
    public void afterReturningAdvice(JoinPoint joinPoint, Object result) {
        System.out.println("After returning method: " + joinPoint.getSignature().getName() + ", Result: " + result);
    }
}
~~~



## [2. Spring AOP 和 AspectJ AOP 有什么区别？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-aop-和-aspectj-aop-有什么区别)

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** 

Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

## [3. AOP 常见的通知类型有哪些？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#aop-常见的通知类型有哪些)

- **Before**（前置通知）：目标对象的方法调用之前触发
- **After** （后置通知）：目标对象的方法调用之后触发
- **AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发
- **AfterThrowing**（异常通知）：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around** （环绕通知）：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法

- **`@After`**：不管连接点（目标方法）是正常返回还是抛出异常，它都会在目标方法执行之后执行。可以把它理解成一个 “最终通知”，类似于 Java 中 `try-catch-finally` 结构里的 `finally` 块，无论目标方法的执行结果如何，`@After` 标注的通知方法都会被调用。
- **`@AfterReturning`**：只有在目标方法正常返回（没有抛出异常）时才会执行。如果目标方法执行过程中抛出了异常，那么 `@AfterReturning` 标注的通知方法不会被调用。

![image-20250131213334591](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501312133624.png)

## [4. 多个切面的执行顺序如何控制？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#多个切面的执行顺序如何控制)

通常使用`@Order` 注解直接定义切面顺序。

实现`Ordered` 接口重写 `getOrder` 方法。

# [Spring MVC](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-mvc)

## [1. 说说自己对于 Spring MVC 了解?](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#说说自己对于-spring-mvc-了解)

**MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。**

## [2. Spring MVC 的核心组件有哪些？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-mvc-的核心组件有哪些)

- **`DispatcherServlet`**：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应。
- **`HandlerMapping`**：**处理器映射器**，根据 URL 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
- **`HandlerAdapter`**：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`；
- **`Handler`**：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`**：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端

## [3. SpringMVC 工作原理了解吗?](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#springmvc-工作原理了解吗)

1. 客户端（浏览器）发送请求， `DispatcherServlet`拦截请求。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping` 。`HandlerMapping` 根据 URL 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
3. `DispatcherServlet` 调用 `HandlerAdapter`适配器执行 `Handler` 。
4. `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`，`ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
6. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7. 把 `View` 返回给请求者（浏览器）

![img](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501312152693.png)

上述流程是传统开发模式（JSP，Thymeleaf 等）的工作原理。

对于前后端分离时，后端通常不再返回具体的视图，而是返回**纯数据**（通常是 JSON 格式，Spring 会自动将其转换为 JSON 格式），由前端负责渲染和展示。

## [4. 统一异常处理怎么做？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#统一异常处理怎么做)

见javaguide。

# [Spring 的循环依赖](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-的循环依赖)

## [1. Spring 循环依赖了解吗，怎么解决？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-循环依赖了解吗-怎么解决)

两个或多个 Bean 之间相互持有对方的引用。

~~~java
@Component
public class CircularDependencyA {
    @Autowired
    private CircularDependencyB circB;
}

@Component
public class CircularDependencyB {
    @Autowired
    private CircularDependencyA circA;
}
~~~

有三级缓存：

1. **一级缓存（singletonObjects）**：存放最终形态的 Bean（已经实例化、属性填充、初始化），单例池，为“Spring 的单例属性”⽽⽣。一般情况我们获取 Bean 都是从这里获取的，但是并不是所有的 Bean 都在单例池里面，例如原型 Bean 就不在里面。
2. **二级缓存（earlySingletonObjects）**：存放过渡 Bean（半成品，尚未属性填充，未进行依赖注入），也就是三级缓存中`ObjectFactory`产生的对象，与三级缓存配合使用的，可以防止 AOP 的情况下，每次调用`ObjectFactory#getObject()`都是会产生新的代理对象的。
3. **三级缓存（singletonFactories）**：存放`ObjectFactory`，`ObjectFactory`的`getObject()`方法（最终调用的是`getEarlyBeanReference()`方法）可以生成原始 Bean 对象或者代理对象（如果 Bean 被 AOP 切面代理）。三级缓存只会对单例 Bean 生效。



1. 先去 **一级缓存 `singletonObjects`** 中获取，存在就返回；
2. 如果不存在或者对象正在创建中，于是去 **二级缓存 `earlySingletonObjects`** 中获取；
3. 如果还没有获取到，就去 **三级缓存 `singletonFactories`** 中获取，通过执行 `ObjectFacotry` 的 `getObject()` 就可以获取该对象，获取成功之后，**从三级缓存移除B**，并将B对象加入到二级缓存中。



步骤 1：创建 Bean A

1. Spring 容器开始创建 Bean A，首先将 Bean A 的创建状态标记为 “正在创建”。
2. 实例化 Bean A，将 Bean A 的早期引用（一个 `ObjectFactory` 对象）放入三级缓存 `singletonFactories` 中。这个 `ObjectFactory` 对象可以在需要时返回 Bean A 的早期实例。
3. 开始对 Bean A 进行属性注入，发现 Bean A 依赖于 Bean B。

步骤 2：创建 Bean B

1. Spring 容器开始创建 Bean B，同样将 Bean B 的创建状态标记为 “正在创建”。
2. 实例化 Bean B，将 Bean B 的早期引用放入三级缓存 `singletonFactories` 中。
3. 开始对 Bean B 进行属性注入，发现 Bean B 依赖于 Bean A。

步骤 3：解决 Bean B 对 Bean A 的依赖

1. 当 Bean B 需要注入 Bean A 时，Spring 容器首先从一级缓存 `singletonObjects` 中查找 Bean A，发现没有找到。
2. 接着从二级缓存 `earlySingletonObjects` 中查找，也没有找到。
3. 然后从三级缓存 `singletonFactories` 中查找，找到了 Bean A 的早期引用（`ObjectFactory` 对象）。
4. 调用 `ObjectFactory` 的 `getObject()` 方法，获取 Bean A 的早期实例。如果 Bean A 需要进行 AOP 代理，会在这里创建代理对象，并将代理对象放入二级缓存 `earlySingletonObjects` 中，同时从三级缓存 `singletonFactories` 中移除该引用。
5. 将获取到的 Bean A 的早期实例注入到 Bean B 中。

步骤 4：完成 Bean B 的创建

1. Bean B 完成属性注入后，进行初始化操作。
2. 将完全创建好的 Bean B 实例放入一级缓存 `singletonObjects` 中，并从二级缓存 `earlySingletonObjects` 和三级缓存 `singletonFactories` 中移除相关引用。

步骤 5：完成 Bean A 的创建

1. 由于 Bean B 已经创建完成，将 Bean B 实例注入到 Bean A 中。
2. Bean A 完成属性注入后，进行初始化操作。
3. 将完全创建好的 Bean A 实例放入一级缓存 `singletonObjects` 中，并从二级缓存 `earlySingletonObjects` 和三级缓存 `singletonFactories` 中移除相关引用。

## [2. @Lazy 能解决循环依赖吗？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#lazy-能解决循环依赖吗)

如果一个 Bean 没有被标记为懒加载，那么它会在 Spring IoC 容器启动的过程中被创建和初始化。

如果一个 Bean 被标记为懒加载，那么它不会在 Spring IoC 容器启动时立即实例化，而是在第一次被请求时才创建。这可以帮助减少应用启动时的初始化时间，也可以用来解决循环依赖问题。

- **没有 `@Lazy` 的情况下**：在 Spring 容器初始化 `A` 时会立即尝试创建 `B`，而在创建 `B` 的过程中又会尝试创建 `A`，最终导致循环依赖（即无限递归，最终抛出异常）。
- **使用 `@Lazy` 的情况下**：Spring 不会立即创建 `B`，而是会注入一个 `B` 的代理对象。由于此时 `B` 仍未被真正初始化，`A` 的初始化可以顺利完成。等到 `A` 实例实际调用 `B` 的方法时，代理对象才会触发 `B` 的真正初始化。

`@Lazy` 能够在一定程度上打破循环依赖链，允许 Spring 容器顺利地完成 Bean 的创建和注入。但这并不是一个根本性的解决方案，尤其是在构造函数注入、复杂的多级依赖等场景中。

## [3. SpringBoot 允许循环依赖发生么？](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#springboot-允许循环依赖发生么)

允许，但是不推荐。

# [Spring 事务](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#spring-事务)

# SpringBoot

## 1. 什么是 Spring Boot Starters?

Spring Boot Starters 是一组便捷的依赖描述符，它们预先打包了常用的库和配置。当我们开发 Spring 应用时，只需添加一个 Starter 依赖项，即可自动引入所有必要的库和配置，快速引入相关功能。

在没有 Spring Boot Starters 之前，开发一个 RESTful 服务或 Web 应用程序通常需要手动添加多个依赖，比如 Spring MVC、Tomcat、Jackson 等。这不仅繁琐，还容易导致版本不兼容的问题。而有了 Spring Boot Starters，我们只需添加一个依赖，如 spring-boot-starter-web，即可包含所有开发 REST 服务所需的库和依赖。

这个 spring-boot-starter-web 依赖包含了 Spring MVC（用于处理 Web 请求）、Tomcat（默认嵌入式服务器）、Jackson（用于 JSON 处理）等依赖项。这种方式极大地简化了开发过程，让我们可以更加专注于业务逻辑的实现。

## 2. 介绍一下@SpringBootApplication 注解

- 
  @EnableAutoConfiguration: 启用 Spring Boot 的自动配置机制。它是自动配置的核心，允许 Spring Boot 根据项目的依赖和配置自动配置 Spring 应用的各个部分。
- @ComponentScan: 启用组件扫描，扫描被 @Component（以及 @Service、@Controller 等）注解的类，并将这些类注册为 Spring 容器中的 Bean。默认情况下，它会扫描该类所在包及其子包下的所有类。
- @Configuration: 允许在上下文中注册额外的 Bean 或导入其他配置类。它相当于一个具有 @Bean 方法的 Spring 配置类。

## 3. Spring Boot 的自动配置是如何实现的?

Spring Boot 通过`@EnableAutoConfiguration`开启自动装配，通过 SpringFactoriesLoader 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配.

## 4. 开发 RESTful Web 服务常用的注解有哪些？

- @Autowired : 自动导入对象到类中，被注入进的类同样要被 Spring 容器管理。
- @RestController : @RestController注解是@Controller和@ResponseBody的合集,表示这是个控制器 bean,并且是将函数的返回值直 接填入 HTTP 响应体中,是 REST 风格的控制器。RestController一般把 JSON 数据写入http的reponse中，而Controller返回的是页面。
- @Component ：通用的注解，可标注任意类为 Spring 组件。如果一个 Bean 不知道属于哪个层，可以使用@Component 注解标注。
- @Repository : 对应持久层即 Dao 层，主要用于数据库相关操作。
- @Service : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- @Controller : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。
- @GetMapping : GET 请求
- @PostMapping : POST 请求
- @PutMapping : PUT 请求
- @DeleteMapping : DELETE 请求
- @RequestParam：用于获取查询参数
- @Pathvariable：用于获取路径参数
- @RequestBody：接收到数据之后会自动将数据绑定到 Java 对象上去

## 5.Spirng Boot 常用的两种配置文件，如何读取？

我们可以通过 application.properties或者 application.yml 对 Spring Boot 程序进行简单的配置。

通过 @value 读取比较简单的配置信息

~~~java
@Value("${wuhan2020}")
String wuhan2020;
~~~

通过@ConfigurationProperties读取并与 bean 绑定
