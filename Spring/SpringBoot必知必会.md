### SpringBoot的核心注解组成？
核心注解：@SpringBootApplication它由3个子注解组成：
**@SpringBootConfiguration**：组合了@Configuration注解，实现配置文件的功能
**@EnableAutoConfiguration**：开启自动配置，也可以通过exclude关闭
**@ComponentScan**：配置扫描的包

### SpringBoot自动配置的原理?
自动配置的注解是@EnableAutoConfiguration，从上面的@Import注解可以找到加载自动配置的映射；
SpringFactoriesLoader.loadFactoryNames方法会加载类路径及所有jar包下META-INF/spring.factories配置中映射的自动配置的类；
@Configuration,@ConditionalOnClass就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下是否有这个类去自动配置；

### SpringBoot Starter启动器是什么？
它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成Spring及其他技术，而不需要到处找示例代码和依赖包；
SpringBoot官方的启动器命名都是spring-boot-starter-xxx，第三方一般是xxx-spring-boot-starter；

### 如何在SpringBoot启动的时候运行一些特定的代码？
可以实现接口 ApplicationRunner或者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个run方法；

### SpringBoot有几种读取配置的方式？
SpringBoot可以通过 @PropertySource,@Value,@Environment, @ConfigurationProperties 来绑定变量；

### SpringBoot支持的日志框架有什么？
SpringBoot支持Java Util Logging,Log4j2,Lockback作为日志框架，如果你使用starters启动器，Spring Boot将使用Logback作为默认日志框架；

### spring-boot-maven-plugin有什么用？
它提供了一些像jar一样打包或者运行应用程序的命令

### spring-boot-starter-parent有什么用？
* 定义了Java编译版本为1.8
* 使用UTF-8格式编码
* 继承自spring-boor-dependencies，这里面定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号
* 执行打包操作的配置
* 自动化的资源过滤
* 自动化的插件配置
* 针对application.peoperties和application.yuml的资源过滤，包括通过profile定义的不同环境的配置文件，例如application-dev.properties和application-dev.yml

### SpringBoot打的jar包和普通的jar包有什么区别？
Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过java -jar xxx.jar命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。
> Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。
>
> 在configuration中配置classifier标识可执行的jar；下例会生成两个jar包，xxx.jar和xxx-exec.jar
>
> ```
> <plugin>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-maven-plugin</artifactId>
>     <version>2.5.5</version>
>     <configuration>
>         <classifier>exec</classifier>
>     </configuration>
> </plugin>
> ```

### 运行SpringBoot的几种方式？
* 打包用命令或者放到容器中运行
* 用Maven或Gradle插件运行
* 直接执行main方法运行

### SpringBoot的核心配置文件有哪些？
SpringBoot的核心配置文件是application和bootstrap配置文件。
application配置文件这个容易理解，主要用于Spring Boot项目的自动化配置。
bootstrap配置文件有以下几个应用场景：

* 使用Spring Cloud Config配置中心时，这时需要在bootstrap配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
* 一些固定的不能被覆盖的属性；
* 一些加密/解密的场景。

| 配置文件    |                                                              |                        |
| ----------- | ------------------------------------------------------------ | ---------------------- |
| application | 用于配置应用程序上下文<br />主要用于配置应用程序的自动化配置，比如日志、数据库、Web等 | 属性可以被其他方式覆盖 |
| bootstrap   | 用于配置引导上下文（应用程序上下文的父上下文）<br />优先级高于application<br />主要用于从外部源加载配置属性，比如Spring Cloud Config配置中心<br />还可以在本地外部配置文件中解密属性 | 属性不能被覆盖         |

