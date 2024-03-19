

## SpringBoot



### SpringBoot 自动加载原理

SpringBoot的主配置类上标记 **@SpringBootApplication** 注解，它是一个复合注解，主要是有 @EnableAutoConfiguration 注解，用来通知 SpringBoot 开启自动配置功能，这样自动配置才能生效。它的原理是有一个 **@Import(EnableAutoConfigurationImportSelector.class)** 注解，这个类的作用就是SpringBoot启动的时候从 classpath 下的 **META-INF/spring.factories**中获取EnableAutoConfiguration指定的值，并将这些值作为自动配置类导入到容器中，自动配置类就会生效，最后完成自动配置工作。

（或者念下面的）：

Spring Boot 自动装配执行流程如下：

1. Spring Boot 启动时会创建一个 SpringApplication 实例，该实例存储了应用相关信息，它负责启动并运行应用。
2. 实例化 SpringApplication 时，会自动装载 META-INF/spring.factories 中配置的自动装配类。
3. SpringApplication 实例调用 run() 方法启动应用。
4. 在 run() 方法中，实例会创建默认的应用上下文 Environment 以及 ApplicationContext。
5. SpringApplication 会通过 ListableBeanFactory 加载应用上下文 ApplicationContext 中的所有 BeanDefinition。
6. 在 BeanDefinition 加载过程中，SpringApplication 会检测是否存在基于 @Conditional 条件装配注解的自动装配类。
7. 如果存在且 @Conditional 条件校验成功，则会装配这些自动装配类。
8. 这些自动装配类通过 @EnableAutoConfiguration、@Configuration 等注解，装配默认的 Spring Bean。
9. 装配完成后，Spring Boot 将启动应用，这里会启动嵌入的 Web 服务器，如 Tomcat 并发布 Web 应用。
10. 发布完成，Spring Boot 应用启动成功。



### 启动流程分析

1、获取并启动监听器 

2、加载配置，读取Spring.factory

3、启动 Spring 容器，创建 ApplicationContext

4、容器前置处理，**将启动类注入容器**

5、刷新容器（refresh）

> 1、读取 xml 并且初始化 BeanFactory
>
> 2、加载 BeanDefinition
>
> 3、BeanFactory 的 后处理
>
> 4、注册 BeanPostProcessor
>
> 5、初始化 事件广播器
>
> 6、初始化监听器
>
> 7、初始化单例 Bean

6、执行 Runner 的 run 方法

















