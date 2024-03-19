## Spring

### 介绍一下 Spring

Spring 是一款顶级开源框架，它是包含了众多工具方法的 IoC 容器。

Spring 包含了很多模块，比如 spring-core、spring-beans、spring-aop、spring-context、spring-expression、spring-test 等，使用 Spring 可以帮我们快速的开发 Java 程序。





### Spring 有什么优点?

Spring 优点如下：

1. 开源免费的热门框架，稳定性高、解决问题成本低；
2. 方便集成各种优秀的框架；
3. 降低了代码耦合性，通过 Spring 提供的 IoC 容器，我们可以将对象之间的依赖关系交由 Spring 进行控制，避免硬编码所造成的过度程序耦合；
4. 方便程序测试，在 Spring 里，测试变得非常简单，例如：Spring 对 Junit 的支持，可以通过注解方便的测试 Spring 程序；
5. 降低 Java EE API 的使用难度，Spring 对很多难用的 Java EE API（如 JDBC、JavaMail、远程调用等）提供了一层封装，通过 Spring 的简易封装，让这些 Java EE API 的使用难度大为降低。



### Spring 使用了哪些设计模式？

1. 代理模式：在 AOP 中有使用；
2. 单例模式：bean 默认是单例模式；
3. 模板方法模式：jdbcTemplate；
4. 工厂模式：BeanFactory；
5. 观察者模式：Spring 事件驱动模型就是观察者模式很经典的一个应用，比如，ContextStartedEvent 就是 ApplicationContext 启动后触发的事件；
6. 适配器模式：Spring MVC 中也是用到了适配器模式适配 Controller。



### Spring 加载过程（ApplicationContext 容器 refresh 过程）

1、读取 xml 并且初始化 BeanFactory

2、加载 BeanDefinition

3、BeanFactory 的 后处理

4、注册 BeanPostProcessor

5、初始化 事件广播器

6、初始化监听器

7、初始化单例 Bean



### BeanFactory 和 FactoryBean 区别

BeanFactory.getBean() 方法中会先判断该 bean 是不是 FactoryBean 对象，如果不是，直接 createBean 然后返回；如果是，createBean 之后需要调用 FactoryBean 中的 getObject 方法返回代理对象。例如 MyBatis 就是实现了 FactoryBean 对象来实现增删改查操作的。

MyBatis 就是实现了一个 MapperFactorybean 类，在 getObject 方法中提供 sqlSession 进行增删改查操作，返回的就是一个 Mapper 接口对象。



这个操作和 MyBatis 中的插件实现方式类似，spring 是在 createBean 之后，判断是不是 FactoryBean 类，然后再调用 getObject 方法生成代理对象；

MyBatis 可以拦截多个类的多个方法，例如拦截 StatemenHandler，在执行查询前 new StatemenHandler 的时候，判断有没有插件，如果有就嵌入插件，嵌入插件的过程就是建立新的反射对象的过程，对于每个拦截器通过实现 Interceptor 接口，重写 Intercept 方法，建立的反射对象的 InvocationHandler 是一个 Plugin 类，这个类里面有 Intercepter 对象，invoke() 方法执行了 intercept 方法。，如果有多个拦截器就循环建立代理对象，然后返回 StatementHandler 的代理对象。



### IOC

#### 如何理解 IOC？

IoC是 “控制反转”，是一种设计思想。它能指导我们如何设计出松耦合、更优良的程序。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器注入对象，所以对象与对象之间是 松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。所有的类都会在spring容器中登记，比如一个类，告诉 spring 你是什么类，需要什么东西，然后spring会把你要的东西主动给你，同时也把你交给其他需要你的类。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。

控制反转是通过DI（Dependency Injection，依赖注入）来实现的，而实现依赖注入是靠反射注入的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。依赖注入的名字就这么来的。



#### IoC 有什么优点？

IoC 的优点有以下几个：

1. 使用更方便，拿来即用，无需显式的创建和销毁的过程；
2. 可以很容易提供众多服务，比如事务管理、消息服务等；
3. 提供了单例模式的支持；
4. 提供了 AOP 抽象，利用它很容易实现权限拦截、运行期监控等功能；
5. 更符合面向对象的设计法则；
6. 低侵入式设计，代码的污染极低，降低了业务对象替换的复杂性。



#### 什么是 DI？

DI 是 Dependency Injection 的缩写，翻译成中文是“依赖注入”的意思。**依赖注入不是一种设计实现，而是一种具体的技术**，它是**在 IoC 容器运行期间，动态地将某个依赖对象注入到当前对象的技术就叫做 DI（依赖注入）**。

比如 A 对象需要依赖 B 对象，那么在 A 运行时，动态的将依赖对象 B 注入到当前类中，而非通过直接 new 的方式获取 B 对象的方式，就是依赖注入。



#### IoC 和 DI 有什么区别？

IoC 和 DI 虽然定义不同，但它们所做的事情都是一样的，都是用来实现对象解耦的，而二者又有所不同：**IoC 是一种设计思想，而 DI 是一种具体的实现技术**。





### AOP

#### 如何理解AOP

系统中会存在很多组件是跟业务无关的，例如日志、事务、权限校验等，这些服务组件经常融入到具体的业务逻辑中，如果我们为每一个具体业务逻辑操作都添加这样的代码，很明显代码冗余太多，因此我们需要将这些公共的代码逻辑抽象出来变成一个切面，然后注入到具体业务中去，AOP正是基于这样的一个思路实现的，通过动态代理的方式，将需要注入切面的对象进行代理，在进行调用的时候，将公共的逻辑直接添加进去，而不需要修改原有业务的逻辑代码，只需要在原来的业务逻辑基础之上做一些增强功能即可。





#### AOP 有什么优点？常见使用场景有哪些？

**AOP 优点：**

1. 集中处理某一类问题，方便维护；
2. 逻辑更加清晰；
3. 降低模块间的耦合度。

**AOP 常见的使用场景：**

1. 用户登录和鉴权
2. 统一日志记录
3. 统一方法执行时间统计
4. 统一的返回格式设置
5. 统一的异常处理
6. 事务的开启和提交等



#### AspectJ 定义的通知类型有哪些？

- **Before**（前置通知）：目标对象的方法调用之前触发
- **After** （后置通知）：目标对象的方法调用之后触发
- **AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发
- **AfterThrowing**（异常通知）：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around** （环绕通知）：编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法





#### AOP 是如何组成的？

AOP 是由：切面（Aspect）、切点（Pointcut）、连接点（Join Point）和通知（Advice）组成的，它们的具体含义如下。

**① 切面（Aspect）**

切面（Aspect）由切点（Pointcut）和通知（Advice）组成，它既包含了横切逻辑的定义，也包括了连接点的定义。

简单来说，切面就是当前 AOP 功能的类型，比如当前 AOP 是用户登录和鉴权的功能，那么它就是一个切面。

**② 切点（Pointcut）**

切点 Pointcut：它的作用就是提供一组规则（使用 AspectJ pointcut expression language 来描述）用来匹配连接点的。

简单来说，切点就是设置拦截规则的，满足规则的方法将会被拦截。

③ **连接点（Join Point）**

应用执行过程中能够插入切面的一个点，这个点可以是方法调用时，抛出异常时，甚至修改字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

简单来说，所有可以触发切点拦截规则的功能都是连接点。比如所有要登录才能访问的控制器（方法），它们都属于连接点。

**④ 通知（Advice）**

切面也是有目标的 ——它必须完成的工作。在 AOP 术语中，切面的工作被称之为通知。

简单来说，当控制器（方法）被拦截之后，触发执行的具体方法就是通知。

小结：切面定义了 AOP 的功能，切点提供了具体的拦截规则，通知决定了具体的执行方法，而连接点就是用来触发 AOP 的这些功能的，它们共同组成了 AOP。



#### Spring AOP 有几种通知类型（Advice）？

Spring AOP 中有 5 种通知类型：

1. **前置通知**使用 @Before 实现：通知方法会在目标方法调用之前执行；
2. **后置通知**使用 @After 实现：通知方法会在目标方法返回或者抛出异常后调用；
3. **返回通知**使用 @AfterReturning 实现：通知方法会在目标方法返回后调用；
4. **抛出异常通知**使用 @AfterThrowing 实现：通知方法会在目标方法抛出异常后调用；
5. **环绕通知**使用 @Around 实现：通知包裹了被通知的方法，在被通知的方法通知之前和调用之后执行自定义的行为。



#### 说一下 Spring AOP 实现原理？

Spring AOP 是在动态代理的基础上实现的，如果我们为 Spring 的某个 bean 配置了切面，那么 Spring 在创建这个 bean 的时候，实际上创建的是这个 bean 的一个代理对象，我们后续对 bean 中方法的调用，实际上调用的是代理类重写的代理方法。

Spring AOP 支持两种动态代理：JDK Proxy 和 CGLIB 动态代理。默认情况下，实现了接口的类，使用 AOP 会基于 JDK 生成代理类，没有实现接口的类，会基于 CGLIB 生成代理类。



### Bean

#### Bean 加载过程（Bean 生命周期）

关于 Bean 的生命周期，可以从 Spring 的 createBean() 方法中体现出来，我结合 createBean 方法来说明一下 Bean 的生命周期。



调用 getBean 加载 Bean：

第一步 判断一二三级缓存中有无，如果有，就取出 

第二步 如果bean声明为FactoryBean类型，则当提取bean时候提取的是FactoryBean中对应的 getObject 方法返回的 bean ,然后调用 postProcessObjectFromFactoryBean方法，然后返回 Bean。（*因为Spring获取bean的规则中有这样一条：尽可能保证所有bean初始化后都会调用 postProcessAfterInitialization 方法进行处理，在实际开发过程中可以针对此特性设计自己的业务处理。*）

第三步，如果缓存中没有，就执行 createBean 流程：

- `createBeanInstance()` 使用反射实例化 bean
- `populateBean()` 属性填充，这里根据注入类型提取属性，并通过反射调用 set 方法赋值，填充属性过程中，涉及到了循环依赖的处理：singletonObjects（成品对象）、earlySingletonObjects（半成品对象）、singletonFactories（工厂对象，放的ObjectFactory 对象，有 getObject 方法）
- `initializeBean()` 初始化 bean：用来执行用户设置的初始化操作，分为三步：

1、设置 Aware 方法，例如 setBeanFactoryAware 

2、后置处理器的应用，BeanPostProcessor在调用用户自定义初始化方法前或者调用自定义初始化方法后分别会调用 postProcessBeforeInitialization 和 postProcessAfterinitialization 方法，使用户可以根据自己的业务需求就行相应的处理。

3、自定义 init 方法，首先检查是否实现了 InitializingBean 接口，如果实现了就先执行 afterPropertiesSet ，然后检查是否指定了 init-method() ，如果指定了就同过反射调用该方法。

（ *我们可以使用 `<beans>` 标签的 `default-init-method` 属性来统一指定初始化方法，这样就省了需要在每个 `<bean>` 标签中都设置 `init-method` 这样的繁琐工作了。比如在 `default-init-method` 规定所有初始化操作全部以 `initBean()` 命名。*）



#### 说一下 Bean 的生命周期？

**Spring 中 Bean 的生命周期是指：Bean 在 Spring（IoC）中从创建到销毁的整个过程**。

![Spring Bean 生命周期](./ssm框架.assets/b5d264565657a5395c2781081a7483e1.jpg)

1. 实例化：为 Bean 分配内存空间；
2. 设置属性：将当前类依赖的 Bean 属性，进行注入和装配；
3. 初始化：
   1. 执行各种通知；
   2. 执行初始化的前置方法；
   3. 执行初始化方法；
   4. 执行初始化的后置方法。
4. 使用 Bean：在程序中使用 Bean 对象；
5. 销毁 Bean：将 Bean 对象进行销毁操作。



#### 三级缓存相关问题

**三级缓存解决循环依赖的完整流程：**

一级：singletonObjects（成品对象）

二级：earlySingletonObjects（半成品对象）

三级：singletonFactories（工厂对象，放的ObjectFactory 对象，有 getObject 方法）

执行getBean(A)：实例化后，提前暴露A，根据 A 有没有加 AOP 代理，创建singletonFactory（如果没有加AOP，这个工厂生产的就是实例化后的Bean对象，如果加了AOP，生产的就是AOP代理对象），并放入三级缓存，接着对 A 填充属性，A 依赖B，需要调用 getBean(B) 

执行 getBean(B)：先从缓存中找是否有B，发现没有，执行 createBean，实例化后，提前暴露B，放入三级缓存，填充属性 A，调用 getBean(A)

执行getBean(A)：发现第三级缓存中有 A 的ObjectFactory 对象，调用 getObject() 方法获取 A 对象或者 A 的代理对象，并且把获取的A对象放进二级缓存，并移除三级缓存的A对象，返回这个A对象。

B 填充 A 属性完成，继续完成 B 的所有初始化操作，然后把 B 放入一级缓存，并删除 二三级缓存的 B，返回成品对象B。

A 获取到成品对象B，完成属性填充、初始化操作，然后把 A 放入一级缓存，并删除二三级缓存中的 A，返回成品对象 A。



**为什么需要三级缓存？一级二级不行吗？**

1、解决循环依赖

2、因为 Spring 设计原则是在 Bean 初始化之后才为其创建代理对象，如果只有二级缓存，那么就需要在 Bean 实例化后立马为其创建代理对象，不符合 Spring 设计原则







#### ApplicationContext 和 BeanFactory 有什么区别？

它们都可以用来获取 Spring 容器，它们的区别如下：

1. 继承关系和功能：Spring 容器有两个顶级的接口：BeanFactory 和 ApplicationContext。其中 BeanFactory 提供了基础的访问容器的能力，而 ApplicationContext 属于 BeanFactory 的子类，它除了继承了 BeanFactory 的所有功能之外，它还拥有独特的特性，还添加了对国际化支持、资源访问支持、以及事件传播等方面的支持。
2. 性能：ApplicationContext 是一次性加载并初始化所有的 Bean 对象，而 BeanFactory 是需要那个才去加载那个，因此更加轻量。



#### Bean 注入有几种方式？

Spring 中对象注入的方法有 3 种：

1. 属性注入（Field Injection） 直接 @Autowired
2. Setter 注入（Setter Injection）在 set 方法上 @Autowired
3. 构造方法注入（Constructor Injection）在构造方法上 @Autowired 

Spring 官方推荐使用构造方法注入，原因如下：

1. 属性注入的优点是简洁，使用方便；缺点是只能用于 IoC 容器，如果是非 IoC 容器不可用，并且只有在使用的时候才会出现 NPE（空指针异常）；
2. **构造方法注入是 Spring 推荐的注入方式**，它的缺点是如果有多个注入会显得比较臃肿，但出现这种情况你应该考虑一下当前类是否符合程序的单一职责的设计模式了，它的优点是通用性，在使用之前一定能把保证注入的类不为空；
3. Setter 方式是 Spring 前期版本推荐的注入方式，但通用性不如构造方法，所有 Spring 现版本已经推荐使用构造方法注入的方式来进行类注入了。



#### 属性注入有什么缺点？

1. 功能性问题：无法注入一个不可变的对象（final 修饰的对象）；
2. 通用性问题：只能适应于 IoC 容器；
3. 设计原则问题：更容易违背单一设计原则。



#### Setter 注入有什么缺点？

1. 不能注入不可变对象（final 修饰的对象）；
2. 注入的对象可被修改。



#### @Autowired 和 @Resource 有什么区别？

1. 来源不同：@Autowired 来自 Spring 框架，而 @Resource 来自于（Java）JSR-250；
2. 依赖查找的顺序不同：@Autowired 先根据类型再根据名称查询，而 @Resource 先根据名称再根据类型查询；
3. 依赖注入的用法支持不同：@Autowired 既支持构造方法注入，又支持属性注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入；
4. 编译器 IDEA 的提示不同：当注入 Mapper 对象时，使用 @Autowired 注解编译器会提示错误，而使用 @Resource 注解则不会提示错误。



#### Bean 是线程安全的吗？

Spring 框架中的 Bean 是否线程安全，取决于其作用域和状态。

我们这里以最常用的两种作用域 prototype 和 singleton 为例介绍。几乎所有场景的 Bean 作用域都是使用默认的 singleton ，重点关注 singleton 作用域即可。

prototype 作用域下，每次获取都会创建一个新的 bean 实例，不存在资源竞争问题，所以不存在线程安全问题。singleton 作用域下，IoC 容器中只有唯一的 bean 实例，可能会存在资源竞争问题（取决于 Bean 是否有状态）。如果这个 bean 是有状态的话，那就存在线程安全问题（有状态 Bean 是指包含可变的成员变量的对象）。

不过，大部分 Bean 实际都是无状态（没有定义可变的成员变量）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。

对于有状态单例 Bean 的线程安全问题，常见的有两种解决办法：

1. 在 Bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）



#### Bean 有几种作用域？

在 Spring 中，Bean 的常见作用域有以下 5 种：

1. singleton：单例作用域；
2. prototype：原型作用域（多例作用域）；
3. request：请求作用域（适用于 Spring MVC ）；
4. session：会话作用域（适用于 Spring MVC ）；
5. application：全局作用域（适用于 Spring MVC ）。

**① singleton**

**Spring 默认选择该作用域**。

该作用域下的 Bean 在 IoC 容器中只存在一个实例：获取 Bean（即通过 applicationContext.getBean等方法获取）及装配 Bean（即通过 @Autowired 注入）都是同一个对象。

**② prototype**

通常**有状态**的 Bean 使用该作用域。

每次对该作用域下的 Bean 的请求都会创建新的实例：获取 Bean（即通过 applicationContext.getBean 等方法获取）及装配 Bean（即通过 @Autowired 注入）都是新的对象实例。

**③ request**

每次 Http 请求会创建新的 Bean 实例，类似于 prototype。

一次 Http 的请求和响应的共享 Bean。

只 Spring MVC 框架中使用。

**④ session**

在一个 Http Session 中，定义一个 Bean 实例。

用户会话的共享 Bean, 比如：记录一个用户的登陆信息。

只 Spring MVC 框架中使用。

**⑤ application**

在一个 Http Servlet Context 中，定义一个 Bean 实例。

Web 应用的上下文信息，比如：记录一个应用的共享信息。

只 Spring MVC 框架中使用。





### Spring 事务

####  Spring 管理事务的方式有几种？

- **编程式事务**：在代码中硬编码(不推荐使用) : 通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
- **声明式事务**：在 XML 配置文件中配置或者直接基于注解（推荐使用） : 实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多。



#### Spring 事务传播行为

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

**1.REQUIRED**

使用的最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

**2.REQUIRES_NEW**

创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，REQUIRES_NEW 修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。

**3.NESTED**

如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于 REQUIRED。

**4.MANDATORY**

如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

这个使用的很少。

若是错误的配置以下 3 种事务传播行为，事务将不会发生回滚：

- **SUPPORTS**: 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **NOT_SUPPORTED**: 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **NEVER**: 以非事务方式运行，如果当前存在事务，则抛出异常。



####  @Transactional(rollbackFor = Exception.class) 注解

当 `@Transactional` 注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在 `@Transactional` 注解中如果不配置`rollbackFor`属性,那么事务只会在遇到`RuntimeException`的时候才会回滚，加上 `rollbackFor=Exception.class`,可以让事务在遇到非运行时异常时也回滚。



#### @Transactional有哪些参数

传播行为 隔离级别 只读事务 超时时间 回滚规则

#### @Transactional失效的情况

1、数据库本身不支持事务 ，例如 MyISAM 引擎是不支持事务操作的，要支持事务都会使用 InnoDB引擎。

2、事务没有被Spring管理

3、同一类中的自身方法调用

4、方法不是Public

5、事物传播性问题

6、数据源未配置事务管理器

7、rollbackFor异常指定错误 例如rollbaclFor指定的是DataFormatException，但程序没有抛DataFormatException，因此事务不会回滚。若没有指定回滚异常，默认的回滚异常是RuntimeException ，如果出现其他异常那么就不会回滚事物。

8、异常被catch住了

代码中手动catch了异常，然后又未抛出来，此时事务就不生效了。