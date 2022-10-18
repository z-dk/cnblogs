## SpringBean的生命周期

![](https://www.zdk.world/wp-content/uploads/2022/10/springBean.svg)

SpringBean的生命周期大体如下:

1. Instantiation:实例化bean(完成构造器（依赖）注入)
2. 依赖注入:属性(接口)注入,setter注入
3. aware:beanName,beanFactory,applicationContext
4. Initialization:初始化
5. destroy

### Spring扩展点

#### ApplicationContextInitializer

这是整个spring容器在刷新之前初始化`ConfigurableApplicationContext`的回调接口，简单来说，就是在容器刷新之前调用此类的`initialize`方法。这个点允许被扩展。可以在整个spring容器还没被初始化之前做一些事情。

使用场景:在最开始激活一些配置，或者利用这时候class还没被类加载器加载的时机，进行动态字节码注入等操作。


因为这时候spring容器还没被初始化，所以想要自己的扩展的生效，有以下三种方式：<br/>
* 1.在启动类中用springApplication.addInitializers(new TestApplicationContextInitializer())语句加入<br/>
* 2.配置文件配置context.initializer.classes=com.example.demo.TestApplicationContextInitializer<br/>
* 3.Spring SPI扩展，在spring.factories中加入org.springframework.context.ApplicationContextInitializer=com.zdk.hello.spring.HelloWorldInitializer

#### BeanDefinitionRegistryPostProcessor

在读取项目中的`beanDefinition`之后执行，提供一个补充的扩展点;

其继承了**BeanFactoryPostProcessor**,可以通过此方法获取所有bean definition,并修改已注册bean的元信息,bean此时尚未实例化

使用场景：可以在这里动态注册自己的`beanDefinition`，可以加载classpath之外的bean

#### **InstantiationAwareBeanPostProcessor**

继承了BeanPostProcessor

**`BeanPostProcess`接口只在bean的初始化阶段进行扩展（注入spring上下文前后），而`InstantiationAwareBeanPostProcessor`接口在此基础上增加了3个方法，把可扩展的范围增加了实例化阶段和属性注入阶段。**

该类主要的扩展点有以下5个方法，主要在bean生命周期的两大阶段：**「实例化阶段」** 和**「初始化阶段」** ，按调用顺序为：

- `postProcessBeforeInstantiation`：实例化bean之前，相当于new这个bean之前
- `postProcessAfterInstantiation`：实例化bean之后，相当于new这个bean之后
- `postProcessProperties`：bean已经实例化完成，在属性注入时阶段触发，`@Autowired`,`@Resource`等注解原理基于此方法实现
- `postProcessBeforeInitialization`：初始化bean之前，相当于把bean注入spring上下文之前
- `postProcessAfterInitialization`：初始化bean之后，相当于把bean注入spring上下文之后

使用场景：这个扩展点非常有用 ，无论是写中间件和业务中，都能利用这个特性。比如对实现了某一类接口的bean在各个生命期间进行收集，或者对某个类型的bean进行统一的设值等等。

#### **SmartInstantiationAwareBeanPostProcessor**

该扩展接口有3个触发点方法：

- `predictBeanType`：该触发点发生在`postProcessBeforeInstantiation`之前(在图上并没有标明，因为一般不太需要扩展这个点)，这个方法用于预测Bean的类型，返回第一个预测成功的Class类型，如果不能预测返回null；当你调用`BeanFactory.getType(name)`时当通过bean的名字无法得到bean类型信息时就调用该回调方法来决定类型信息。
- `determineCandidateConstructors`：该触发点发生在`postProcessBeforeInstantiation`之后，用于确定该bean的构造函数之用，返回的是该bean的所有构造函数列表。用户可以扩展这个点，来自定义选择相应的构造器来实例化这个bean。
- `getEarlyBeanReference`：该触发点发生在`postProcessAfterInstantiation`之后，当有循环依赖的场景，当bean实例化好之后，为了防止有循环依赖，会提前暴露回调方法，用于bean实例化的后置处理。这个方法就是在提前暴露的回调方法中触发。

#### Aware

| 接口名                         | 描述                                | 触发点                                                       | 备注                                                         |
| ------------------------------ | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BeanNameAware                  | 获取beanName                        | bean的初始化之前，也就是`postProcessBeforeInitialization`之前 | 在初始化bean之前拿到spring容器中注册的的beanName             |
| BeanFactoryAware               | 获取`BeanFactory`对象               | bean的实例化之后，注入属性之前，也就是Setter之前             | 可以在bean实例化之后，但还未初始化之前，拿到 `BeanFactory`，在这个时候，可以对每个bean作特殊化的定制。也或者可以把`BeanFactory`拿到进行缓存，之后使用 |
| ApplicationContextAware        | 获取`ApplicationContext`的扩展类    | bean实例化之后，初始化之前                                   | 获取任何在spring上下文注册的bean                             |
| EnvironmentAware               | 获取`EnviromentAware`对象           | bean实例化之后，初始化之前                                   | 可以获取系统内所有参数,也可以通过注入的方式直接获取          |
| EmbeddedValueResolverAware     | 获取`StringValueResolver`对象       | bean实例化之后，初始化之前                                   | 与@Value注解功能一样,用于获取配置属性                        |
| ResourceLoaderAware            | 获取`ResourceLoader`对象            | bean实例化之后，初始化之前                                   | 可以获取classpath内所有的资源对                              |
| ApplicationEventPublisherAware | 获取`ApplicationEventPublisher`对象 | bean实例化之后，初始化之前                                   | 可以用来发布事件，结合`ApplicationListener`来共同使用;也可以通过注入的方式获取 |
| MessageSourceAware             | 获取`MessageSource`扩展类对象       | bean实例化之后，初始化之前                                   | 主要用来做国际化                                             |
|                                |                                     |                                                              |                                                              |

#### **@PostConstruct**

JSR250规范中的注解,触发点是在`postProcessBeforeInitialization`之后，`InitializingBean.afterPropertiesSet`之前

是bean初始化方法,与init-method作用一致

#### **InitializingBean**

也是用来初始化bean的。`InitializingBean`接口为bean提供了初始化方法的方式，它只包括`afterPropertiesSet`方法，凡是继承该接口的类，在初始化bean的时候都会执行该方法。这个扩展点的触发时机在`postProcessAfterInitialization`之前。

使用场景：用户实现此接口，来进行系统启动的时候一些业务指标的初始化工作。

#### **FactoryBean**

一般情况下，Spring通过反射机制利用bean的class属性去实例化bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在bean中提供大量的配置信息。配置方式的灵活性是受限的，这时提供了一种采用编码的方式来实例化bean。Spring为此提供了一个`org.springframework.bean.factory.FactoryBean`的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。

该工厂bean会实例化两个bean对象:

- 工厂内定义的bean(getObject方法返回的):beanName为工厂类名首字母小写(factoryBean)
- 工厂本身:可以通过按类型获取该bean或者按name(&factoryBean)获取

使用场景：用户可以扩展这个类，来为要实例化的bean作一个代理，可参考`ProxyFactoryBean`的功能。

#### **SmartInitializingSingleton**

这个接口中只有一个方法`afterSingletonsInstantiated`，其作用是是在spring容器管理的所有单例对象（非懒加载对象）初始化完成之后调用的回调接口。其触发时机为`postProcessAfterInitialization`之后。

使用场景：可以扩展此接口在对所有单例对象初始化完毕后，做一些后置的业务处理。

#### **CommandLineRunner**

这个接口也只有一个方法：`run(String... args)`，触发时机为整个项目启动完毕后，自动执行。如果有多个`CommandLineRunner`，可以利用`@Order`来进行排序。

使用场景：用户扩展此接口，进行启动项目之后一些业务的预处理。

#### **DisposableBean**

这个扩展点也只有一个方法：`destroy()`，其触发时机为当此对象销毁时，会自动执行这个方法。比如说运行`applicationContext.registerShutdownHook`时，就会触发这个方法。

#### **ApplicationListener**

`ApplicationListener`可以监听某个事件的`event`，触发时机可以穿插在业务方法执行过程中，用户可以自定义某个业务事件。spring内部也有一些内置事件，这种事件，可以穿插在启动中。可以利用这个特性，来自己做一些内置事件的监听器来达到和前面一些触发点大致相同的事情。

接下来罗列下spring主要的内置事件：

- ContextRefreshedEvent

  ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在`ConfigurableApplicationContext`接口中使用 `refresh()`方法来发生。此处的初始化是指：所有的Bean被成功装载，后处理Bean被检测并激活，所有Singleton Bean 被预实例化，`ApplicationContext`容器已就绪可用。

- ContextStartedEvent

  当使用 `ConfigurableApplicationContext` （ApplicationContext子接口）接口中的 start() 方法启动 `ApplicationContext`时，该事件被发布。可以在监听到到这个事件后重启任何停止的应用程序。

- ContextStoppedEvent

  当使用 `ConfigurableApplicationContext`接口中的 `stop()`停止`ApplicationContext` 时，发布这个事件。可以在监听到这个事件后做必要的清理的工作

- ContextClosedEvent

  当使用 `ConfigurableApplicationContext`接口中的 `close()`方法关闭 `ApplicationContext` 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启

- RequestHandledEvent

  这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。只能应用于使用DispatcherServlet的Web应用。在使用Spring作为前端的MVC控制器时，当Spring处理用户请求结束后，系统会自动触发该事件


### FAQ

#### BeanFactory与ApplicationContext的区别?

- BeanFactory是SpringIOC容器所定义的最底层的接口
- ApplicationContext是BeanFactory高级子接口之一,且在其基础之上做了更多的扩展

#### 如何设置或修改Bean名称?

- @Component,@Service,@Controller等注解指定
- @Bean注解指定

#### 依赖注入的各个方式都发生在(bean生命周期)哪个阶段?

- 构造器注入:instantiation实例化阶段
- setter注入:postProcessProperties属性设置阶段
- 属性(接口)注入:postProcessProperties属性设置阶段