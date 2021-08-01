# SringBoot特性

本节深入介绍了Spring Boot的细节。在这里你可以了解到你可能想要使用和定制的关键功能。如果你还没有这样做，你可能需要阅读[开始使用](/spring-boot/getting-started.html) 和 [使用Spring Boot](/spring-boot/using.html) 部分，以便你对基础知识有一个良好的了解。

## 1. SpringApplication

`SpringApplication`类提供了一种方便的方式来引导一个从`main()`方法启动的Spring应用程序。在许多情况下，你可以委托给静态的`SpringApplication.run`方法，如以下例子所示。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

当你的应用程序启动时，你应该看到类似于以下的输出。

```text
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.5.3

2021-02-03 10:33:25.224  INFO 17321 --- [           main] o.s.b.d.s.s.SpringAppplicationExample    : Starting SpringAppplicationExample using Java 1.8.0_232 on mycomputer with PID 17321 (/apps/myjar.jar started by pwebb)
2021-02-03 10:33:25.226  INFO 17900 --- [           main] o.s.b.d.s.s.SpringAppplicationExample    : No active profile set, falling back to default profiles: default
2021-02-03 10:33:26.046  INFO 17321 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-02-03 10:33:26.054  INFO 17900 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-02-03 10:33:26.055  INFO 17900 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.41]
2021-02-03 10:33:26.097  INFO 17900 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-02-03 10:33:26.097  INFO 17900 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 821 ms
2021-02-03 10:33:26.144  INFO 17900 --- [           main] s.tomcat.SampleTomcatApplication         : ServletContext initialized
2021-02-03 10:33:26.376  INFO 17900 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-02-03 10:33:26.384  INFO 17900 --- [           main] o.s.b.d.s.s.SpringAppplicationExample    : Started SampleTomcatApplication in 1.514 seconds (JVM running for 1.823)
```

默认情况下，显示`INFO`日志信息，包括一些相关的启动细节，例如启动应用程序的用户。如果你需要一个除`INFO`以外的日志级别，你可以设置它，如[日志级别](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging.log-levels)中所述。应用程序的版本是通过主应用程序类的包的实现版本来确定的。启动信息的记录可以通过设置`spring.main.log-startup-info`为`false`来关闭。这也将关闭应用程序的活动配置文件的日志记录。

> 为了在启动过程中增加额外的日志记录，你可以在`SpringApplication`的子类中覆盖`logStartupInfo(boolean)`。

### 1.1. 启动失败

如果你的应用程序启动失败，注册的`FailureAnalyzers`有机会提供专门的错误信息和具体的行动来解决这个问题。例如，如果你在端口`8080`上启动一个网络应用程序，而该端口已经被使用，你应该看到类似于下面的信息。

```text
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

> Spring Boot提供了许多 "FailureAnalyzers" 的实现，你也可以[添加你自己的](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.application.failure-analyzer)。

如果没有故障分析器能够处理异常，你仍然可以显示完整的条件报告，以更好地了解出错的原因。为此，你需要为`org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` [启用debug属性](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)或[启用DEBUG日志记录](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging.log-levels)。

例如，如果你通过使用`java -jar`运行你的应用程序，你可以启用`debug`属性，如下所示。

```bash
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 1.2. 延迟初始化

`SpringApplication`允许应用程序被延迟初始化。当启用延迟初始化时，Bean在需要时被创建，而不是在应用程序启动时。因此，启用延迟初始化可以减少应用程序的启动时间。在一个Web应用程序中，启用延迟初始化将导致许多与Web相关的Bean在收到HTTP请求之前不会被初始化。

延迟初始化的一个缺点是，它可以延迟发现应用程序的问题。如果一个配置错误的Bean被初始化，在启动过程中就不会再出现故障，问题只有在Bean被初始化时才会显现。还必须注意确保JVM有足够的内存来容纳应用程序的所有Bean，而不仅仅是那些在启动时被初始化的Bean。由于这些原因，默认情况下不启用延迟初始化，建议在启用延迟初始化之前，对JVM的堆大小进行微调。

可以使用`SpringApplicationBuilder`上的`lazyInitialization`方法或`SpringApplication`上的`setLazyInitialization`方法以编程方式启用延迟初始化。另外，也可以使用`spring.main.lazy-initialization`属性来启用，如下例所示。

```yaml
spring:
  main:
    lazy-initialization: true
```

如果你想禁用某些Bean的延迟初始化，同时对应用程序的其他部分使用延迟初始化，你可以使用`@Lazy(false)`注解将它们的懒惰属性明确地设置为false。

### 1.3. 自定义Banner

启动时打印的Banner可以通过在classpath中添加`banner.txt`文件或将`spring.banner.location`属性设置为此类文件的位置来改变。如果该文件的编码不是UTF-8，你可以设置`spring.banner.charset`。除了文本文件，你还可以在classpath中添加`banner.gif`, `banner.jpg`, 或`banner.png`图像文件，或者设置`spring.banner.image.location`属性。图像被转换为ASCII艺术表现，并打印在任何文本横幅之上。

在你的`banner.txt`文件中，你可以使用以下任何一个占位符。

表 1. Banner 中可以使用的变量

|Variable|Description|
| --- | --- |
|`${application.version}`|你的应用程序的版本号，正如在`MANIFEST.MF`中声明的那样。例如，`Implementation-Version: 1.0`被打印为`1.0`。|
|`${application.formatted-version}`|你的应用程序的版本号，如在`MANIFEST.MF`中声明的那样，并以格式化的方式显示（用括号包围并以`v`为前缀）。例如 `(v1.0)` 。|
|`${spring-boot.version}`|你正在使用的Spring Boot版本。例如`2.5.3`。|
|`${spring-boot.formatted-version}`|您正在使用的Spring Boot版本，并以格式化显示（用括号包围，前缀为`v`）。例如，`(v2.5.3)` 。|
|`${Ansi.NAME}` (or `${AnsiColor.NAME}` , `${AnsiBackground.NAME}` , `${AnsiStyle.NAME}` )|其中`NAME`是一个ANSI转义代码的名称。详见[`AnsiPropertySource`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。|
|`${application.title}`|你的应用程序的标题，正如在`MANIFEST.MF`中声明的那样。例如，`Implementation-Title: MyApp`被打印成 `MyApp`。|

> 如果你想以编程方式生成一个Banner，可以使用`SpringApplication.setBanner(..)`方法。使用`org.springframework.boot.Banner`接口并实现你自己的`printBanner()`方法。

你也可以使用`spring.main.banner-mode`属性来决定是否将banner打印到`System.out`（`console`），发送到配置的logger（`log`），或根本不产生（`off`）。

打印的Banner被注册为一个单例Bean，名字是：`springBootBanner`。

> `${application.version}`和`${application.formatted-version}`属性只有在使用Spring Boot启动器时才可用。如果你运行一个未打包的jar并使用`java -cp <classpath> <mainclass>`启动它，这些值将不会被解析。
>
> 这就是为什么我们建议你总是使用`java org.springframework.boot.loader.JarLauncher`来启动未打包的jar。这将在构建classpath和启动你的应用程序之前初始化`application.*`的Banner变量。

### 1.4. 定制SpringApplication

如果 `SpringApplication` 的默认值不符合你的口味，你可以创建一个本地实例并对其进行自定义。例如，要关闭Banner，你可以这样写。

```java
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
    }

}
```

> 传递给`SpringApplication`的构造参数是Spring Bean的配置源。在大多数情况下，这些是对`@Configuration`类的引用，但它们也可能是对`@Component`类的直接引用。

也可以通过使用 `application.properties` 文件来配置`SpringApplication`。参见[外部化配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)以了解详情。

关于配置选项的完整列表，请参见[SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/SpringApplication.html)。

### 1.5. Fluent Builder API

如果你需要建立一个`ApplicationContext`层次结构（具有父/子关系的多个Context），或者你喜欢使用一个 `Fluent` 构建器API，你可以使用`SpringApplicationBuilder`。

`SpringApplicationBuilder`允许你将多个方法调用串联起来，并包括`parent`和`child`方法，让你创建一个层次结构，如以下例子所示。

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);


```

> 在创建 `ApplicationContext` 层次结构时，有一些限制。例如，Web组件**必须**包含在子context，并且父Context和子Context使用相同的`Environment`。请参阅[SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/builder/SpringApplicationBuilder.html)以了解全部细节。

### 1.6. 应用程序的可用性

在平台上部署时，应用程序可以使用[Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)等基础设施向平台提供有关其可用性的信息。Spring Boot包括对常用的 "liveness"和 "readiness"可用性状态的开箱即用支持。如果你使用Spring Boot的`actuator`支持，那么这些状态将作为健康端点组暴露出来。

此外，你也可以通过将 `ApplicationAvailability` 接口注入到你自己的Bean中来获得可用性状态。

#### 1.6.1. Liveness State

一个应用程序的 "Liveness"状态告诉我们它的内部状态是否允许它正常工作，或者在当前失败的情况下自行恢复。一个破碎的 "Liveness"状态意味着应用程序处于无法恢复的状态，基础设施应该重新启动该应用程序。

> 一般来说，"Liveness" 状态不应该基于外部检查，比如[健康检查](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health)。如果是这样，一个失败的外部系统（数据库、Web API、外部缓存）将引发大规模的重启和整个平台的级联故障。

Spring Boot应用程序的内部状态大多由Spring `ApplicationContext`表示。如果应用程序Context已经成功启动，Spring Boot就认为应用程序处于有效状态。一旦Context被刷新，应用程序就被认为是活的，见[Spring Boot应用程序生命周期和相关应用程序事件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-events-and-listeners)。

#### 1.6.2. Readiness State

一个应用程序的 "Readiness"状态告诉人们该应用程序是否准备好处理流量。一个失败的 "Readiness"状态告诉平台，它暂时不应该把流量发送到该应用程序。这通常发生在启动过程中，当`CommandLineRunner`和`ApplicationRunner`组件被处理时，或者在任何时候，如果应用程序决定它太忙了，无法处理额外的流量。

一旦应用程序和命令行运行器被调用，就认为应用程序已经准备好了，见[Spring Boot应用程序生命周期和相关的应用程序事件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-events-and-listeners)。

#### 1.6.3. 管理应用程序的可用性状态

应用组件可以在任何时候通过注入`ApplicationAvailability`接口并调用其上的方法来检索当前的可用性状态。更多时候，应用程序会想要监听状态更新或更新应用程序的状态。

例如，我们可以把应用程序的 "Readiness"状态导出到一个文件，这样Kubernetes的 "exec Probe"就可以查看这个文件了。

```java
import org.springframework.boot.availability.AvailabilityChangeEvent;
import org.springframework.boot.availability.ReadinessState;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class MyReadinessStateExporter {

    @EventListener
    public void onStateChange(AvailabilityChangeEvent<ReadinessState> event) {
        switch (event.getState()) {
        case ACCEPTING_TRAFFIC:
            // create file /tmp/healthy
            break;
        case REFUSING_TRAFFIC:
            // remove file /tmp/healthy
            break;
        }
    }

}
```

当应用程序中断而无法恢复时，我们还可以更新应用程序的状态。

```java
import org.springframework.boot.availability.AvailabilityChangeEvent;
import org.springframework.boot.availability.LivenessState;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Component;

@Component
public class MyLocalCacheVerifier {

    private final ApplicationEventPublisher eventPublisher;

    public MyLocalCacheVerifier(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void checkLocalCache() {
        try {
            // ...
        }
        catch (CacheCompletelyBrokenException ex) {
            AvailabilityChangeEvent.publish(this.eventPublisher, ex, LivenessState.BROKEN);
        }
    }

}
```

Spring Boot提供了[Kubernetes HTTP探测 "Liveness"和 "Readiness"与Actuator Health Endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.kubernetes-probes)。你可以得到更多关于[在Kubernetes上部署Spring Boot应用程序的专门章节](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.cloud.kubernetes)的指导。

### 1.7. Application Events and Listeners

除了常见的Spring框架事件，如[`ContextRefreshedEvent`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，一个`SpringApplication`会发送一些额外的应用事件。

有些事件实际上是在`ApplicationContext`被创建之前被触发的，所以你不能以`@Bean`的形式注册一个监听器。你可以通过`SpringApplication.addListeners(...)`方法或`SpringApplicationBuilder.listeners(...)`方法注册它们。

如果你希望这些监听器被自动注册，无论应用程序是以何种方式创建的，你可以在项目中添加一个`META-INF/spring.plants`文件，并通过使用`org.springframework.context.ApplicationListener`键来引用你的监听器，如以下例子所示。

```properties
org.springframework.context.ApplicationListener=com.example.project.MyListener
```

当你的应用程序运行时，应用程序事件按以下顺序发送：

1. `ApplicationStartingEvent`在运行开始时被发送，但在任何处理之前，除了监听器和初始化器的注册之外。
2. 当已知将在Context中使用的环境，但在创建Context之前，将发送一个 `ApplicationEnvironmentPreparedEvent`。
3. 当 `ApplicationContext` 被准备好并且 `ApplicationContextInitializers` 被调用，但在任何 bean 定义被加载之前，`ApplicationContextInitializedEvent` 被发送。
4. `ApplicationPreparedEvent`在刷新开始前但在Bean定义被加载后被发送。
5. `ApplicationStartedEvent`在Context被刷新之后，但在任何应用程序和命令行运行程序被调用之前被发送。
6. 紧接着发送一个`AvailabilityChangeEvent`，并注明`LivenessState.CORRECT`，以表明应用程序被认为是有效的。
7. 在任何[application and command-line runners](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.command-line-runner)被调用后，将发送一个 `ApplicationReadyEvent`。
8. 紧接着发送一个带有`ReadinessState.ACCEPTING_TRAFFIC`的`AvailabilityChangeEvent`，以表明应用程序已经准备好为请求提供服务。
9. 如果在启动时出现异常，将发送一个`ApplicationFailedEvent`。

以上列表仅包括与 `SpringApplication` 相关的 `SpringApplicationEvent`。除此以外，以下事件也会在`ApplicationPreparedEvent`之后和`ApplicationStartedEvent`之前发布。

* `WebServerInitializedEvent`是在`WebServer`准备好后发送的。`ServletWebServerInitializedEvent`和`ReactiveWebServerInitializedEvent`分别是Servlet和Reactive的变体。
* `ContextRefreshedEvent`在`ApplicationContext`被刷新时发送。

你通常不需要使用应用程序事件，但知道它们的存在会很方便。在内部，Spring Boot使用事件来处理各种任务。

> Event listeners不应该运行潜在的冗长任务，因为它们默认是在同一个线程中执行。考虑使用[application and command-line runners](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.command-line-runner)来代替。

Application event是通过使用Spring框架的事件发布机制来发送的。该机制的一部分确保了发布给子Context中的listener的事件也会发布给任何祖先Context中的listener。因此，如果你的应用程序使用`SpringApplication`实例的层次结构，一个监听器可能会收到同一类型Application event的多个实例。

为了让你的监听器区分其Context的事件和后代Context的事件，它应该请求其应用程序Context被注入，然后将注入的Context与事件的Context进行比较。Context可以通过实现`ApplicationContextAware`来注入，或者，如果listener是一个Bean，可以通过使用`@Autowired`。

### 1.8. Web 环境

`SpringApplication`试图帮你你创建正确类型的`ApplicationContext`。用于确定`WebApplicationType`的算法如下。

* 如果Spring MVC存在，就会使用`AnnotationConfigServletWebServerApplicationContext`。
* 如果Spring MVC不存在而Spring WebFlux存在，则使用 `AnnotationConfigReactiveWebServerApplicationContext`。
* 否则，将使用 `AnnotationConfigApplicationContext`。

这意味着，如果你在同一个应用程序中使用Spring MVC和Spring WebFlux的新`WebClient`，将默认使用Spring MVC。你可以通过调用`setWebApplicationType(WebApplicationType)`轻松覆盖。

也可以通过调用`setApplicationContextClass(..)`来完全控制使用的`ApplicationContext`类型。

> 当在JUnit测试中使用`SpringApplication`时，通常需要调用`setWebApplicationType(WebApplicationType.NONE)`。

### 1.9. 访问 Application 参数

如果你需要访问传递给`SpringApplication.run(..)`的应用程序参数，你可以注入一个`org.springframework.boot.ApplicationArguments`bean。`ApplicationArguments`接口提供了对原始`String[]`参数以及经过解析的`option`和`non-option`参数的访问，如以下例子所示。

```java
import java.util.List;

import org.springframework.boot.ApplicationArguments;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        if (debug) {
            System.out.println(files);
        }
        // if run with "--debug logfile.txt" prints ["logfile.txt"]
    }

}
```

Spring Boot还在Spring的`Environment` 中注册了一个`CommandLinePropertySource`。这让你也可以通过使用`@Value`注解来注入单个应用参数。

### 1.10. 使用 ApplicationRunner 或者 CommandLineRunner

如果你需要在`SpringApplication`启动后运行一些特定的代码，你可以实现`ApplicationRunner`或`CommandLineRunner`接口。这两个接口以相同的方式工作，并提供一个单一的`run`方法，该方法在`SpringApplication.run(...)`完成之前被调用。

> 这个约定很适合那些应该在应用程序启动后但在其开始接受访问之前运行的任务。

`CommandLineRunner`接口提供对应用程序参数的访问，作为一个字符串数组，而`ApplicationRunner`使用前面讨论的`ApplicationArguments`接口。下面的例子显示了一个带有`run`方法的`CommandLineRunner`。

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // Do something...
    }

}
```

如果定义了几个 `CommandLineRunner` 或 `ApplicationRunner` Bean，它们必须以特定的顺序被调用，你可以额外实现`org.springframework.core.Ordered`接口或使用`org.springframework.core.annotation.Order`注解。

### 1.11. 退出程序

每个 `SpringApplication` 都向JVM注册了一个shutdown hook，以确保 `ApplicationContext` 在退出时优雅地关闭。所有标准的Spring生命周期回调（如`DisposableBean`接口或`@PreDestroy`注解）都可以使用。

此外，如果Bean希望在调用`SpringApplication.exit()`时返回特定的退出代码，可以实现`org.springframework.boot.ExitCodeGenerator`接口。然后，这个退出代码可以被传递给`System.exit()`，将其作为状态代码返回，如下面的例子所示。

```java
import org.springframework.boot.ExitCodeGenerator;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class MyApplication {

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> 42;
    }

    public static void main(String[] args) {
        System.exit(SpringApplication.exit(SpringApplication.run(MyApplication.class, args)));
    }

}
```

另外，`ExitCodeGenerator`接口可以由异常实现。当遇到这种异常时，Spring Boot会返回由实现的`getExitCode()`方法提供的退出代码。

### 1.12. 管理功能

可以通过指定`spring.application.admin.enabled`属性来启用应用程序的管理相关功能。这暴露了平台上的[`SpringApplicationAdminMXBean`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java) `MBeanServer`。你可以使用这个功能来远程管理你的Spring Boot应用程序。这个功能对任何服务封装器的实现也很有用。

> 如果你想知道应用程序是在哪个HTTP端口上运行的，可以读取KEY为`local.server.port`的属性值。

### 1.13. 应用程序启动跟踪

在应用启动期间，`SpringApplication`和`ApplicationContext`执行许多与应用生命周期、Bean生命周期甚至处理应用事件有关的任务。通过[`ApplicationStartup`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/core/metrics/ApplicationStartup.html)，Spring框架[允许你用StartupStep 对象跟踪应用程序的启动顺序](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/core.html#context-functionality-startup)。这些数据可以为分析目的而收集，或者只是为了更好地了解应用程序的启动过程。

你可以在设置 `SpringApplication` 实例时选择一个 `ApplicationStartup` 实现。例如，为了使用`BufferingApplicationStartup`，你可以写。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.metrics.buffering.BufferingApplicationStartup;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setApplicationStartup(new BufferingApplicationStartup(2048));
        application.run(args);
    }

}
```

第一个可用的实现，`FlightRecorderApplicationStartup`是由Spring框架提供的。它将Spring特有的启动事件添加到Java Flight Recorder会话中，旨在对应用程序进行分析，并将其Spring上下文生命周期与JVM事件（如分配、GC、类加载......）联系起来。一旦配置好，你就可以通过启用Flight Recorder运行应用程序来记录数据。

```shell
$ java -XX:StartFlightRecording:filename=recording.jfr,duration=10s -jar demo.jar
```

Spring Boot提供了 `BufferingApplicationStartup` 变体；这个实现是为了缓冲启动步骤，并将其排入外部度量系统。应用程序可以在任何组件中要求获得`BufferingApplicationStartup`类型的bean。

Spring Boot也可以被配置为公开一个[startup端点](https://docs.spring.io/spring-boot/docs/2.5.3/actuator-api/htmlsingle/#startup)，以JSON文档的形式提供这一信息。

## 2. 外部化配置

Spring Boot让你将配置外部化，这样你就可以在不同的环境中使用相同的应用程序代码。你可以使用各种外部配置源，包括Java properties文件、YAML文件、环境变量和命令行参数。

属性值可以通过使用`@Value`注解直接注入你的Bean，通过Spring的`Environment`抽象访问，或者通过`@ConfigurationProperties`[绑定到结构化对象](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties)。

Spring Boot使用一个非常特殊的`PropertySource`顺序，旨在允许合理地覆盖值。属性是按以下顺序考虑的（较低项目的值会覆盖较早项目的值）。

1. 默认属性（通过设置`SpringApplication.setDefaultProperties`指定）。
2. `@Configuration`类上的[@PropertySource](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/annotation/PropertySource.html)注解。请注意，这样的属性源直到application context被刷新时才会被添加到环境中。这对于配置某些属性来说已经太晚了，比如`logging.*`和`spring.main.*`，它们在刷新开始前就已经被读取了。
3. 配置数据（如`application.properties`文件）。
4. 一个`RandomValuePropertySource`，它的属性只有`random.*`。
5. 操作系统环境变量。
6. Java系统属性（`System.getProperties()`）。
7. 来自`java:comp/env`的JNDI属性。
8. `ServletContext`初始参数。
9. `ServletConfig`初始参数。
10. 来自`SPRING_APPLICATION_JSON`的属性（嵌入环境变量或系统属性中的内联JSON）。
11. 命令行参数
12. `properties`属性在你的`tests`上。在[`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/test/context/SpringBootTest.html)和[用于测试你的应用程序的特定片断的测试注释](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-tests)上可用。
13. [`@TestPropertySource`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/test/context/TestPropertySource.html)对你的测试进行注解。
14. 当devtools处于活动状态时，在`$HOME/.config/spring-boot`目录下的[Devtools全局设置属性](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.globalsettings)。

配置数据文件按以下顺序考虑。

1. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files)打包在你的jar里面(`application.properties`和YAML变体)。
2. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)打包在你的jar中（`application-{profile}.properties`和YAML变量）。
3. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files) 在你打包的jar之外(`application.properties`和YAML变体)。
4. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)，在你打包的jar之外（`application-{profile}.properties`和YAML变体）。

> 建议你在整个应用程序中坚持使用一种格式。如果你在同一地点有`.properties`和`.yml`格式的配置文件，`.properties`优先。

为了提供一个具体的例子，假设你开发了一个`@Component`，使用了`name`属性，如下例所示。

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在你的应用程序的classpath（例如，在你的jar中），你可以有一个`application.properties`文件，为`name`提供一个合理的默认属性值。当在一个新的环境中运行时，可以在jar之外提供一个`application.properties`文件，覆盖`name`。对于一次性的测试，你可以用一个特定的命令行开关来启动（例如，`java -jar app.jar --name="Spring"`）。

> `env`和`configprops`端点在确定一个属性为什么有一个特定的值时很有用。你可以使用这两个端点来诊断意外的属性值。详见"[Production ready features](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints) "部分。

### 2.1.  访问命令行属性

默认情况下，`SpringApplication`会将任何命令行选项参数（即以`--`开头的参数，如`--server.port=9000`）转换为`property`并将其添加到Spring`Environment`中。如前所述，命令行属性总是优先于基于文件的属性源。

如果你不希望命令行属性被添加到 `Environment` 中，你可以通过使用`SpringApplication.setAddCommandLineProperties(false)`来禁用它们。

### 2.2. JSON Application Properties

环境变量和系统属性往往有限制，这意味着有些属性名称不能使用。为了帮助解决这个问题，Spring Boot允许你将一个属性块编码为一个单一的JSON结构。

当你的应用程序启动时，任何`spring.application.json`或`SPRING_APPLICATION_JSON`属性将被解析并添加到`Environment`中。

例如，`SPRING_APPLICATION_JSON`属性可以在UN*X shell的命令行中作为环境变量提供。

```shell
$ SPRING_APPLICATION_JSON='{"my":{"name":"test"}}' java -jar myapp.jar
```

在前面的例子中，你在Spring的`Environment`中最终得到了`my.name=test`。

同样的JSON也可以作为一个系统属性提供。

```shell
$ java -Dspring.application.json='{"my":{"name":"test"}}' -jar myapp.jar
```

或者你可以通过使用一个命令行参数来提供JSON。

```shell
$ java -jar myapp.jar --spring.application.json='{"my":{"name":"test"}}'
```

如果你要部署到一个经典的应用服务器，你也可以使用一个名为`java:comp/env/spring.application.json`的JNDI变量。

> 尽管JSON中的 `null` 值将被添加到生成的属性源中，但`PropertySourcesPropertyResolver`将 "`null` 属性视为缺失值。这意味着JSON不能用 `null` 值覆盖来自低阶属性源的属性。

### 2.3. 外部应用属性

当你的应用程序启动时，Spring Boot会自动从以下位置找到并加载`application.properties`和`application.yaml`文件。

1. classpath
    1. classpath的根路径
    2. classpath `/config`包
2. 当前目录
    1. 当前目录根路径
    2. 当前目录下的`/config`子目录
    3. `/config`子目录的直接子目录。

列表按优先级排序（较低项目的值覆盖较早项目的值）。加载的文件被作为 `PropertySources` 添加到Spring的 `Environment` 中。

如果你不喜欢`application`作为配置文件的名称，你可以通过指定`spring.config.name`环境属性来切换到另一个文件名。例如，为了寻找`myproject.properties`和`myproject.yaml`文件，你可以按以下方式运行你的应用程序。

```shell
$ java -jar myproject.jar --spring.config.name=myproject
```

你也可以通过使用`spring.config.location`环境属性来引用一个明确的位置。该属性接受一个逗号分隔的列表，其中包含一个或多个要检查的位置。

下面的例子显示了如何指定两个不同的文件。

```shell
$ java -jar myproject.jar --spring.config.location=\
    optional:classpath:/default.properties,\
    optional:classpath:/override.properties
```

> 如果[位置是可选的](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.optional-prefix)并且你不介意它们不存在，请使用前缀`optional:`。

`spring.config.name` , `spring.config.location` , 和`spring.config.extra-location`很早就用来确定哪些文件必须被加载。它们必须被定义为环境属性（通常是操作系统环境变量，系统属性，或命令行参数）。

如果`spring.config.location`包含目录（而不是文件），它们应该以`/`结尾。在运行时，它们将被附加上由`spring.config.name`生成的名称，然后被加载。在`spring.config.location`中指定的文件被直接导入。

> 目录和文件位置值也都被扩展，以检查[配置文件特定文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)。例如，如果你的`spring.config.location`是`classpath:myconfig.properties`，你也会发现适当的`classpath:myconfig-<profile>.properties`文件被加载。

在大多数情况下，你添加的每个`spring.config.location`项目将引用一个文件或目录。位置是按照定义的顺序来处理的，后面的位置可以覆盖前面的位置的值。

如果你有一个复杂的位置设置，并且你使用特定的配置文件，你可能需要提供进一步的提示，以便Spring Boot知道它们应该如何分组。一个位置组是一个位置的集合，这些位置都被认为是在同一级别。例如，你可能想把所有classpath位置分组，然后是所有外部位置。一个位置组内的项目应该用`;`分开。更多细节见 [配置文件特定文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific) 一节中的例子。

通过使用`spring.config.location`配置的位置将取代默认位置。例如，如果`spring.config.location`的配置值为`optional:classpath:/custom-config/,optional:file:./custom-config/`，则完整位置集为。

1. `optional:classpath:custom-config/`
2. `optional:file:./custom-config/`

如果你喜欢添加额外的位置，而不是替换它们，你可以使用`spring.config.extra-location`。从附加位置加载的属性可以覆盖默认位置的属性。例如，如果`spring.config.extra-location`的配置值为`optional:classpath:/custom-config/,optional:file:./custom-config/`，那么完整位置集是。

1. `optional:classpath:/;optional:classpath:/config/`
2. `optional:file:./;optional:file:./config/;optional:file:./config/*/`
3. `optional:classpath:custom-config/`
4. `optional:file:./custom-config/`

这种搜索排序让你在一个配置文件中指定默认值，然后在另一个文件中选择性地覆盖这些值。你可以在`application.properties`（或你用`spring.config.name`选择的任何其他basename）中为你的应用程序提供默认值，在默认位置之一。然后，这些默认值可以在运行时被位于其中一个自定义位置的不同文件所覆盖。

如果你使用环境变量而不是系统属性，大多数操作系统不允许使用句点分隔的键名，但你可以使用下划线代替（例如，`SPRING_CONFIG_NAME`代替`spring.config.name`）。详见[从环境变量绑定](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)。

如果你的应用程序在servlet容器或应用服务器中运行，那么JNDI属性（在`java:comp/env`中）或servlet上下文初始化参数可以代替环境变量或系统属性，或者与之一样。

#### 2.3.1. 可选路径

默认情况下，当指定的配置数据位置不存在时，Spring Boot将抛出一个`ConfigDataLocationNotFoundException`，你的应用程序将无法启动。

如果你想指定一个位置，但你不介意它并不总是存在，你可以使用`optional:`前缀。你可以在`spring.config.location`和`spring.config.extra-location`属性中使用这个前缀，也可以在[`spring.config.import`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.importing)声明中使用。

例如，`spring.config.import`值为`optional:file:./myconfig.properties`允许你的应用程序启动，即使`myconfig.properties`文件丢失。

如果你想忽略所有的`ConfigDataLocationNotFoundExceptions`并总是继续启动你的应用程序，你可以使用`spring.config.onnot-found`属性。使用`SpringApplication.setDefaultProperties(..)`或使用系统/环境变量将该值设置为`ignore`。

#### 2.3.2. 通配符路径

如果一个配置文件的位置在最后一个路径段中包括`*`字符，它就被认为是一个通配符位置。通配符在配置被加载时被扩展，因此，直接的子目录也被检查。通配符位置在Kubernetes这样的环境中特别有用，因为有多个来源的配置属性。

例如，如果你有一些Redis配置和一些MySQL配置，你可能想把这两块配置分开，同时要求这两块都存在于一个`application.properties`文件中。这可能会导致两个独立的`application.properties`文件挂载在不同的位置，如`/config/redis/application.properties`和`/config/mysql/application.properties`。在这种情况下，有一个`config/*/`的通配符位置，将导致两个文件都被处理。

默认情况下，Spring Boot将`config/*/`列入默认搜索位置。这意味着你的jar之外的`/config`目录的所有子目录都会被搜索到。

你可以通过`spring.config.location`和`spring.config.extra-location`属性自己使用通配符位置。

通配符位置必须只包含一个 `*`，并以 `*/`结尾，用于搜索属于目录的位置，或 `*/<filename>`用于搜索属于文件的位置。带有通配符的位置将根据文件名的绝对路径按字母顺序排序。

> 通配符位置只对外部目录起作用。你不能在`classpath:`位置中使用通配符。

#### 2.3.3. Profile Specific Files

除了`application`属性文件，Spring Boot还将尝试使用命名惯例`application-{profile}`加载profile特定文件。例如，如果你的应用程序激活了名为`prod`的配置文件并使用YAML文件，那么`application.yml`和`application-prod.yml`都将被考虑。

特定于配置文件的属性与标准的`application.properties`的位置相同，特定于配置文件的文件总是优先于非特定文件。如果指定了几个配置文件，则采用最后胜出的策略。例如，如果配置文件`prod,live`是由`spring.profiles.active`属性指定的，`application-prod.properties`中的值可以被`application-live.properties`中的值覆盖。

最后获胜的策略适用于[location group](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.location-groups)级别。`spring.config.location`为`classpath:/cfg/,classpath:/ext/`将不会有与`classpath:/cfg/;classpath:/ext/`一样的覆盖规则。

例如，继续我们上面的 `prod,live` 例子，我们可能有以下文件。

```text
/cfg
  application-live.properties
/ext
  application-live.properties
  application-prod.properties
```

当我们的`spring.config.location`为`classpath:/cfg/,classpath:/ext/`时，我们会在所有`/ext`文件之前处理所有`cfg`文件。

1. `/cfg/application-live.properties`。
2. `/ext/application-prod.properties`。
3. `/ext/application-live.properties`。

当我们用`classpath:/cfg/;classpath:/ext/`代替时（用`;`分隔符），我们在同一级别处理`/cfg`和`/ext`。

1. `/ext/application-prod.properties`。
2. `/cfg/application-live.properties`。
3. `/ext/application-live.properties`。

`Environment`有一组默认的配置文件（默认为[default]），如果没有设置活动的配置文件，就会使用这些配置文件。换句话说，如果没有明确激活的配置文件，那么就会考虑来自`application-default`的属性。

> 属性文件只被加载一次。如果你已经直接[导入](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.importing)一个配置文件的特定属性文件，那么它将不会被第二次导入。

#### 2.3.4. 导入额外的数据

应用程序属性可以使用`spring.config.import`属性从其他位置导入进一步的配置数据。导入在被发现时被处理，并被视为紧接着声明导入的文件下面插入的额外文件。

例如，你在classpath的`application.properties`文件中可能有以下内容。

```properties
spring.application.name=myapp
spring.config.import=optional:file:./dev.properties
```

这将触发导入当前目录下的 `dev.properties` 文件（如果存在这样的文件）。导入的`dev.properties`中的值将优先于触发导入的文件。在上面的例子中，`dev.properties`可以将`spring.application.name`重新定义为一个不同的值。

一个导入只能被导入一次，无论它被声明多少次。一个导入在properties/yaml文件中的单个文件中被定义的顺序并不重要。例如，下面的两个例子产生相同的结果。

```properties
spring.config.import=my.properties
my.property=value
```

```properties
my.property=value
spring.config.import=my.properties
```

在上述两个例子中，`my.properties`文件的值将优先于触发其导入的文件。

在一个单一的`spring.config.import`键下可以指定多个位置。位置将按照它们被定义的顺序进行处理，后来的导入将被优先考虑。

在适当的时候，[Profile-specific variants](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific) 也被考虑导入。上面的例子将导入 `my.properties` 以及任何 `my-<profile>.properties` 变体。

Spring Boot包括可插入的API，允许支持各种不同的位置地址。默认情况下，你可以导入Java属性、YAML和 [configuration trees](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.configtree)。

第三方的jars可以提供对其他技术的支持（不要求文件是本地的）。例如，你可以想象配置数据来自外部存储，如Consul、Apache ZooKeeper或Netflix Archaius。

如果你想支持你自己的位置，请参阅`org.springframework.boot.context.config`包中的`ConfigDataLocationResolver`和`ConfigDataLoader`类。

#### 2.3.5. 导入无扩展名的文件

有些云平台不能为卷装文件添加文件扩展名。要导入这些无扩展名的文件，你需要给Spring Boot一个提示，以便它知道如何加载它们。你可以通过在方括号里放一个扩展名提示来做到这一点。

例如，假设你有一个`/etc/config/myconfig`文件，你希望以yaml形式导入。你可以用下面的方法从你的`application.properties`中导入它。

```properties
spring.config.import=file:/etc/config/myconfig[.yaml]
```

#### 2.3.6. 使用Configuration Trees

当在云平台（如Kubernetes）上运行应用程序时，你经常需要读取平台提供的配置值。为这种目的使用环境变量并不罕见，但这可能有缺点，特别是如果该值被认为是secret的。

作为环境变量的替代品，许多云平台现在允许你将配置映射到安装的数据卷中。例如，Kubernetes可以将[`ConfigMaps`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#populate-a-volume-with-data-stored-in-a-configmap)和[`Secrets`](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)都装入卷中。

有两种常见的卷装模式可以使用。

1. 一个单一的文件包含一套完整的属性（通常写成YAML）。
2. 多个文件被写入一个目录树，文件名成为 "key"，内容成为 "value"。

对于第一种情况，你可以使用`spring.config.import`直接导入YAML或属性文件，如[上文](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.importing)所述。对于第二种情况，你需要使用`configtree:`前缀，以便Spring Boot知道它需要将所有文件作为属性公开。

作为一个例子，让我们想象一下，Kubernetes已经安装了以下卷。

```text
etc/
  config/
    myapp/
      username
      password
```

`username`文件的内容将是一个配置值，而`password`的内容将是一个secret。

要导入这些属性，你可以在你的`application.properties`或`application.yaml`文件中添加以下内容。

```yaml
spring:
  config:
    import: "optional:configtree:/etc/config/"
```

然后你可以从 `Environment` 中以常规方式访问或注入`myapp.username`和`myapp.password`属性。

> 配置树的值可以被绑定到字符串`String`和`byte[]`类型，这取决于预期的内容。

如果你有多个配置树要从同一个父文件夹导入，你可以使用通配符快捷方式。任何以`/*/`结尾的`configtree`:位置都将导入所有直接的子文件夹作为配置树。

例如，给定以下卷。

```text
etc/
  config/
    dbconfig/
      db/
        username
        password
    mqconfig/
      mq/
        username
        password
```

你可以使用`configtree:/etc/config/*/`作为导入位置。

```properties
spring.config.import=optional:configtree:/etc/config/*/
```

这将添加`db.username`、`db.password`、`mq.username`和`mq.password`属性。

> 使用通配符加载的目录是按字母顺序排列的。如果你需要一个不同的顺序，那么你应该把每个位置作为一个单独的导入列出

配置树也可以用于Docker secrets。当Docker swarm服务被授予对secrets的访问权时，该secrets会被装载到容器中。例如，如果一个名为`db.password`的secrets被挂载在`/run/secrets/`的位置，你可以用以下方法让`db.password`对Spring环境可用。

```properties
spring.config.import=optional:configtree:/run/secrets/
```

#### 2.3.7. 属性占位符

`application.properties`和`application.yml`中的值在使用时通过现有的`Environment`过滤，所以你可以参考以前定义的值（例如，从系统属性）。标准的`${name}`属性占位符语法可以在一个值的任何地方使用。

例如，下面的文件将把`app.description`设置为 "MyApp is a Spring Boot application"。

```yaml
app:
  name: "MyApp"
  description: "${app.name} is a Spring Boot application"
```

> 你也可以使用这种技术来创建现有Spring Boot属性的 "short" 变体。详见[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.properties-and-configuration.short-command-line-arguments)的方法。

#### 2.3.8. 处理多文档文件

Spring Boot允许你将一个物理文件分成多个逻辑文件，每个文件都是独立添加的。文件是按顺序处理的，从上到下。后面的文件可以覆盖前面文件中定义的属性。

对于`application.yml`文件，使用标准的YAML多文档语法。三个连续的连字符代表一个文件的结束，以及下一个文件的开始。

例如，下面的文件有两个逻辑文档。

```yaml
spring.application.name: MyApp
---
spring.config.activate.on-cloud-platform: kubernetes
spring.application.name: MyCloudApp
```

对于 `application.properties` 文件，一个特殊的 `#---`注释被用来标记文件的分割。

```properties
spring.application.name=MyApp
#---
spring.config.activate.on-cloud-platform=kubernetes
spring.application.name=MyCloudApp
```

属性文件的分隔符不能有任何前导空白，而且必须正好有三个连字符。分隔符的前后两行不能是注释。

多文档属性文件通常与激活属性一起使用，如`spring.config.activated.on-profile`。详见[下一节](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.activation-properties)。

不能通过使用`@PropertySource`或`@TestPropertySource`注解加载多文档属性文件。

#### 2.3.9. 激活配置

有时，只有在满足某些条件时才能激活一个给定的属性。例如，你可能有一些属性只有在特定的配置文件被激活时才相关。

你可以使用`spring.config.activation.*`有条件地激活一个属性文件。

以下的激活属性是可用的。

表 2. activation properties

|Property|Note|
| --- | --- |
|`on-profile`|一个配置文件表达式，必须与之匹配才能使文件处于活动状态。|
|`on-cloud-platform`|必须检测到的 `CloudPlatform`，以使文件处于活动状态。|

例如，下面指定第二个文件只有在`Kubernetes`上运行时才有效，并且只有在 `prod` 或 `staging` 配置文件激活时才有效。

```yaml
myprop:
  always-set
---
spring:
  config:
    activate:
      on-cloud-platform: "kubernetes"
      on-profile: "prod | staging"
myotherprop: sometimes-set
```

### 2.4. 加密配置属性

Spring Boot没有为加密属性值提供任何内置支持，但它确实提供了修改Spring `Environment`中包含的值所需的hook。`EnvironmentPostProcessor` 接口允许你在应用程序启动前对 `Environment` 进行操作。详见[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.application.customize-the-environment-or-application-context)。

如果你正在寻找一种安全的方式来存储证书和密码，[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)项目提供了对[HashiCorp Vault](https://www.vaultproject.io/)中存储外部化配置的支持。

### 2.5. 使用YAML工作

[YAML](https://yaml.org/)是JSON的超集，因此是指定分层配置数据的方便格式。只要你的classpath上有[SnakeYAML](https://bitbucket.org/asomov/snakeyaml)库，`SpringApplication`类就会自动支持YAML作为属性的替代品。

> 如果你使用 "Starter"，SnakeYAML将由`spring-boot-starter`自动提供。

#### 2.5.1. 将YAML映射到属性

YAML 文档需要从其分层格式转换为可与 Spring `Environment`一起使用的扁平结构。例如，考虑下面这个YAML文档。

```yaml
environments:
  dev:
    url: https://dev.example.com
    name: Developer Setup
  prod:
    url: https://another.example.com
    name: My Cool App
```

为了从 `Environment` 中访问这些属性，它们将被扁平化，如下所示。

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

同样地，YAML列表也需要进行扁平化处理。它们被表示为带有`[index]`脱引器的属性键。例如，考虑下面的YAML。

```yaml
my:
 servers:
 - dev.example.com
 - another.example.com
```

前面的例子将被转化为这些属性

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

使用 `[index]` 符号的属性可以使用Spring Boot的 `Binder` 类绑定到Java `List` 或 `Set` 对象。详情请见下面的 [类型安全的配置属性](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties)部分。

YAML文件不能通过使用`@PropertySource`或`@TestPropertySource`注解来加载。所以，在你需要以这种方式加载值的情况下，你需要使用一个properties文件。

#### 2.5.2. 直接加载YAML

Spring Framework提供了两个方便的类，可以用来加载YAML文档。`YamlPropertiesFactoryBean`将YAML作为`Properties` 加载，`YamlMapFactoryBean`将YAML作为`Map` 加载。

如果你想将YAML作为Spring的`PropertySource`来加载，你也可以使用`YamlPropertySourceLoader`类。

### 2.6. 配置随机值

`RandomValuePropertySource` 对于注入随机值很有用（例如，注入secrets或test cases）。它可以产生Integer、Long、UUID或字符串，如下面的例子所示。

```yaml
my:
  secret: "${random.value}"
  number: "${random.int}"
  bignumber: "${random.long}"
  uuid: "${random.uuid}"
  number-less-than-ten: "${random.int(10)}"
  number-in-range: "${random.int[1024,65536]}"
```

`random.int*`的语法是`OPEN value (,max) CLOSE`，其中`OPEN,CLOSE`是任何字符，`value,max`是整数。如果提供了`max`，那么`value`是最小值，`max`是最大值（独占）。

### 2.7. 配置系统环境属性

Spring Boot支持为环境属性设置一个前缀。如果系统环境被多个具有不同配置要求的Spring Boot应用程序共享，这将非常有用。系统环境属性的前缀可以直接在`SpringApplication`上设置。

例如，如果你将前缀设置为`input`，诸如`remote.timeout`这样的属性在系统环境中也将被解析为`input.remote.timeout`。

### 2.8. 类型安全的配置属性

使用`@Value("${property}")`注解来注入配置属性有时会很麻烦，特别是当你要处理多个属性或你的数据是分层的。Spring Boot提供了一种处理属性的替代方法，让强类型的Bean管理和验证你的应用程序的配置。

> 另请参见[`@Value`和类型安全配置属性的区别](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.vs-value-annotation)。

#### 2.8.1. JavaBean属性绑定

如下面的例子所示，可以绑定一个声明了标准JavaBean属性的bean。

```java
import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my.service")
public class MyProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() {
        return this.enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() {
            return this.username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return this.password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

        public List<String> getRoles() {
            return this.roles;
        }

        public void setRoles(List<String> roles) {
            this.roles = roles;
        }

    }

}
```

前面的POJO定义了以下属性。

* `my.service.enabled`, 默认值为`false`。
* `my.service.remote-address`, 具有一个可以从`String'强制的类型。
* `my.service.security.username` ，有一个嵌套的 `security` 对象，其名称由属性名称决定。特别是，那里根本没有使用返回类型，可以是`SecurityProperties`。
* `my.service.security.password` 。
* `my.service.security.role` ，有一个默认为`USER`的`String`集合。

映射到Spring Boot中可用的`@ConfigurationProperties`类的属性，通过属性文件、YAML文件、环境变量等进行配置，是公共API，但类本身的访问器（getters/setters）并不意味着可以直接使用。

这样的安排依赖于一个默认的空构造函数，而获取器和设置器通常是强制性的，因为绑定是通过标准的Java Beans属性描述符，就像Spring MVC中一样。在下列情况下，可以省略设置器。

* Map，只要它们被初始化，就需要一个getter，但不一定需要setter，因为它们可以被绑定器改变。
* Collection和Array可以通过索引（通常用YAML）或使用单个逗号分隔的值（属性）来访问。在后一种情况下，一个setter是必须的。我们建议总是为这类类型添加一个setter。如果你初始化一个集合，确保它不是不可变的（如前面的例子）。
* 如果嵌套的POJO属性被初始化（就像前面的例子中的`security`字段），就不需要setter了。如果你想让绑定器通过使用它的默认构造函数来即时创建实例，你需要一个setter。

有些人使用Project Lombok来自动添加getter和setter。确保Lombok不会为这样的类型生成任何特定的构造函数，因为容器会自动使用它来实例化对象。

最后，只考虑标准的Java Bean属性，不支持对静态属性的绑定。

#### 2.8.2. 构造函数绑定

上一节的例子可以用不可变的方式重写，如下例所示。

```java
import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;

@ConstructorBinding
@ConfigurationProperties("my.service")
public class MyProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public MyProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public boolean isEnabled() {
        return this.enabled;
    }

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        public String getUsername() {
            return this.username;
        }

        public String getPassword() {
            return this.password;
        }

        public List<String> getRoles() {
            return this.roles;
        }

    }

}
```

在这个设置中，`@ConstructorBinding`注解被用来表示应该使用构造函数绑定。这意味着绑定器将期望找到一个具有你希望绑定的参数的构造函数。

`@ConstructorBinding`类的嵌套成员（比如上面例子中的`Security`）也将通过其构造函数被绑定。

默认值可以使用`@DefaultValue`来指定，同样的转换服务将被应用于将`String`值强制到缺失属性的目标类型。默认情况下，如果没有属性被绑定到`Security`，`MyProperties`实例将包含一个`security`的`null`值。如果你希望返回一个非空的`Security`实例，即使没有属性与之绑定，你可以使用一个空的`@DefaultValue`注释来实现。

```java
public MyProperties(boolean enabled, InetAddress remoteAddress, @DefaultValue Security security) {
    this.enabled = enabled;
    this.remoteAddress = remoteAddress;
    this.security = security;
}
```

要使用构造函数绑定，该类必须使用`@EnableConfigurationProperties`或配置属性扫描启用。你不能对通过常规Spring机制创建的Bean使用构造函数绑定（例如：`@Component`Bean，通过`@Bean`方法创建的Bean或使用`@Import`加载的Bean）。

如果你的类有一个以上的构造函数，你也可以直接在应该被绑定的构造函数上使用`@ConstructorBinding`。

不建议将`java.util.Optional`与`@ConfigurationProperties`一起使用，因为它主要是作为一个返回类型使用。因此，它并不适合配置属性注入。为了与其他类型的属性保持一致，如果你确实声明了一个`Optional`属性，但它没有值，`null`而不是一个空的`Optional`将被绑定。

#### 2.8.3. 启用@ConfigurationProperties-annotated类型

Spring Boot提供了绑定`@ConfigurationProperties`类型并将其注册为Bean的基础设施。你可以在逐个类的基础上启用配置属性，或者启用配置属性扫描，其工作方式与组件扫描类似。

有时，用`@ConfigurationProperties`注解的类可能不适合扫描，例如，如果你正在开发你自己的自动配置或者你想有条件地启用它们。在这些情况下，使用`@EnableConfigurationProperties` 注解指定要处理的类型列表。这可以在任何`@Configuration`类上进行，如下面的例子所示。

```java
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(SomeProperties.class)
public class MyConfiguration {

}
```

要使用配置属性扫描，请将`@ConfigurationPropertiesScan`注解添加到你的应用程序。通常情况下，它被添加到用`@SpringBootApplication`注解的主应用程序类中，但它也可以被添加到任何`@Configuration`类。默认情况下，扫描将从声明该注解的类的包中发生。如果你想定义特定的包来扫描，你可以这样做，如下面的例子所示。

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationPropertiesScan;

@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "com.example.another" })
public class MyApplication {

}
```

> 当`@ConfigurationProperties` Bean使用配置属性扫描或通过`@EnableConfigurationProperties`注册时，Bean有一个传统的名字：`<prefix>-<fqn>`，其中`<prefix>`是`@ConfigurationProperties`注解中指定的环境键前缀，`<fqn>`是Bean的完全限定名称。如果注解没有提供任何前缀，则只使用Bean的完全合格名称。
>
> 上面例子中的Bean名称是`com.example.app-com.example.app.SomeProperties`。

我们建议`@ConfigurationProperties`只处理环境，特别是不从上下文注入其他Bean。对于角落里的情况，可以使用设置器注入或框架提供的任何`*Aware`接口（如`EnvironmentAware`，如果你需要访问`Environment`）。如果你仍然想使用构造器注入其他Bean，配置属性Bean必须用`@Component`来注释，并使用基于JavaBean的属性绑定。

#### 2.8.4. 使用@ConfigurationProperties-annotated类型

这种配置风格与SpringApplication的外部YAML配置配合得特别好，如下例所示。

```yaml
my:
    service:
        remote-address: 192.168.1.1
        security:
            username: admin
            roles:
              - USER
              - ADMIN

```

要使用`@ConfigurationProperties` bean，你可以用与其他bean相同的方式注入它们，如下例所示

```java
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final SomeProperties properties;

    public MyService(SomeProperties properties) {
        this.properties = properties;
    }

    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        server.start();
        // ...
    }

    // ...

}
```

> 使用`@ConfigurationProperties`还可以让你生成元数据文件，可以被IDE用来为你自己的键提供自动完成。详见[附录](https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html#configuration-metadata)。

#### 2.8.5. 第三方配置

除了使用`@ConfigurationProperties`来注释一个类之外，你也可以在公共的`@Bean`方法上使用它。当你想把属性绑定到你控制之外的第三方组件时，这样做特别有用。

要从`Environment`属性中配置一个Bean，请将`@ConfigurationProperties`添加到其Bean注册中，如下例所示。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class ThirdPartyConfiguration {

    @Bean
    @ConfigurationProperties(prefix = "another")
    public AnotherComponent anotherComponent() {
        return new AnotherComponent();
    }

}
```

任何用`another`前缀定义的JavaBean属性都会被映射到`AnotherComponent`Bean上，其方式类似于前面的`SomeProperties`例子。

#### 2.8.6. 宽松的绑定

Spring Boot在将 `Environment` 属性绑定到`@ConfigurationProperties`bean时使用了一些宽松的规则，因此 `Environment` 属性名称和bean属性名称之间不需要完全匹配。这很有用，常见的例子包括破折号分隔的环境属性（例如，`context-path`绑定到`contextPath`），和大写的环境属性（例如，`PORT`绑定到`port`）。

作为一个例子，考虑下面的`@ConfigurationProperties`类。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "my.main-project.person")
public class MyPersonProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

通过前面的代码，以下的属性名称都可以使用。

表 3. relaxed binding

|Property|Note|
| --- | --- |
|`my.main-project.person.first-name`|短横线案例，建议在`.properties`和`.yml`文件中使用。|
|`my.main-project.person.firstName`|标准的驼峰语法。|
|`my.main-project.person.first_name`|下划线符号，这是一种用于`.properties`和`.yml`文件的替代格式。|
|`MY_MAINPROJECT_PERSON_FIRSTNAME`|大写格式，在使用系统环境变量时建议使用大写格式。|

> 注释的 `prefix` 值必须是短横线（小写并以`-`分隔，如`my.main-project.person`）。

表 4. relaxed binding rules per property source

|配置源|Simple|List|
| --- | --- | --- |
|Properties 文件|驼峰、短横线或下划线|使用`[]`或逗号分隔值的标准列表语法|
|YAML 文件|驼峰、短横线或下划线|标准YAML列表语法或逗号分隔的值|
|Environment Variables|大写格式，下划线为分隔符（见[从环境变量绑定](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)）。|用下划线包围的数值（见[从环境变量绑定](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding.environment-variables)）。|
|System properties|驼峰、短横线或下划线|使用`[]`或逗号分隔值的标准列表语法|

> 我们建议，在可能的情况下，属性应以小写的短横线格式存储，例如`my.person.first-name=Rod`。

Binding Maps

当绑定到`Map`属性时，你可能需要使用一个特殊的括号符号，以便保留原始的`key`值。如果键没有被`[]`包围，任何非字母数字、`-`或`.`的字符都会被删除。

例如，考虑将以下属性绑定到一个`Map<String,String>`。

```yaml
my:
  map:
    "[/key1]": "value1"
    "[/key2]": "value2"
    "/key3": "value3"

```

对于YAML文件，括号需要用引号包围，以使键被正确解析。

上面的属性将绑定到一个`Map`，`/key1`，`/key2`和`key3`作为地图的键。斜线已经从`key3`中移除，因为它没有被方括号包围。

如果你的`key`包含一个`.`，并且你要绑定到非标量值，你可能偶尔也需要使用括号符号。例如，将`a.b=c`绑定到`Map<String, Object>`将返回一个Map，其条目为`{"a"={"b"="c"}`，而`[a.b]=c`将返回一个Map，其条目为`{"a.b"="c"}`。

从环境变量绑定

大多数操作系统对可用于环境变量的名称有严格的规定。例如，Linux shell变量只能包含字母（`a`到`z`或`A`到`Z`）、数字（`0`到`9`）或下划线字符（`_`）。按照惯例，Unix shell变量的名称也将采用大写字母。

Spring Boot宽松的绑定规则被设计为尽可能与这些命名限制兼容。

要将规范形式的属性名称转换为环境变量名称，你可以遵循这些规则。

* 用下划线 ( `_` ) 替换点 ( `.` ) 。
* 删除任何破折号 ( `-` )。
* 转换为大写字母。

例如，配置属性`spring.main.log-startup-info`将是一个名为`SPRING_MAIN_LOGSTARTUPINFO`的环境变量。

环境变量也可以在绑定到对象列表时使用。要绑定到一个`List`，在变量名称中，元素编号应该用下划线包围。

例如，配置属性`my.service[0].other`将使用一个名为`MY_SERVICE_0_OTHER`的环境变量。

#### 2.8.7. 合并复杂类型

当列表被配置在多个地方时，覆盖的作用是替换整个列表。

例如，假设一个`MyPojo`对象的`name`和`description`属性默认为`null`。下面的例子从`MyProperties`暴露了一个`MyPojo`对象的列表。

```java
import java.util.ArrayList;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my")
public class MyProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

考虑以下配置。

```yaml
my:
  list:
  - name: "my name"
    description: "my description"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  list:
  - name: "my another name"
```

如果 `dev` 配置文件没有激活，`MyProperties.list` 包含一个 `MyPojo` 条目，如之前定义的。然而，如果 `dev` 配置文件被激活，`list` 仍然只包含一个条目（名字为 `my another name`，描述为 `null`）。这种配置*不会*在列表中添加第二个`MyPojo`实例，也不会合并项目。

当一个 `List` 在多个配置文件中被指定时，具有最高优先级的一个（而且只有那个）被使用。考虑一下下面的例子。

```yaml
my:
  list:
  - name: "my name"
    description: "my description"
  - name: "another name"
    description: "another description"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  list:
  - name: "my another name"

```

在前面的例子中，如果`dev`配置文件是激活的，`MyProperties.list`包含*一个*`MyPojo`条目（名字为`my another name`，描述为`null`）。对于YAML，逗号分隔的列表和YAML列表都可以用来完全覆盖列表的内容。

对于`Map`属性，你可以用从多个来源抽取的属性值进行绑定。然而，对于多个来源中的同一属性，使用具有最高优先级的那个。下面的例子从`MyProperties`暴露了一个`Map<String, MyPojo>`。

```java
import java.util.LinkedHashMap;
import java.util.Map;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("my")
public class MyProperties {

    private final Map<String, MyPojo> map = new LinkedHashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```

考虑以下配置。

```yaml
my:
  map:
    key1:
      name: "my name 1"
      description: "my description 1"
---
spring:
  config:
    activate:
      on-profile: "dev"
my:
  map:
    key1:
      name: "dev name 1"
    key2:
      name: "dev name 2"
      description: "dev description 2"

```

如果`dev`配置文件没有激活，`MyProperties.map`包含一个键`key1`的条目（名称为`my name 1`，描述为`my description 1`）。然而，如果 `dev` 配置文件被激活，`map`包含两个条目，键为`key1`（名称为`dev name 1`，描述为`my description 1`）和`key2`（名称为`dev name 2`，描述为`dev description 2`）。

> 前面的合并规则适用于所有属性源的属性，而不仅仅是文件。

#### 2.8.8. 属性转换

当Spring Boot与`@ConfigurationProperties` Bean绑定时，它试图将外部应用程序的属性胁迫为正确的类型。如果你需要自定义类型转换，你可以提供一个`ConversionService` bean（有一个名为 `conversionService` 的bean）或自定义属性编辑器（通过`CustomEditorConfigurer`bean）或自定义`Converters`（有注释为`@ConfigurationPropertiesBinding`的bean定义）。

由于这个Bean是在应用程序生命周期的早期被请求的，请确保限制你的`ConversionService'``ConversionService`不需要配置键强制，你可能想重命名它，并且只依赖用`@ConfigurationPropertiesBinding`限定的自定义转换器。

##### Converting Durations

Spring Boot对表达持续时间有专门的支持。如果你公开了一个`java.time.Duration`属性，应用程序属性中就有以下格式。

* 普通的`long`表示法（使用毫秒作为默认单位，除非指定了`@DurationUnit`）。
* 标准的ISO-8601格式[由`java.time.Duration`使用](https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html#parse-java.lang.CharSequence-)
* 一个更易读的格式，其中值和单位是耦合的（例如，`10s`表示10秒）

请考虑以下例子。

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.convert.DurationUnit;

@ConfigurationProperties("my")
public class MyProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```

要指定一个30秒的会话超时，`30`、`PT30S`和`30s`都是等价的。读取超时为500ms，可以用以下任何一种形式指定。`500`, `PT0.5S`和`500ms`。

你也可以使用任何支持的单位。这些单位是

* `ns`代表纳秒
* `us`代表微秒
* `ms`代表毫秒
* `s`代表秒
* `m`代表分钟
* `h`代表小时
* `d`代表天

默认单位是毫秒，可以使用`@DurationUnit`来重写，如上面的例子所示。

如果你喜欢使用构造函数绑定，同样的属性可以被暴露出来，如下面的例子所示。

```java
import java.time.Duration;
import java.time.temporal.ChronoUnit;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;
import org.springframework.boot.convert.DurationUnit;

@ConfigurationProperties("my")
@ConstructorBinding
public class MyProperties {

    private final Duration sessionTimeout;

    private final Duration readTimeout;

    public MyProperties(@DurationUnit(ChronoUnit.SECONDS) @DefaultValue("30s") Duration sessionTimeout,
            @DefaultValue("1000ms") Duration readTimeout) {
        this.sessionTimeout = sessionTimeout;
        this.readTimeout = readTimeout;
    }

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

}
```

如果你正在升级一个`Long属`性，如果它不是毫秒，请确保定义单位（使用`@DurationUnit`）。这样做提供了一个透明的升级路径，同时支持更丰富的格式。

Converting periods

除了期限，Spring Boot还可以使用`java.time.Period`类型。以下格式可以在应用程序属性中使用。

* 常规的`int`表示法（使用天作为默认单位，除非指定了`@PeriodUnit`）。
* 标准的ISO-8601格式[由`java.time.Period`使用](https://docs.oracle.com/javase/8/docs/api/java/time/Period.html#parse-java.lang.CharSequence-)
* 一个更简单的格式，其中值和单位对是耦合的（例如，`1y3d`表示1年3天)

简单格式支持以下单位。

* `y`代表年
* `m`代表月
* `w`代表周
* `d`代表天

`java.time.Period`类型实际上从未存储过周数，它是一个快捷方式，意味着 "7天"。

##### 转换数据大小

Spring Framework有一个`DataSize`值类型，以字节为单位表达大小。如果你公开了一个`DataSize`属性，在应用程序属性中可以使用以下格式。

* 常规的 `long` 表示（使用字节作为默认单位，除非指定了`@DataSizeUnit`）。
* 一个更易读的格式，其中值和单位是耦合的（例如，`10MB`意味着10兆字节）。

考虑一下下面的例子。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties("my")
public class MyProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```

要指定一个10兆字节的缓冲区大小，`10`和`10MB`是等价的。256字节的大小阈值可以指定为`256`或`256B`。

你也可以使用任何支持的单位。这些单位是

* `B`代表字节
* `KB`代表千字节
* `MB`表示兆字节
* `GB`代表千兆字节
* `TB`代表太字节

默认单位是字节，可以使用`@DataSizeUnit`来重写，如上面的例子所示。

如果你喜欢使用构造函数绑定，同样的属性可以被暴露出来，如下面的例子所示。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;
import org.springframework.boot.convert.DataSizeUnit;
import org.springframework.util.unit.DataSize;
import org.springframework.util.unit.DataUnit;

@ConfigurationProperties("my")
@ConstructorBinding
public class MyProperties {

    private final DataSize bufferSize;

    private final DataSize sizeThreshold;

    public MyProperties(@DataSizeUnit(DataUnit.MEGABYTES) @DefaultValue("2MB") DataSize bufferSize,
            @DefaultValue("512B") DataSize sizeThreshold) {
        this.bufferSize = bufferSize;
        this.sizeThreshold = sizeThreshold;
    }

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

}
```

> 如果你正在升级一个`Long`属性，如果它不是字节，请确保定义单位（使用`@DataSizeUnit`）。这样做提供了一个透明的升级路径，同时支持更丰富的格式。

#### 2.8.9. @ConfigurationProperties 验证

只要使用Spring的`@Validated`注解，Spring Boot就会尝试验证`@ConfigurationProperties`类。你可以直接在你的配置类上使用JSR-303的`javax.validation`约束注解。要做到这一点，请确保你的classpath上有一个兼容的JSR-303实现，然后将约束注解添加到你的字段中，如下面的例子所示。

```java
import java.net.InetAddress;

import javax.validation.constraints.NotNull;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

}
```

> 你也可以通过在创建配置属性的`@Bean`方法上注解`@Validated`来触发验证。

为了确保总是触发嵌套属性的验证，即使没有找到属性，相关的字段必须用`@Valid`来注释。下面的例子建立在前面的 `MyProperties` 的基础上。

```java
import java.net.InetAddress;

import javax.validation.Valid;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.validation.annotation.Validated;

@ConfigurationProperties("my.service")
@Validated
public class MyProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    public InetAddress getRemoteAddress() {
        return this.remoteAddress;
    }

    public void setRemoteAddress(InetAddress remoteAddress) {
        this.remoteAddress = remoteAddress;
    }

    public Security getSecurity() {
        return this.security;
    }

    public static class Security {

        @NotEmpty
        private String username;

        public String getUsername() {
            return this.username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

    }

}
```

你也可以通过创建一个名为 `configurationPropertiesValidator` 的bean定义来添加一个自定义的Spring`Validator`。`@Bean`方法应该被声明为`static`。配置属性验证器是在应用程序生命周期的早期创建的，将`@Bean`方法声明为静态，可以让Bean被创建而不需要实例化`@Configuration`类。这样做可以避免早期实例化可能引起的任何问题。

> `spring-boot-actuator`模块包括一个端点，暴露了所有`@ConfigurationProperties` Bean。将你的网络浏览器指向`/actuator/configprops`或使用相应的JMX端点。详见 [生产就绪特性](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints) 部分。

#### 2.8.10. @ConfigurationProperties vs. @Value

`@Value`注解是一个核心的容器功能，它不提供与类型安全的配置属性相同的功能。下表总结了`@ConfigurationProperties`和`@Value`所支持的功能。

|Feature|`@ConfigurationProperties`|`@Value`|
| --- | --- | --- |
|[Relaxed binding](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.relaxed-binding)|Yes|Limited (see [note below](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.vs-value-annotation.note))|
|[Meta-data support](https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html#configuration-metadata)|Yes|No|
|`SpEL` evaluation|No|Yes|

如果你想使用`@Value`，我们建议你使用属性名称的规范形式（只使用小写字母的kebab-case）。这将允许Spring Boot使用与放松绑定`@ConfigurationProperties`时一样的逻辑。例如，`@Value("{demo.item-price}")`将从`application.properties`文件中获取`demo.item-price`和`demo.itemPrice`形式，以及从系统环境中获取`DEMO_ITEMPRICE`。如果你用`@Value("{demo.itemPrice}")`代替，`demo.item-price`和`DEMO_ITEMPRICE`将不会被考虑。

> 如果你为自己的组件定义了一组配置键，我们建议你将它们归入一个用`@ConfigurationProperties`注解的POJO。这样做将为你提供结构化的、类型安全的对象，你可以将其注入到你自己的bean中。

来自[应用程序属性文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files)的`SpEL`表达式在解析这些文件和填充环境时不会被处理。然而，我们可以在`@Value`中写一个`SpEL`表达式。如果来自应用程序属性文件的属性值是一个`SpEL`表达式，当通过`@Value`消耗时，它将被评估。

## 3. Profiles

Spring Profiles提供了一种方法来隔离你的应用程序配置的一部分，使其只在某些环境下可用。任何`@Component`、`@Configuration` 或`@ConfigurationProperties`都可以用`@Profile`来标记，以限制它被加载的时间，如下面的例子所示。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

> 如果`@ConfigurationProperties` Bean是通过`@EnableConfigurationProperties`注册的，而不是自动扫描，则需要在具有`@EnableConfigurationProperties`注释的`@Configuration`类上指定`@Profile`注释。在`@ConfigurationProperties`被扫描的情况下，`@Profile` 可以在`@ConfigurationProperties` 类本身指定。

你可以使用`spring.profiles.active` `Environment`属性来指定哪些配置文件是活动的。你可以通过本章前面描述的任何方式来指定该属性。例如，你可以在你的`application.properties`中包含它，如下面的例子所示。

```yaml
spring:
  profiles:
    active: "dev,hsqldb"
```

你也可以通过使用以下开关在命令行中指定它：`--spring.profiles.active=dev,hsqldb` 。

如果没有激活配置文件，就会启用一个默认的配置文件。默认配置文件的名称是`default`，可以使用`spring.profiles.default``Environment`属性对其进行调整，如下面例子所示。

```yaml
spring:
  profiles:
    default: "none"
```

### 3.1. 添加活动配置文件

`spring.profiles.active`属性遵循与其他属性相同的排序规则。最高的`PropertySource`获胜。这意味着你可以在`application.properties`中指定活动配置文件，然后通过使用命令行开关替换它们。

有时，有一些属性可以添加到活动配置文件中，而不是替换它们，这很有用。`SpringApplication`入口点有一个Java API用于设置额外的配置文件（也就是在那些由`spring.profiles.active`属性激活的配置文件之上）。参见[SpringApplication](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/SpringApplication.html)中的`setAdditionalProfiles()`方法。配置文件组，在[下一节](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.profiles.groups)中描述，如果一个给定的配置文件是激活的，也可以用来添加激活的配置文件。

### 3.2. Profile Groups

偶尔，你在你的应用程序中定义和使用的配置文件过于精细，使用起来就会很麻烦。例如，你可能有`proddb`和`prodmq`配置文件，用来独立启用数据库和消息传递功能。

为了帮助解决这个问题，Spring Boot允许你定义配置文件组。配置文件组允许你为相关的配置文件组定义一个逻辑名称。

例如，我们可以创建一个`production`组，由`proddb`和`prodmq`配置文件组成。

```yaml
spring:
  profiles:
    group:
      production:
      - "proddb"
      - "prodmq"
```

现在可以使用`--spring.profiles.active=production`来启动我们的应用程序，以一次性激活`production`、`proddb`和`prodmq`配置文件。

### 3.3. 以编程方式设置配置文件

你可以在应用运行前通过调用`SpringApplication.setAdditionalProfiles(...)`以编程方式设置活动配置文件。也可以通过使用Spring的`ConfigurableEnvironment`接口来激活配置文件。

### 3.4. 特定的配置文件

`application.properties`（或`application.yml`）和通过`@ConfigurationProperties`引用的文件的配置文件特定变体都被视为文件并加载。详情请见 [配置文件特定文件](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.profile-specific)。

## 4. Logging

Spring Boot在所有内部日志中使用[Commons Logging](https://commons.apache.org/logging)，但对底层日志的实现保持开放。为[Java Util Logging](https://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html)、[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback](https://logback.qos.ch/)提供默认配置。在每一种情况下，记录器都被预设为使用控制台输出，也可以选择文件输出。

默认情况下，如果你使用 "Starter"，Logback被用来做日志记录。适当的Logback路由也包括在内，以确保使用Java Util Logging、Commons Logging、Log4J或SLF4J的依赖库都能正确工作。

有很多适用于Java的日志框架。如果上面的列表看起来很混乱，请不要担心。一般来说，你不需要改变你的日志依赖，Spring Boot的默认值就很好用。

当你把你的应用程序部署到servlet容器或应用服务器时，通过Java Util Logging API执行的日志不会被传送到你的应用程序的日志。这可以防止由容器或其他已经部署到它的应用程序执行的日志出现在你的应用程序的日志中。

### 4.1. 日志格式

Spring Boot的默认日志输出类似于下面的例子。

```text
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出的项目如下。

* 日期和时间：精确到毫秒，易于排序。
* 日志级别: `ERROR`, `WARN`, `INFO`, `DEBUG`, 或`TRACE`.
* 进程ID。
* 一个`---`分隔符来区分实际日志信息的开始。
* 线程名称：包含在方括号中（对于控制台输出可能会被截断）。
* 记录器名称：这通常是源类的名称（通常是缩写）。
* 日志消息。

> Logback没有`FATAL`级别。它被映射到 `ERROR`。

### 4.2. 控制台输出

默认的日志配置是在写信息的时候向控制台echo。默认情况下，`ERROR` 级别、`WARN` 级别和 `INFO` 级别的消息被记录下来。你也可以通过使用`--debug`标志启动你的应用程序来启用 `调debug` 模式。

```shell
$ java -jar myapp.jar --debug
```

> 你也可以在你的 `application.properties`中指定`debug=true`。

当调试模式被启用时，一些核心记录器（嵌入式容器、Hibernate和Spring Boot）被配置为输出更多信息。启用调试模式并不配置你的应用程序以`DEBUG`级别记录所有信息。

另外，你可以通过在启动应用程序时设置`--trace`标志（或在`application.properties`中设置`trace=true`）来启用 `trace` 模式。这样做可以对一些核心记录器（嵌入式容器、Hibernate模式生成和整个Spring组合）进行跟踪记录。

#### 4.2.1. 彩色编码的输出

如果你的终端支持ANSI，就会使用彩色输出来帮助阅读。你可以将`spring.output.ansi.enabled`设置为一个[支持的值](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)来覆盖自动检测。

颜色编码是通过使用`%clr`转换词来配置的。在其最简单的形式下，转换器根据日志级别对输出进行着色，如下面的例子所示。

```text
%clr(%5p)
```

下表描述了日志级别与颜色的映射。

|级别|颜色|
| --- | --- |
|`FATAL`|红色|
|`ERROR`|红色|
|`WARN`|黄色|
|`INFO`|绿色|
|`DEBUG`|绿色|
|`TRACE`|绿色

另外，你也可以通过为转换提供一个选项来指定应该使用的颜色或样式。例如，要使文本为黄色，请使用以下设置。

```text
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式。

* `blue`
* `cyan`
* `faint`
* `green`
* `magenta`
* `red`
* `yellow`

### 4.3. 输出到文件

默认情况下，Spring Boot只向控制台记录日志，不写日志文件。如果你想在控制台输出之外写日志文件，你需要设置`logging.file.name`或`logging.file.path`属性（例如，在你的`application.properties`）。

下表显示了`logging.*`属性如何被一起使用。

Table 5. Logging properties

|`logging.file.name`|`logging.file.path`|Example|Description|
| --- | --- | --- | --- |
|*(none)*|*(none)*||只在控制台进行记录。|
|指定文件|*(none)*|`my.log`|写到指定的日志文件。名称可以是准确的位置，也可以是与当前目录的相对位置。|
|*(none)*|指定目录|`/var/log`|将`spring.log`写到指定目录。名称可以是准确的位置，也可以是与当前目录的相对位置。|

日志文件在达到10MB时就会轮换，与控制台输出一样，默认情况下会记录`ERROR`-级、`WARN`-级和`INFO`-级的信息。

> Logging properties 独立于实际的日志基础设施。因此，特定的配置KEY（如Logback的`logback.configurationFile`）不由spring Boot管理。

### 4.4. 文件轮换

如果你使用`Logback`，可以使用你的`application.properties`或`application.yaml`文件来微调日志轮换设置。对于所有其他的日志系统，你需要自己直接配置旋转设置（例如，如果你使用Log4J2，那么你可以添加一个`log4j.xml`文件）。

支持以下轮换策略属性。

|Name|Description|
| --- | --- |
|`logging.logback.rollingpolicy.file-name-pattern`|用于创建日志档案的文件名模式。|
|`logging.logback.rollingpolicy.clean-history-on-start`|如果应用程序启动时应进行日志归档清理。|
|`logging.logback.rollingpolicy.max-file-size`|日志文件归档前的最大尺寸。|
|`logging.logback.rollingpolicy.total-size-cap`|日志档案在被删除前的最大尺寸。|
|`logging.logback.rollingpolicy.max-history`|保存日志档案的天数（默认为7天）。|

### 4.5. 日志级别

所有支持的日志系统都可以通过使用`logging.level.<logger-name>=<level>`在Spring的`Environment`中（例如，在`application.properties`中）设置日志级别，其中`level`是TRACE, DEBUG, INFO, WARN, ERROR, FATAL, 或OFF之一。`root`记录器可以通过使用`logging.level.root`来配置。

下面的例子显示了`application.properties`中潜在的日志设置。

```yaml
logging:
  level:
    root: "warn"
    org.springframework.web: "debug"
    org.hibernate: "error"

```

也可以使用环境变量来设置日志级别。例如，`LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG`将设置`org.springframework.web`为`DEBUG`。

> 上述方法只适用于包级日志。由于放松的绑定总是将环境变量转换为小写字母，所以不可能用这种方式为单个类配置日志。如果你需要为一个类配置日志，你可以使用[the `SPRING_APPLICATION_JSON`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.application-json)变量。

### 4.6. Log Groups

能够将相关的日志记录器分组，以便同时对它们进行配置，这通常很有用。例如，你可能经常改变所有Tomcat相关的日志记录器的日志级别，但你不容易记住最高级别的包。

为了帮助解决这个问题，Spring Boot允许你在Spring `Environment`中定义日志组。例如，你可以这样定义一个 "tomcat" 组，把它添加到你的`application.properties`中。

```yaml
logging:
  group:
    tomcat: "org.apache.catalina,org.apache.coyote,org.apache.tomcat"
```

一旦定义，就可以用一行字来改变组中所有记录仪的级别。

```yaml
logging:
  level:
    tomcat: "trace"
```

Spring Boot包括以下预定义的日志组，可以开箱即用。

|Name|Loggers|
| --- | --- |
|web|`org.springframework.core.codec` , `org.springframework.http` , `org.springframework.web` , `org.springframework.boot.actuate.endpoint.web` , `org.springframework.boot.web.servlet.ServletContextInitializerBeans`|
|sql|`org.springframework.jdbc.core` , `org.hibernate.SQL` , `org.jooq.tools.LoggerListener`|

### 4.7. 使用日志停机Hook

为了在你的应用程序终止时释放日志资源，我们提供了一个关机Hook，它将在JVM退出时触发日志系统清理。这个关机钩子是自动注册的，除非你的应用程序是以war文件的形式部署。如果你的应用程序有复杂的上下文层次结构，关闭Hook可能无法满足你的需求。如果不能，请禁用关机Hook，并研究底层日志系统直接提供的选项。例如，Logback提供了[context selectors](http://logback.qos.ch/manual/loggingSeparation.html)，允许每个Logger在它自己的上下文中被创建。你可以使用`logging.register-shutdown-hook`属性来禁用关机Hook。将其设置为`false`将禁用注册。你可以在你的`application.properties`或`application.yaml`文件中设置该属性。

```yaml
logging:
  register-shutdown-hook: false
```

### 4.8. 自定义日志配置

各种日志系统可以通过在classpath上包含适当的库来激活，并且可以通过在classpath的根部或由以下Spring `Environment` 属性指定的位置提供合适的配置文件来进一步定制：`logging.config` 。

你可以通过使用`org.springframework.boot.logging.LoggingSystem`系统属性来强制Spring Boot使用一个特定的日志系统。该值应该是`LoggingSystem`实现的完全合格类名。你也可以通过使用`none`的值来完全禁用Spring Boot的日志配置。

由于日志是在创建`ApplicationContext`**之前**初始化的，所以不可能从Spring`@Configuration`文件中的`@PropertySources`控制日志。改变日志系统或完全禁用它的唯一方法是通过系统属性。

根据你的日志系统，会加载以下文件。

|Logging System|Customization|
| --- | --- |
|Logback|`logback-spring.xml` , `logback-spring.groovy` , `logback.xml` , or `logback.groovy`|
|Log4j2|`log4j2-spring.xml` or `log4j2.xml`|
|JDK (Java Util Logging)|`logging.properties`|

> 在可能的情况下，我们建议你使用`-spring`变体来进行日志配置（例如，`logback-spring.xml`而不是`logback.xml`）。如果你使用标准配置位置，Spring不能完全控制日志初始化。

当从 `executable jar`中运行时，Java Util Logging有一些已知的类加载问题，会导致问题。如果可能的话，我们建议你在从 "executable jar"中运行时避免它。

为了帮助定制，其他一些属性从Spring环境转移到系统属性，如下表所述。

|Spring Environment|System Property|Comments|
| --- | --- | --- |
|`logging.exception-conversion-word`|`LOG_EXCEPTION_CONVERSION_WORD`|记录异常时使用的转换词。|
|`logging.file.name`|`LOG_FILE`|如果定义了，它将用于默认的日志配置中。|
|`logging.file.path`|`LOG_PATH`|如果定义了，它将用于默认的日志配置中。|
|`logging.pattern.console`|`CONSOLE_LOG_PATTERN`|在控制台（stdout）使用的日志模式。|
|`logging.pattern.dateformat`|`LOG_DATEFORMAT_PATTERN`|日志日期格式的格式化。|
|`logging.charset.console`|`CONSOLE_LOG_CHARSET`|控制台使用的字符集。|
|`logging.pattern.file`|`FILE_LOG_PATTERN`|要在文件中使用的日志模式（如果`LOG_FILE`被启用）。|
|`logging.charset.file`|`FILE_LOG_CHARSET`|用于文件记录的字符集（如果`LOG_FILE`被启用）。|
|`logging.pattern.level`|`LOG_LEVEL_PATTERN`|呈现日志级别时使用的格式（默认为`%5p`）。|
|`PID`|`PID`|当前的进程ID（如果可能的话，在没有定义为操作系统环境变量的情况下被发现）。|

如果你使用Logback，以下属性也会被转移。

|Spring Environment|System Property|Comments|
| --- | --- | --- |
|`logging.logback.rollingpolicy.file-name-pattern`|`LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN`|滚动日志文件名的模式（默认为`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`）。|
|`logging.logback.rollingpolicy.clean-history-on-start`|`LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START`|是否在启动时清理归档日志文件。|
|`logging.logback.rollingpolicy.max-file-size`|`LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE`|最大日志文件大小。|
|`logging.logback.rollingpolicy.total-size-cap`|`LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP`|要保存的日志备份的总大小。|
|`logging.logback.rollingpolicy.max-history`|`LOGBACK_ROLLINGPOLICY_MAX_HISTORY`|要保留的最大归档日志文件数量。|

所有支持的日志系统在解析其配置文件时都可以查阅系统属性。例子见spring-boot.jar中的默认配置。

* [Logback](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
* [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
* [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

如果你想在日志属性中使用占位符，你应该使用[Spring Boot的语法](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files.property-placeholders)，而不是底层框架的语法。值得注意的是，如果你使用Logback，你应该使用`:`作为属性名和其默认值之间的分隔符，而不是使用:`-`。

你可以只通过覆盖 `LOG_LEVEL_PATTERN` (或 Logback 的 `logging.pattern.level`)来向日志行添加 MDC 和其他临时内容。例如，如果你使用`logging.pattern.level=user:%X{user} %5p` ，那么默认的日志格式包含一个 "user" 的MDC条目，如果它存在的话，如下例所示。

```text
2019-08-30 12:30:04.031 user:someone INFO 22174 --- [  nio-8080-exec-0] demo.Controller
Handling authenticated request
```

### 4.9. Logback扩展

Spring Boot包括一些对Logback的扩展，可以帮助进行高级配置。你可以在你的`logback-spring.xml`配置文件中使用这些扩展。

> 因为标准的`logback.xml`配置文件被过早加载，你不能在其中使用扩展。你需要使用`logback-spring.xml`或者定义一个`logging.config`属性。

这些扩展不能与Logback的[配置扫描](https://logback.qos.ch/manual/configuration.html#autoScan)一起使用。如果你试图这样做，对配置文件进行修改会导致类似于以下的错误被记录下来。

```text
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

#### 4.9.1. 特定配置文件配置

`<springProfile>`标签让你可以根据活动的Spring配置文件选择性地包括或排除配置的部分。配置文件部分支持在`<configuration>`元素的任何地方。使用`name`属性来指定接受配置的配置文件。`<springProfile>`标签可以包含一个配置文件名称（例如`staging`）或一个配置文件表达式。配置文件表达式允许表达更复杂的配置逻辑，例如`production & (eu-central | eu-west)`。查看[参考指南](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/core.html#beans-definition-profiles-java)了解更多细节。下面的列表显示了三个配置文件的例子。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

#### 4.9.2. Environment Properties

`<springProperty>`标签让你从Spring `Environment`中公开属性，以便在Logback中使用。如果你想在Logback配置中访问`application.properties`文件中的值，这样做会很有用。该标签的工作方式与Logback的标准`<property>`标签类似。然而，你不是直接指定一个 `value`，而是指定该属性的 `source`（来自 `Environment`）。如果你需要在 `local` 范围以外的地方存储该属性，你可以使用 `scope` 属性。如果你需要一个后备值（万一该属性没有在`Environment`中设置），你可以使用`defaultValue`属性。下面的例子显示了如何公开属性以便在Logback中使用。

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

`source`必须以短横线格式指定（如`my.property-name`）。然而，属性可以通过使用宽松的规则添加到 `Environment` 中。

## 5. 国际化

Spring Boot支持本地化的消息，这样你的应用程序就可以满足不同语言偏好的用户。默认情况下，Spring Boot会在classpath的根部寻找`messages`资源包的存在。

> 自动配置适用于已配置的资源包的默认属性文件（即默认为`messages.properties`）。如果你的资源包只包含特定语言的属性文件，你需要添加默认的。如果没有找到与任何配置的基础名称相匹配的属性文件，将没有自动配置的`MessageSource`。

资源包的基名以及其他几个属性可以使用`spring.messages`命名空间进行配置，如下例所示。

```yaml
spring:
  messages:
    basename: "messages,config.i18n.messages"
    fallback-to-system-locale: false
```

`spring.messages.basename`支持逗号分隔的位置列表，可以是包的限定词或从classpath根部解析的资源。

更多支持的选项见[`MessageSourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/context/MessageSourceProperties.java)。

## 6. JSON

Spring Boot提供与三个JSON映射库的集成。

* Gson
* Jackson
* JSON-B

Jackson是首选和默认的库。

### 6.1. Jackson

为Jackson提供自动配置，Jackson是`spring-boot-starter-json`的一部分。当Jackson在classpath上时，会自动配置一个`ObjectMapper`bean。提供了几个配置属性用于[定制 ObjectMapper 的配置](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.spring-mvc.customize-jackson-objectmapper)。

### 6.2. Gson

为Gson提供了自动配置。当Gson在classpath上时，会自动配置一个`Gson`bean。提供了几个`spring.gson.*`配置属性来定制配置。为了获得更多的控制权，可以使用一个或多个`GsonBuilderCustomizer` bean。

### 6.3. JSON-B

为JSON-B提供了自动配置功能。当JSON-B API和一个实现在classpath上时，一个`Jsonb`bean将被自动配置。首选的JSON-B实现是Apache Johnzon，为其提供了依赖性管理。

## 7. 开发 Web Applications

Spring Boot非常适用于Web Application开发。你可以通过使用嵌入式Tomcat、Jetty、Undertow或Netty创建一个独立的HTTP服务器。大多数Web应用程序使用`spring-boot-starter-web`模块来快速启动和运行。你也可以选择通过使用`spring-boot-starter-webflux`模块来构建reactive Web应用。

如果你还没有开发过Spring Boot的Web应用，你可以按照[入门](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.first-application)章节中的 "Hello World!"的例子来做。

### 7.1. Spring Web MVC 框架

[Spring Web MVC framework](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc)（通常被称为 "Spring MVC"）是一个丰富的 "model view controller" Web框架。Spring MVC让你创建特殊的`@Controller`或`@RestController`bean来处理进入的HTTP请求。控制器中的方法通过使用`@RequestMapping`注解被映射到HTTP。

下面的代码显示了一个典型的`@RestController`，它提供JSON数据。

```java
import java.util.List;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/users")
public class MyRestController {

    private final UserRepository userRepository;

    private final CustomerRepository customerRepository;

    public MyRestController(UserRepository userRepository, CustomerRepository customerRepository) {
        this.userRepository = userRepository;
        this.customerRepository = customerRepository;
    }

    @GetMapping("/{user}")
    public User getUser(@PathVariable Long userId) {
        return this.userRepository.findById(userId).get();
    }

    @GetMapping("/{user}/customers")
    public List<Customer> getUserCustomers(@PathVariable Long userId) {
        return this.userRepository.findById(userId).map(this.customerRepository::findByUser).get();
    }

    @DeleteMapping("/{user}")
    public void deleteUser(@PathVariable Long userId) {
        this.userRepository.deleteById(userId);
    }

}
```

Spring MVC是Spring框架核心的一部分，详细的信息可以在[参考文档](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc)中找到。在[spring.io/guides](https://spring.io/guides)上也有一些涵盖Spring MVC的指南。

#### 7.1.1. Spring MVC自动配置

Spring Boot为Spring MVC提供了自动配置功能，对大多数应用程序都很适用。

自动配置在Spring默认的基础上增加了以下功能。

* 包含`ContentNegotiatingViewResolver`和`BeanNameViewResolver` Bean。
* 支持为静态资源提供服务，包括对WebJars的支持（[本文稍后报道](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.static-content)）。
* 自动注册`Converter`、`GenericConverter`和`Formatter` Bean。
* 支持`HttpMessageConverters`([本文稍后报道](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.message-converters))。
* 自动注册 `MessageCodesResolver` ([本文稍后报道](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.message-codes))。
* 静态的`index.html`支持。
* 自动使用 `ConfigurableWebBindingInitializer` bean（[本文稍后介绍](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.binding-initializer)）。

如果你想保留那些Spring Boot MVC定制，并做更多的[MVC定制](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc)（拦截器、格式化器、视图控制器和其他功能），你可以添加你自己的`@Configuration`类，类型为`WebMvcConfigurer`，但不含`@EnableWebMvc`。

如果你想提供`RequestMappingHandlerMapping`、`RequestMappingHandlerAdapter`或`ExceptionHandlerExceptionResolver`的自定义实例，并且仍然保持Spring Boot MVC的自定义，你可以声明一个`WebMvcRegistrations`类型的bean，用它来提供这些组件的自定义实例。

如果你想完全控制Spring MVC，你可以添加你自己的`@Configuration`注释`@EnableWebMvc`，或者添加你自己的`@Configuration`注释`DelegatingWebMvcConfiguration`，如`@EnableWebMvc`的Javadoc中所述。

> Spring MVC使用不同的`ConversionService`来转换`application.properties`或`application.yaml`文件中的值。这意味着`Period`、`Duration`和`DataSize`转换器不可用，`@DurationUnit`和`@DataSizeUnit`注释将被忽略。
>
> 如果你想定制Spring MVC使用的 `ConversionService`，你可以提供一个带有 `addFormatters` 方法的 `WebMvcConfigurer` bean。从这个方法中，你可以注册任何你喜欢的转换器，或者你可以委托给`ApplicationConversionService`上的静态方法。

#### 7.1.2. HttpMessageConverters

Spring MVC使用`HttpMessageConverter`接口来转换HTTP请求和响应。合理的默认值是开箱即用的。例如，对象可以自动转换为JSON（通过使用Jackson库）或XML（通过使用Jackson XML扩展，如果可用的话，或通过使用JAXB，如果Jackson XML扩展不可用）。默认情况下，字符串是以`UTF-8`编码的。

如果你需要添加或定制转换器，你可以使用Spring Boot的`HttpMessageConverters`类，如以下列表所示。

```java
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;

@Configuration(proxyBeanMethods = false)
public class MyHttpMessageConvertersConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = new AdditionalHttpMessageConverter();
        HttpMessageConverter<?> another = new AnotherHttpMessageConverter();
        return new HttpMessageConverters(additional, another);
    }

}
```

任何存在于上下文中的`HttpMessageConverter`bean都会被添加到转换器的列表中。你也可以用同样的方式覆盖默认的转换器。

#### 7.1.3. 自定义 JSON Serializers 和 Deserializers

如果你使用Jackson来序列化和反序列化JSON数据，你可能想编写自己的`JsonSerializer`和`JsonDeserializer`类。自定义序列化器通常是[通过模块与Jackson注册](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers)，但Spring Boot提供了一个替代性的`@JsonComponent`注解，使直接注册Spring Beans更容易。

你可以直接在`JsonSerializer`、`JsonDeserializer`或`KeyDeserializer`实现上使用`@JsonComponent`注解。你也可以在包含序列化器/反序列化器作为内层类的类上使用它，如下面的例子所示。

```java
import java.io.IOException;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.ObjectCodec;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import org.springframework.boot.jackson.JsonComponent;

@JsonComponent
public class MyJsonComponent {

    public static class Serializer extends JsonSerializer<MyObject> {

        @Override
        public void serialize(MyObject value, JsonGenerator jgen, SerializerProvider serializers) throws IOException {
            jgen.writeStringField("name", value.getName());
            jgen.writeNumberField("age", value.getAge());
        }

    }

    public static class Deserializer extends JsonDeserializer<MyObject> {

        @Override
        public MyObject deserialize(JsonParser jsonParser, DeserializationContext ctxt)
                throws IOException, JsonProcessingException {
            ObjectCodec codec = jsonParser.getCodec();
            JsonNode tree = codec.readTree(jsonParser);
            String name = tree.get("name").textValue();
            int age = tree.get("age").intValue();
            return new MyObject(name, age);
        }

    }

}
```

`ApplicationContext`中的所有`@JsonComponent` Bean 都会自动向Jackson注册。因为`@JsonComponent`是用`@Component`元注释的，所以通常的组件扫描规则适用。

Spring Boot还提供了[`JsonObjectSerializer`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java)和[`JsonObjectDeserializer`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java)基类，在序列化对象时提供了标准Jackson版本的有用替代品。详情见Javadoc中的[`JsonObjectSerializer`](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/jackson/JsonObjectSerializer.html)和[`JsonObjectDeserializer`](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/jackson/JsonObjectDeserializer.html)。

上面的例子可以改写为使用`JsonObjectSerializer`/`JsonObjectDeserializer`，如下所示。

```java
import java.io.IOException;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.ObjectCodec;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.SerializerProvider;

import org.springframework.boot.jackson.JsonComponent;
import org.springframework.boot.jackson.JsonObjectDeserializer;
import org.springframework.boot.jackson.JsonObjectSerializer;

@JsonComponent
public class MyJsonComponent {

    public static class Serializer extends JsonObjectSerializer<MyObject> {

        @Override
        protected void serializeObject(MyObject value, JsonGenerator jgen, SerializerProvider provider)
                throws IOException {
            jgen.writeStringField("name", value.getName());
            jgen.writeNumberField("age", value.getAge());
        }

    }

    public static class Deserializer extends JsonObjectDeserializer<MyObject> {

        @Override
        protected MyObject deserializeObject(JsonParser jsonParser, DeserializationContext context, ObjectCodec codec,
                JsonNode tree) throws IOException {
            String name = nullSafeValue(tree.get("name"), String.class);
            int age = nullSafeValue(tree.get("age"), Integer.class);
            return new MyObject(name, age);
        }

    }

}
```

#### 7.1.4. MessageCodesResolver

Spring MVC有一个生成错误代码的策略，用于从绑定错误中渲染错误信息。`MessageCodesResolver`。如果你设置了`spring.mvc.message-codes-resolver-format`属性`PREFIX_ERROR_CODE`或`POSTFIX_ERROR_CODE`，Spring Boot就会为你创建一个（见[`DefaultMessageCodesResolver.Format`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html)中的枚举)。

#### 7.1.5. 静态内容

默认情况下，Spring Boot从classpath中的`/static`（或`/public`或`/resources`或`/META-INF/resources`）目录或`ServletContext`的根中提供静态内容。它使用了Spring MVC中的`ResourceHttpRequestHandler`，因此你可以通过添加你自己的`WebMvcConfigurer`并重写`addResourceHandlers`方法来修改该行为。

在独立的Web应用中，容器中的默认servlet也被启用，并作为一个fallback，如果Spring决定不处理它，就从`ServletContext`的根部提供内容。大多数时候，这种情况不会发生（除非你修改默认的MVC配置），因为Spring总是可以通过`DispatcherServlet`来处理请求。

默认情况下，资源被映射到`/**`上，但你可以通过`spring.mvc.static-path-pattern`属性来调整。例如，将所有资源重新定位到`/resources/**`，可以通过以下方式实现。

```yaml
spring:
  mvc:
    static-path-pattern: "/resources/**"
```

你也可以通过使用`spring.web.resources.static-locations`属性来定制静态资源的位置（用目录位置的列表代替默认值）。根Servlet上下文路径，`"/"`，也被自动添加为一个位置。

除了前面提到的 "标准 "静态资源位置外，还为[Webjars内容](https://www.webjars.org/)提供了一个特殊情况。任何路径为`/webjars/**`的资源，如果是以Webjars格式打包的，则从jar文件中提供。

> 如果你的应用程序被打包成jar，请不要使用`src/main/webapp`目录。虽然这个目录是一个通用的标准，但它只适用于war打包，如果你生成一个jar，它就会被大多数构建工具默默地忽略。

Spring Boot还支持Spring MVC提供的高级资源处理功能，允许使用的情况包括破坏缓存的静态资源或为Webjars使用版本无关的URL。

要对Webjars使用版本不可知的URL，请添加`webjars-locator-core`依赖。然后声明你的Webjar。以jQuery为例，添加`"/webjars/jquery/jquery.min.js "的结果是`"/webjars/jquery/x.y.z/jquery.min.js"，其中`x.y.z`是Webjar的版本。

> 如果你使用JBoss，你需要声明`webjars-locator-jboss-vfs`依赖关系而不是`webjars-locator-core`。否则，所有的Webjars都会解析为`404`。

要使用缓存破坏，以下配置为所有静态资源配置了一个缓存破坏解决方案，有效地在URL中添加了一个内容哈希，如`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`，。

```yaml
spring:
  web:
    resources:
      chain:
        strategy:
          content:
            enabled: true
            paths: "/**"
```

> 由于Thymeleaf和FreeMarker自动配置了`ResourceUrlEncodingFilter`，资源的链接在运行时被重写在模板中。在使用JSP的时候，你应该手动声明这个过滤器。其他模板引擎目前没有自动支持，但可以通过自定义template macros/helpers和使用[`ResourceUrlProvider`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/web/servlet/resource/ResourceUrlProvider.html)。

当用例如JavaScript模块加载器动态加载资源时，重命名文件不是一个选项。这就是为什么也支持其他策略，并且可以组合使用。一个 `fixed` 策略在URL中添加一个静态的版本字符串，而不改变文件名，如下面的例子所示。

```yaml
spring:
  web:
    resources:
      chain:
        strategy:
          content:
            enabled: true
            paths: "/**"
          fixed:
            enabled: true
            paths: "/js/lib/"
            version: "v12"

```

通过这种配置，位于`"/js/lib/"`下的JavaScript模块使用固定的版本策略（`"/v12/js/lib/mymodule.js"`），而其他资源仍然使用内容策略（`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`）。

更多支持的选项见[`ResourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java)。

> 在专门的[博文](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources)和Spring Framework的[参考文档](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-config-static-resources)中已经对这一特性进行了详尽的描述。

#### 7.1.6. Welcome Page

Spring Boot同时支持静态和模板化的欢迎页面。它首先在配置的静态内容位置寻找一个`index.html`文件。如果没有找到，它就会寻找`index`模板。如果找到了其中之一，它就会自动作为应用程序的欢迎页面使用。

#### 7.1.7. 路径匹配和内容协商

Spring MVC可以通过查看请求路径并将其与应用程序中定义的映射（例如，控制器方法上的`@GetMapping`注解）相匹配，将传入的HTTP请求映射到处理程序。

Spring Boot选择默认禁用后缀模式匹配，这意味着像 `GET /projects/spring-boot.json`这样的请求不会被匹配到`@GetMapping("/projects/spring-boot")`映射。这被认为是一个[Spring MVC应用的最佳实践](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-ann-requestmapping-suffix-pattern-match)。这个功能在过去主要是针对HTTP客户端，他们没有发送正确的 `Accept` 请求头；我们需要确保向客户端发送正确的内容类型。现在，内容协商（Content Negotiation）要可靠得多。

还有其他方法来处理那些没有持续发送正确 `Accept` 请求头的HTTP客户端。我们可以不使用后缀匹配，而是使用一个查询参数来确保像`"GET /projects/spring-boot?format=json"`这样的请求会被映射到`@GetMapping("/projects/spring-boot")`。

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: true
```

或者如果你喜欢使用一个不同的参数名称。

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: true
      parameter-name: "myparam"
```

大多数标准媒体类型都是开箱即用的，但你也可以定义新的媒体类型。

```yaml
spring:
  mvc:
    contentnegotiation:
      media-types:
        markdown: "text/markdown"
```

后缀模式匹配已被废弃，并将在未来的版本中被删除。如果你了解这些注意事项，并且仍然希望你的应用程序使用后缀模式匹配，则需要进行以下配置。

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-path-extension: true
    pathmatch:
      use-suffix-pattern: true
```

另外，与其开放所有后缀模式，不如只支持已注册的后缀模式更安全。

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-path-extension: true
    pathmatch:
      use-registered-suffix-pattern: true
```

从Spring Framework 5.3开始，Spring MVC支持几种实现策略来匹配请求路径和控制器处理程序。它以前只支持 `AntPathMatcher` 策略，但现在也提供 `PathPatternParser`。Spring Boot现在提供了一个配置属性，可以选择并加入新的策略。

```yaml
spring:
  mvc:
    pathmatch:
      matching-strategy: "path-pattern-parser"
```

关于你为什么要考虑这个新实施的更多细节，请查看[专用博文](https://spring.io/blog/2020/06/30/url-matching-with-pathpattern-in-spring-mvc)。

> `PathPatternParser`是一个优化的实现，但限制了[一些路径模式变体](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-ann-requestmapping-uri-templates)的使用，并且与后缀模式匹配（`spring.mvc.pathmatch.use-suffix-pattern`，`spring.mvc.pathmatch.use-registered-suffix-pattern`）或将`DispatcherServlet`与Servlet前缀映射（`spring.mvc.Servlet.path`）不兼容。

#### 7.1.8. ConfigurableWebBindingInitializer

Spring MVC使用`WebBindingInitializer`来为特定的请求初始化`WebDataBinder`。如果你创建了自己的`ConfigurableWebBindingInitializer` `@Bean`，Spring Boot会自动配置Spring MVC来使用它。

#### 7.1.9. 模板引擎

除了REST网络服务，你还可以使用Spring MVC来提供动态HTML内容。Spring MVC支持各种模板技术，包括Thymeleaf、FreeMarker和JSP。此外，许多其他模板引擎也包括它们自己的Spring MVC集成。

Spring Boot包括对以下模板引擎的自动配置支持。

* [FreeMarker](https://freemarker.apache.org/docs/)
* [Groovy](https://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)
* [Thymeleaf](https://www.thymeleaf.org/)
* [Mustache](https://mustache.github.io/)

> 如果可能的话，应该避免使用JSP。在与嵌入式Servlet容器一起使用它们时，有几个[已知的限制](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.embedded-container.jsp-limitations)。

当你使用这些默认配置的模板引擎之一时，你的模板会自动从`src/main/resources/templates`中获取。

> 根据你运行应用程序的方式，你的IDE可能会对classpath进行不同的排序。在IDE中从其主方法中运行你的应用程序，与你通过使用Maven或Gradle或从其打包的jar中运行你的应用程序时的排序不同。这可能导致Spring Boot无法找到预期的模板。如果你有这个问题，你可以在IDE中重新排序classpath，把模块的类和资源放在前面。

#### 7.1.10. Error处理

默认情况下，Spring Boot提供了一个`/error`映射，以合理的方式处理所有错误，它被注册为servlet容器中的 `global` 错误页面。对于机器客户端，它会产生一个JSON响应，包含错误的细节、HTTP状态和异常消息。对于浏览器客户端，有一个 `whitelabel` 错误视图，以HTML格式显示相同的数据（要定制它，请添加一个解析为 `error` 的`View`）。

如果你想定制默认的错误处理行为，有一些`server.error`属性可以设置。参见附录中的[Server Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.server)部分。

要完全替换默认行为，你可以实现`ErrorController`并注册该类型的Bean定义，或者添加`ErrorAttributes`类型的Bean来使用现有机制但替换内容。

> `BasicErrorController`可以作为自定义`ErrorController`的基类。如果你想为一个新的内容类型添加一个处理程序（默认是专门处理`text/html`，并为其他所有内容提供一个fallback ），这特别有用。要做到这一点，请扩展`BasicErrorController`，添加一个带有`@RequestMapping`的公共方法，该方法具有`produces`属性，并创建一个新类型的bean。

你也可以定义一个带有`@ControllerAdvice`注释的类，为特定的controller/exception 型定制返回的JSON文档，如以下例子所示。

```java
import javax.servlet.RequestDispatcher;
import javax.servlet.http.HttpServletRequest;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

@ControllerAdvice(basePackageClasses = SomeController.class)
public class MyControllerAdvice extends ResponseEntityExceptionHandler {

    @ResponseBody
    @ExceptionHandler(MyException.class)
    public ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new MyErrorBody(status.value(), ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer code = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        HttpStatus status = HttpStatus.resolve(code);
        return (status != null) ? status : HttpStatus.INTERNAL_SERVER_ERROR;
    }

}
```

在前面的例子中，如果`YourException`是由定义在与`SomeController`相同包中的控制器抛出的，那么就会使用`CustomErrorType`POJO的JSON表示，而不是`ErrorAttributes`表示。

在某些情况下，在控制器级别处理的错误不会被[metrics infrastructure](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported.spring-mvc)记录。应用程序可以通过将处理的异常设置为请求属性来确保这些异常被请求度量所记录。

```java
import javax.servlet.http.HttpServletRequest;

import org.springframework.boot.web.servlet.error.ErrorAttributes;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;

@Controller
public class MyController {

    @ExceptionHandler(CustomException.class)
    String handleCustomException(HttpServletRequest request, CustomException ex) {
        request.setAttribute(ErrorAttributes.ERROR_ATTRIBUTE, ex);
        return "errorView";
    }

}
```

##### Custom Error Pages

如果你想为一个给定的状态代码显示一个自定义的HTML错误页面，你可以在`/error`目录下添加一个文件。错误页面可以是静态HTML（即添加在任何一个静态资源目录下），也可以通过使用模板建立。文件的名称应该是准确的状态代码或一系列的掩码。

例如，要把`404`映射到一个静态HTML文件，你的目录结构将如下。

```text
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

通过使用FreeMarker模板来映射所有的`5xx`错误，你的目录结构如下。

```text
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftlh
             +- <other templates>
```

对于更复杂的映射，你也可以添加实现`ErrorViewResolver`接口的Bean，如下例所示。

```java
public class MyErrorViewResolver implements ErrorViewResolver {

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        if (status == HttpStatus.INSUFFICIENT_STORAGE) {
            // We could add custom model values here
            new ModelAndView("myview");
        }
        return null;
    }

}
```

你也可以使用常规的Spring MVC特性，如[`@ExceptionHandler`方法](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-exceptionhandlers)和[`@ControllerAdvice`](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-ann-controller-advice)。`ErrorController`会拾取任何未处理的异常。

##### 在Spring MVC之外映射错误页面

对于不使用Spring MVC的应用程序，你可以使用`ErrorPageRegistrar`接口来直接注册`ErrorPages`。这种抽象直接与底层的嵌入式servlet容器一起工作，即使你没有Spring MVC的`DispatcherServlet`也能工作。

```java
import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.server.ErrorPageRegistrar;
import org.springframework.boot.web.server.ErrorPageRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;

@Configuration(proxyBeanMethods = false)
public class MyErrorPagesConfiguration {

    @Bean
    public ErrorPageRegistrar errorPageRegistrar() {
        return this::registerErrorPages;
    }

    private void registerErrorPages(ErrorPageRegistry registry) {
        registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }

}
```

> 如果你用一个最终由 `Filter` 处理的路径注册 `ErrorPage`（这在一些非Spring的Web框架中很常见，比如Jersey和Wicket），那么 `Filter` 必须明确注册为 `ERROR` dispatcher，如下例所示。

```java
import java.util.EnumSet;

import javax.servlet.DispatcherType;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyFilterConfiguration {

    @Bean
    public FilterRegistrationBean<MyFilter> myFilter() {
        FilterRegistrationBean<MyFilter> registration = new FilterRegistrationBean<>(new MyFilter());
        // ...
        registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
        return registration;
    }

}
```

请注意，默认的`FilterRegistrationBean`不包括`ERROR` dispatcher 类型。

##### War部署中的错误处理

当部署到servlet容器时，Spring Boot使用其错误页面过滤器，将具有错误状态的请求转发到适当的错误页面。这是必要的，因为Servlet规范并没有提供注册错误页面的API。根据你部署war文件的容器和你的应用程序使用的技术，可能需要一些额外的配置。

错误页面过滤器只能在响应尚未提交的情况下将请求转发到正确的错误页面。默认情况下，WebSphere Application Server 8.0 及更高版本会在成功完成 servlet 的服务方法后提交响应。你应该通过将`com.ibm.ws.webcontainer.invokeFlushAfterService`设置为`false`来禁用这种行为。

如果你使用Spring Security并希望在错误页面中访问本金，你必须配置Spring Security的过滤器，使其在error dispatches中被调用。为此，将`spring.security.filter.dispatcher-types`属性设置为`async, error, forward, request`。

#### 7.1.11. Spring HATEOAS

如果你开发的RESTful API使用了hypermedia，Spring Boot为Spring HATEOAS提供了自动配置，对大多数应用都很适用。自动配置取代了使用`@EnableHypermediaSupport` 的需要，并注册了一些Bean，以方便构建基于超媒体的应用程序，包括 `LinkDiscoverers`（用于客户端支持）和 `ObjectMapper`，配置为将响应正确整合为所需的表示方式。通过设置各种 `spring.jackson.*`属性来定制 `ObjectMapper`，如果有的话，可以通过 `Jackson2ObjectMapperBuilder` bean来实现。

你可以通过使用`@EnableHypermediaSupport`来控制Spring HATEOAS的配置。请注意，这样做会使前面描述的`ObjectMapper`定制失效。

> `spring-boot-starter-hateoas`是针对Spring MVC的，不应该与Spring WebFlux结合。为了在Spring WebFlux中使用Spring HATEOAS，你可以在添加`org.springframework.hateoas:spring-hateoas`的同时直接依赖`spring-boot-starter-webflux`。

#### 7.1.12. CORS 支持

[跨源资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS)是一个由[大多数浏览器](https://www.w3.org/TR/cors/)实现的[W3C规范](https://caniuse.com/#feat=cors)，它让你以灵活的方式指定什么样的跨域请求是被授权的。，而不是使用一些不太安全、不太强大的方法，如IFRAME或JSONP。

从4.2版本开始，Spring MVC[支持CORS](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-cors)。在你的Spring Boot应用程序中使用带有[`@CrossOrigin`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)注释的[Controller方法CORS配置](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-cors-controller)不需要任何特定的配置。[全局CORS配置](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc-cors-global)可以通过注册一个`WebMvcConfigurer`bean与定制的`addCorsMappings(CorsRegistry)`方法来定义，如以下例子所示。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration(proxyBeanMethods = false)
public class MyCorsConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {

            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }

        };
    }

}
```

### 7.2. Spring WebFlux框架

Spring WebFlux是Spring Framework 5.0中引入的新的响应式Web框架。与Spring MVC不同，它不需要Servlet API，是完全异步和非阻塞的，并通过[Reactor项目](https://projectreactor.io/)实现了[Reactive Streams](https://www.reactive-streams.org/)规范。

Spring WebFlux有两种风格：功能性的和基于注解的。基于注解的版本与Spring MVC模型非常接近，如下面的例子所示。

```java
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/users")
public class MyRestController {

    private final UserRepository userRepository;

    private final CustomerRepository customerRepository;

    public MyRestController(UserRepository userRepository, CustomerRepository customerRepository) {
        this.userRepository = userRepository;
        this.customerRepository = customerRepository;
    }

    @GetMapping("/{user}")
    public Mono<User> getUser(@PathVariable Long userId) {
        return this.userRepository.findById(userId);
    }

    @GetMapping("/{user}/customers")
    public Flux<Customer> getUserCustomers(@PathVariable Long userId) {
        return this.userRepository.findById(userId).flatMapMany(this.customerRepository::findByUser);
    }

    @DeleteMapping("/{user}")
    public void deleteUser(@PathVariable Long userId) {
        this.userRepository.deleteById(userId);
    }

}
```

"WebFlux.fn"，功能变体，将路由配置与请求的实际处理分开，如下例所示。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.RequestPredicate;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerResponse;

import static org.springframework.web.reactive.function.server.RequestPredicates.DELETE;
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RequestPredicates.accept;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;

@Configuration(proxyBeanMethods = false)
public class MyRoutingConfiguration {

    private static final RequestPredicate ACCEPT_JSON = accept(MediaType.APPLICATION_JSON);

    @Bean
    public RouterFunction<ServerResponse> monoRouterFunction(MyUserHandler userHandler) {
        return route(
                GET("/{user}").and(ACCEPT_JSON), userHandler::getUser).andRoute(
                GET("/{user}/customers").and(ACCEPT_JSON), userHandler::getUserCustomers).andRoute(
                DELETE("/{user}").and(ACCEPT_JSON), userHandler::deleteUser);
    }

}
```

```java
import reactor.core.publisher.Mono;

import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;

@Component
public class MyUserHandler {

    public Mono<ServerResponse> getUser(ServerRequest request) {
        ...
    }

    public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
        ...
    }

    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        ...
    }

}
```

WebFlux是Spring框架的一部分，详细信息可在其[参考文档](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web-reactive.html#webflux-fn)中找到。

> 你可以定义任意多的`RouterFunction` Bean，以使路由器的定义模块化。如果你需要应用一个优先级，Bean可以被排序。

要开始使用，请将`spring-boot-starter-webflux`模块添加到你的应用程序。

> 在你的应用程序中同时添加`spring-boot-starter-web`和`spring-boot-starter-webflux`模块会导致Spring Boot自动配置Spring MVC，而不是WebFlux。选择这种行为是因为许多Spring开发者将`spring-boot-starter-webflux`添加到他们的Spring MVC应用中，以使用响应式`WebClient`。你仍然可以通过将选择的应用类型设置为`SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)`来执行你的选择。

#### 7.2.1. Spring WebFlux Auto-configuration

Spring Boot为Spring WebFlux提供了自动配置，对大多数应用程序都很适用。

自动配置在Spring的默认值基础上增加了以下功能。

* 为 `HttpMessageReader` 和 `HttpMessageWriter` 实例配置编解码器（[本文后面的描述](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-webflux.httpcodecs)）。
* 支持为静态资源提供服务，包括对WebJars的支持（[本文稍后](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.static-content)描述）。

如果你想保留Spring Boot的WebFlux特性，并想添加额外的[WebFlux配置](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web-reactive.html#webflux-config)，你可以添加自己的`@Configuration`类，类型为`WebFluxConfigurer`，但不包括`@EnableWebFlux`。

如果你想完全控制Spring WebFlux，你可以添加你自己的`@Configuration`，并注有`@EnableWebFlux`。

#### 7.2.2. 带有HttpMessageReaders和HttpMessageWriters的HTTP编解码器

Spring WebFlux使用`HttpMessageReader`和`HttpMessageWriter`接口来转换HTTP请求和响应。它们是用`CodecConfigurer`配置的，通过查看classpath中可用的库，具有合理的默认值。

Spring Boot为编解码器提供专门的配置属性，`spring.codec.*`。它还通过使用`CodecCustomizer`实例来应用进一步的定制。例如，`spring.jackson.*`配置键被应用于Jackson编解码器。

如果你需要添加或定制编解码器，你可以创建一个自定义的`CodecCustomizer`组件，如下面的例子所示。

```java
import org.springframework.boot.web.codec.CodecCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.codec.ServerSentEventHttpMessageReader;

@Configuration(proxyBeanMethods = false)
public class MyCodecsConfiguration {

    @Bean
    public CodecCustomizer myCodecCustomizer() {
        return (configurer) -> {
            configurer.registerDefaults(false);
            configurer.customCodecs().register(new ServerSentEventHttpMessageReader());
            // ...
        };
    }

}
```

你也可以利用[Boot的自定义JSON序列化器和反序列化器](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.json)。

#### 7.2.3. 静态内容

默认情况下，Spring Boot从classpath中名为`/static`（或`/public`或`/resources`或`/META-INF/resources`）的目录提供静态内容。它使用Spring WebFlux的`ResourceWebHandler`，所以你可以通过添加你自己的`WebFluxConfigurer'和覆盖`addResourceHandlers`方法来修改该行为。

默认情况下，资源被映射在`/**`上，但你可以通过设置`spring.webflux.static-path-pattern`属性来调整它。例如，将所有资源重新定位到`/resources/**`，可以通过以下方式实现。

```yaml
spring:
  webflux:
    static-path-pattern: "/resources/**"
```

你也可以通过使用`spring.web.resources.static-locations`来定制静态资源位置。这样做将默认值替换为目录位置的列表。如果你这样做，默认的欢迎页面检测会切换到你的自定义位置。因此，如果在启动时你的任何位置有`index.html`，它就是应用程序的主页。

除了前面列出的 "标准" 静态资源位置外，对[Webjars内容](https://www.webjars.org/)也有特殊情况。任何路径为`/webjars/**`的资源，如果是以Webjars格式打包的，则从jar文件中提供。

> Spring WebFlux应用程序并不严格依赖Servlet API，所以它们不能以war文件的形式部署，也不使用`src/main/webapp`目录。

#### 7.2.4. 欢迎页

Spring Boot同时支持静态和模板化的欢迎页面。它首先在配置的静态内容位置寻找一个`index.html`文件。如果没有找到，它就会寻找`index`模板。如果找到了其中之一，它就会自动作为应用程序的欢迎页面使用。

#### 7.2.5. 模板引擎

除了REST Web服务，你还可以使用Spring WebFlux来提供动态HTML内容。Spring WebFlux支持各种模板技术，包括Thymeleaf、FreeMarker和Mustache。

Spring Boot包括对以下模板引擎的自动配置支持。

* [FreeMarker](https://freemarker.apache.org/docs/)
* [Thymeleaf](https://www.thymeleaf.org/)
* [Mustache](https://mustache.github.io/)

当你使用这些默认配置的模板引擎之一时，你的模板会自动从`src/main/resources/templates`中获取。

#### 7.2.6 Error处理

Spring Boot提供了一个`WebExceptionHandler`，以合理的方式处理所有错误。它在处理顺序中的位置紧靠WebFlux提供的处理程序，后者被认为是最后一个。对于机器客户端，它产生一个JSON响应，包含错误的细节、HTTP状态和异常消息。对于浏览器客户端，有一个 `whitelabel` 错误处理程序，以HTML格式渲染相同的数据。你也可以提供你自己的HTML模板来显示错误（见[下一节](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-webflux.error-handling.error-pages)）。

定制这一功能的第一步通常是使用现有的机制，但要替换或增强错误内容。为此，你可以添加一个`ErrorAttributes`类型的bean。

为了改变错误处理行为，你可以实现`ErrorWebExceptionHandler`并注册该类型的bean定义。因为`ErrorWebExceptionHandler`是相当低级的，Spring Boot还提供了一个方便的`AbstractErrorWebExceptionHandler`，让你以WebFlux的功能方式处理错误，如下面的例子所示。

```java
import reactor.core.publisher.Mono;

import org.springframework.boot.autoconfigure.web.WebProperties.Resources;
import org.springframework.boot.autoconfigure.web.reactive.error.AbstractErrorWebExceptionHandler;
import org.springframework.boot.web.reactive.error.ErrorAttributes;
import org.springframework.context.ApplicationContext;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.reactive.function.server.ServerResponse.BodyBuilder;

@Component
public class MyErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

    public MyErrorWebExceptionHandler(ErrorAttributes errorAttributes, Resources resources,
            ApplicationContext applicationContext) {
        super(errorAttributes, resources, applicationContext);
    }

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
        return RouterFunctions.route(this::acceptsXml, this::handleErrorAsXml);
    }

    private boolean acceptsXml(ServerRequest request) {
        return request.headers().accept().contains(MediaType.APPLICATION_XML);
    }

    public Mono<ServerResponse> handleErrorAsXml(ServerRequest request) {
        BodyBuilder builder = ServerResponse.status(HttpStatus.INTERNAL_SERVER_ERROR);
        // ... additional builder calls
        return builder.build();
    }

}
```

为了更全面地了解情况，你也可以直接子类化`DefaultErrorWebExceptionHandler`并重写特定的方法。

在某些情况下，在控制器或处理函数层面上处理的错误不会被[metrics infrastructure](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported.spring-webflux)记录。应用程序可以通过将处理的异常设置为请求属性来确保这些异常被请求度量所记录。

```java
import org.springframework.boot.web.reactive.error.ErrorAttributes;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.reactive.result.view.Rendering;
import org.springframework.web.server.ServerWebExchange;

@Controller
public class MyExceptionHandlingController {

    @GetMapping("/profile")
    public Rendering userProfile() {
        // ...
        throw new IllegalStateException();
    }

    @ExceptionHandler(IllegalStateException.class)
    public Rendering handleIllegalState(ServerWebExchange exchange, IllegalStateException exc) {
        exchange.getAttributes().putIfAbsent(ErrorAttributes.ERROR_ATTRIBUTE, exc);
        return Rendering.view("errorView").modelAttribute("message", exc.getMessage()).build();
    }

}
```

##### 自定义错误页面

如果你想为一个给定的状态代码显示一个自定义的HTML错误页面，你可以在`/error`目录下添加一个文件。错误页面可以是静态HTML（即添加在任何一个静态资源目录下）或用模板构建。文件的名字应该是准确的状态代码或一系列的掩码。

例如，要把`404`映射到一个静态HTML文件，你的目录结构如下。

```text
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>

```

通过使用Mustache模板来映射所有`5xx`错误，你的目录结构如下。

```text
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.mustache
             +- <other templates>

```

#### 7.2.7. Web Filters

Spring WebFlux提供了一个`WebFilter`接口，可以实现它来过滤HTTP请求-响应交换。在应用程序上下文中发现的`WebFilter` Bean将被自动用于过滤每个交换。

如果过滤器的顺序很重要，它们可以实现`Ordered`或用`@Order`来注释。Spring Boot的自动配置可以为你配置网络过滤器。当它这样做时，将使用下表中显示的顺序。

|Web Filter|Order|
| --- | --- |
|`MetricsWebFilter`|`Ordered.HIGHEST_PRECEDENCE + 1`|
|`WebFilterChainProxy` (Spring Security)|`-100`|
|`HttpTraceWebFilter`|`Ordered.LOWEST_PRECEDENCE - 10`|

### 7.3. JAX-RS and Jersey

如果你喜欢REST端点的JAX-RS编程模型，你可以使用其中一个可用的实现来代替Spring MVC。[Jersey](https://jersey.github.io/)和[Apache CXF](https://cxf.apache.org/)开箱即用，效果相当好。CXF要求你将其 `Servlet` 或 `Filter` 注册为应用上下文中的 `@Bean`。Jersey有一些原生的Spring支持，所以我们也在Spring Boot中为它提供了自动配置支持，同时还有一个启动器。

要开始使用Jersey，请将`spring-boot-starter-jersey`作为一个依赖项，然后你需要一个`ResourceConfig`类型的`@Bean`，在其中注册所有的端点，如下例所示。

```java
import org.glassfish.jersey.server.ResourceConfig;

import org.springframework.stereotype.Component;

@Component
public class MyJerseyConfig extends ResourceConfig {

    public MyJerseyConfig() {
        register(MyEndpoint.class);
    }

}
```

> Jersey对扫描可执行压缩文件的支持是相当有限的。例如，它不能扫描在[完全可执行的jar文件](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.installing)中发现的包中的端点，或者在运行可执行的war文件时在`WEB-INF/classes`中发现的端点。为了避免这种限制，不应该使用`packages`方法，而应该使用`register`方法单独注册端点，如前面的例子所示。

对于更高级的定制，你也可以注册任意数量的实现`ResourceConfigCustomizer`的bean。

所有注册的端点都应该是带有HTTP资源注释的`@Components`（`@GET`和其他），如下面的例子所示。

```java
import javax.ws.rs.GET;
import javax.ws.rs.Path;

import org.springframework.stereotype.Component;

@Component
@Path("/hello")
public class MyEndpoint {

    @GET
    public String message() {
        return "Hello";
    }

}
```

由于`Endpoint`是Spring的`@Component`，它的生命周期由Spring管理，你可以使用`@Autowired`注解来注入依赖关系，使用`@Value`注解来注入外部配置。默认情况下，Jersey servlet被注册并映射到`/*`。你可以通过添加`@ApplicationPath`到你的`ResourceConfig`来改变映射。

默认情况下，Jersey被设置为Servlet在一个`@Bean`类型的`ServletRegistrationBean`中，名为`jerseyServletRegistration`。默认情况下，servlet被懒惰地初始化，但你可以通过设置`spring.jersey.servlet.load-on-startup`来定制这一行为。你可以通过创建一个你自己的同名Bean来禁用或覆盖该Bean。你也可以通过设置`spring.jersey.type=filter`来使用过滤器而不是servlet（在这种情况下，要替换或覆盖的`@Bean`是`jerseyFilterRegistration`）。过滤器有一个`@Order`，你可以用`spring.jersey.filter.order`来设置。当使用Jersey作为过滤器时，必须有一个Servlet来处理任何没有被Jersey拦截的请求。如果你的应用程序不包含这样的Servlet，你可能想通过设置`server.servlet.register-default-servlet`为`true`来启用默认Servlet。Servlet和过滤器的注册都可以通过使用`spring.jersey.init.*`来指定属性的映射来获得初始参数。

### 7.4. 嵌入式Servlet容器支持

Spring Boot包括对嵌入式[Tomcat](https://tomcat.apache.org/)、[Jetty](https://www.eclipse.org/jetty/)和[Undertow](https://github.com/undertow-io/undertow) 服务器的支持。大多数开发者使用适当的 "Starter"来获得一个完全配置的实例。默认情况下，嵌入式服务器在端口`8080`上监听HTTP请求。

#### 7.4.1. Servlets, Filters, and listeners

当使用嵌入式Servlet容器时，你可以从Servlet规范中注册Servlet、过滤器和所有监听器（如`HttpSessionListener`），可以使用Spring Bean或通过扫描Servlet组件。

##### 把 Servlets, Filters, and Listeners 注册为 Spring Bean

任何属于Spring Bean的`Servlet`、`Filter`或Servlet`*Listener`实例都会在嵌入式容器中注册。如果你想在配置过程中引用`application.properties`中的一个值，这可能特别方便。

默认情况下，如果上下文只包含一个Servlet，它会被映射到`/`。在有多个Servlet Bean的情况下，bean名称被用作路径前缀。过滤器映射到`/*`。

如果基于惯例的映射不够灵活，你可以使用`ServletRegistrationBean`、`FilterRegistrationBean`和`ServletListenerRegistrationBean`类来完全控制。

通常，让Filter Bean不排序是安全的。如果需要一个特定的顺序，你应该用`@Order`来注解`Filter`或者让它实现`Ordered`。你不能通过给`Filter'的Bean方法加上`@Order'注释来配置其顺序。如果你不能改变`Filter`类来添加`@Order`或实现`Ordered`，你必须为`Filter`定义一个`FilterRegistrationBean`并使用`setOrder(int)`方法设置注册Bean的顺序。避免配置一个以`Ordered.HIGHEST_PRECEDENCE`读取请求正文的Filter，因为它可能违背你的应用程序的字符编码配置。如果一个Servlet过滤器封装了请求，它应该被配置为小于或等于`OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER`的顺序。

> 要查看应用程序中每个 `Filter` 的顺序，请为 "web" [日志组](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging.log-groups)启用调试级别的日志 (`logging.level.web=debug`) 。注册过滤器的细节，包括它们的顺序和URL模式，将在启动时被记录下来。

在注册`Filter`Bean时要小心，因为它们在应用程序生命周期的早期就被初始化了。如果你需要注册一个与其他Bean交互的`Filter`，请考虑使用[`DelegatingFilterProxyRegistrationBean`](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/web/servlet/DelegatingFilterProxyRegistrationBean.html)代替。

#### 7.4.2. Servlet Context Initialization

嵌入式Servlet容器不直接执行Servlet 3.0+ `javax.servlet.ServletContainerInitializer`接口或Spring的`org.springframework.web.WebApplicationInitializer`接口。这是一个有意的设计决定，旨在减少设计在战争中运行的第三方库可能破坏Spring Boot应用程序的风险。

如果你需要在Spring Boot应用程序中执行Servlet上下文初始化，你应该注册一个实现`org.springframework.boot.web.servlet.ServletContextInitializer`接口的bean。单一的`onStartup`方法提供了对`ServletContext`的访问，如果有必要，可以很容易地作为现有`WebApplicationInitializer`的适配器。

##### 扫描 Servlets, Filters, listeners

当使用嵌入式容器时，可以通过使用`@ServletComponentScan` 来启用对`@WebServlet`、`@WebFilter` 和`@WebListener` 注释的类的自动注册。

> `@ServletComponentScan`在独立的容器中没有作用，而是使用容器的内置发现机制。

#### 7.4.3. ServletWebServerApplicationContext

Spring Boot使用不同类型的`ApplicationContext`来支持嵌入式Servlet容器。`ServletWebServerApplicationContext`是一种特殊类型的`WebApplicationContext`，它通过搜索单个`ServletWebServerFactory`bean来引导自己。通常一个`TomcatServletWebServerFactory`、`JettyServletWebServerFactory`或`UndertowServletWebServerFactory`已经被自动配置。

> 你通常不需要知道这些实现类。大多数应用程序都是自动配置的，适当的 `ApplicationContext` 和 `ServletWebServerFactory` 是代表你创建的。

#### 7.4.4. 定制嵌入式Servlet容器

普通的servlet容器设置可以通过使用Spring的`Environment`属性来进行配置。通常情况下，你会在`application.properties`或`application.yaml`文件中定义这些属性。

常见的服务器设置包括。

* 网络设置。传入的HTTP请求的监听端口（`server.port`），与`server.address`绑定的接口地址，等等。
* 会话设置。会话是否持久（`server.servlet.session.persistent`），会话超时（`server.servlet.session.timeout`），会话数据的位置（`server.servlet.session.store-dir`），以及会话cookie配置（`server.servlet.session.cookie.*`）。
* 错误管理。错误页面的位置（`server.error.path`）等。
* [SSL](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.webserver.configure-ssl)
* [HTTP compression](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.webserver.enable-response-compression)

Spring Boot尽可能地公开通用设置，但这并不总是可能的。对于这些情况，专门的命名空间提供了针对服务器的定制功能（见`server.tomcat`和`server.undertow`）。例如，[访问日志](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.webserver.configure-access-logs)可以用嵌入式servlet容器的特定功能进行配置。

> 完整的列表请参见[`ServerProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)类。

##### 程序化定制

如果你需要以编程方式配置你的嵌入式Servlet容器，你可以注册一个实现`WebServerFactoryCustomizer`接口的Spring Bean。`WebServerFactoryCustomizer`提供了对`ConfigurableServletWebServerFactory`的访问，其中包括许多自定义设置方法。下面的例子显示了以编程方式设置端口。

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class MyWebServerFactoryCustomizer implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }

}
```

`TomcatServletWebServerFactory`、`JettyServletWebServerFactory`和`UndertowServletWebServerFactory`是`ConfigurableServletWebServerFactory`的专用变体，分别为Tomcat、Jetty和Undertow提供额外的自定义设置方法。下面的例子展示了如何定制`TomcatServletWebServerFactory`，它提供对Tomcat特定配置选项的访问。

```java
import java.time.Duration;

import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.stereotype.Component;

@Component
public class MyTomcatWebServerFactoryCustomizer implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    @Override
    public void customize(TomcatServletWebServerFactory server) {
        server.addConnectorCustomizers((connector) -> connector.setAsyncTimeout(Duration.ofSeconds(20).toMillis()));
    }

}
```

##### 直接定制ConfigurableServletWebServerFactory

对于需要你从`ServletWebServerFactory`扩展的更高级的用例，你可以自己公开这种类型的bean。

我们为许多配置选项提供了设置器。如果你需要做一些更特殊的事情，也提供了几个受保护的方法 "Hook"。详情请参见[source code documentation](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html)。

> 自动配置的定制器仍然应用在你的定制factory上，所以要小心使用这个选项。

#### 7.4.5. JSP的局限性

当运行使用嵌入式 servlet 容器的 Spring Boot 应用程序时（并且被打包为可执行包），对 JSP 的支持存在一些限制。

* 对于Jetty和Tomcat，如果你使用war打包，应该可以工作。一个可执行的war在用`java -jar`启动时可以工作，并且也可以部署到任何标准的容器。当使用可执行jar时，不支持JSP。
* Undertow不支持JSP。
* 创建一个自定义的`error.jsp`页面并不能覆盖[错误处理](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.error-handling)的默认视图。应该使用[自定义错误页面](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.error-handling.error-pages)来代替。

### 7.5. 嵌入式Reactive务器支持

Spring Boot包括对以下嵌入式反应式Web服务器的支持。Reactor Netty、Tomcat、Jetty和Undertow。大多数开发者使用适当的 "Starter"来获得一个完全配置的实例。默认情况下，嵌入式服务器监听8080端口的HTTP请求。

### 7.6. Reactive Server Resources Configuration

在自动配置Reactor Netty或Jetty服务器时，Spring Boot将创建特定的Bean，为服务器实例提供HTTP资源：`ReactorResourceFactory`或`JettyResourceFactory`。

默认情况下，这些资源也将与Reactor Netty和Jetty客户端共享，以获得最佳性能，因为。

* 服务器和客户端使用相同的技术
* 客户端实例是使用Spring Boot自动配置的`WebClient.Builder`bean构建的。

开发人员可以通过提供自定义的`ReactorResourceFactory`或`JettyResourceFactory`bean来覆盖Jetty和Reactor Netty的资源配置 - 这将适用于客户端和服务器。

你可以在[WebClient Runtime部分](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.webclient.runtime)中了解更多关于客户端的资源配置。

## 8. 优雅关机

所有四个嵌入式Web服务器（Jetty、Reactor Netty、Tomcat和Undertow）以及基于响应式和Servlet的Web应用程序都支持优雅关闭。它作为关闭应用程序上下文的一部分发生，并在停止`SmartLifecycle` bean的最早阶段执行。这种停止处理使用一个超时，提供一个宽限期，在此期间，现有的请求将被允许完成，但不允许有新的请求。不允许新请求的确切方式取决于正在使用的网络服务器。Jetty、Reactor Netty和Tomcat将在网络层停止接受请求。Undertow将接受请求，但立即响应服务不可用（503）的回应。

> 使用Tomcat的优雅关机需要Tomcat 9.0.33或更高版本。

要启用优雅关机，请配置`server.shutdown`属性，如下例所示。

```yaml
server:
  shutdown: "graceful"
```

要配置超时时间，请配置`spring.lifecycle.timeout-per-shutdown-phase`属性，如下例所示。

```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: "20s"
```

**如果你的IDE没有发送适当的 `SIGTERM` 信号，使用优雅关机可能无法正常工作。更多细节请参考你的IDE的文档。**

## 9. RSocket

[RSocket](https://rsocket.io/)是一个用于字节流传输的二进制协议。它通过单个连接的异步消息传递实现对称的交互模型。

Spring框架的`spring-messaging`模块为RSocket请求和响应提供了支持，包括在客户端和服务器端。参见Spring框架参考资料中的[RSocket部分](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web-reactive.html#rsocket-spring)以了解更多细节，包括RSocket协议的概述。

### 9.1. RSocket Strategies Auto-configuration

Spring Boot自动配置了一个`RSocketStrategies` bean，它提供了编码和解码RSocket payloads所需的所有基础设施。默认情况下，自动配置将尝试配置以下内容（按顺序）。

1. 使用Jackson的[CBOR](https://cbor.io/)编解码器
2. 使用Jackson的JSON编解码器

`spring-boot-starter-rsocket`启动器提供了这两个依赖项。查看[Jackson支持部分](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.json.jackson)以了解更多关于定制的可能性。

开发者可以通过创建实现 `RSocketStrategiesCustomizer` 接口的bean来定制`RSocketStrategies`组件。请注意，它们的`@Order`很重要，因为它决定了编解码器的顺序。

### 9.2. RSocket server Auto-configuration

Spring Boot提供了RSocket服务器自动配置功能。所需的依赖性由`spring-boot-starter-rsocket`提供。

Spring Boot允许从WebFlux服务器通过WebSocket暴露RSocket，或建立一个独立的RSocket服务器。这取决于应用程序的类型及其配置。

对于WebFlux应用程序（即 `WebApplicationType.REACTIVE` 类型），只有在以下属性相符时，RSocket服务器才会被插入Web服务器。

```yaml
spring:
  rsocket:
    server:
      mapping-path: "/rsocket"
      transport: "websocket"
```

> 将RSocket插入Web服务器只支持Reactor Netty，因为RSocket本身就是用该库构建的。

另外，RSocket TCP或websocket服务器作为一个独立的嵌入式服务器被启动。除了依赖性要求，唯一需要的配置是为该服务器定义一个端口。

```yaml
spring:
  rsocket:
    server:
      port: 9898
```

### 9.3. Spring Messaging RSocket支持

Spring Boot将为RSocket自动配置Spring Messaging基础设施。

这意味着Spring Boot将创建一个`RSocketMessageHandler` bean，以处理对你的应用程序的RSocket请求。

#### 9.4. 用RSocketRequester调用RSocket服务

一旦服务器和客户端之间建立了`RSocket`通道，任何一方都可以向对方发送或接收请求。

作为服务器，你可以在RSocket `@Controller`的任何处理方法上被注入一个`RSocketRequester`实例。作为客户端，你需要首先配置并建立一个RSocket连接。Spring Boot为这种情况自动配置了一个`RSocketRequester.Builder`，并配备了预期的编解码器。

`RSocketRequester.Builder`实例是一个Prototype Bean，意味着每个注入点都会为你提供一个新的实例。这样做是有目的的，因为这个构建器是有状态的，你不应该用同一个实例创建具有不同设置的请求者。

下面的代码显示了一个典型的例子。

```java
import reactor.core.publisher.Mono;

import org.springframework.messaging.rsocket.RSocketRequester;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final RSocketRequester rsocketRequester;

    public MyService(RSocketRequester.Builder rsocketRequesterBuilder) {
        this.rsocketRequester = rsocketRequesterBuilder.tcp("example.org", 9898);
    }

    public Mono<User> someRSocketCall(String name) {
        return this.rsocketRequester.route("user").data(name).retrieveMono(User.class);
    }

}
```

## 10. Security

如果[Spring Security](https://spring.io/projects/spring-security)在classpath上，那么Web应用程序默认是安全的。Spring Boot依靠Spring Security的内容协商策略来决定是使用`httpBasic`还是`formLogin`。要为Web应用添加方法级安全，你也可以添加`@EnableGlobalMethodSecurity`，并加上你想要的设置。其他信息可以在[Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.5.1/reference/html5/#jc-method)中找到。

默认的`UserDetailsService`有一个用户。用户名是`user`，密码是随机的，当应用程序启动时，密码会被打印在INFO级别，如以下例子所示。

```text
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

> 如果你微调你的日志配置，确保`org.springframework.boot.autoconfigure.security`类别被设置为记录`INFO`级的消息。否则，默认密码不会被打印出来。

你可以通过提供`spring.security.user.name`和`spring.security.user.password`来改变用户名和密码。

你在Web应用程序中默认得到的基本功能是。

* 一个`UserDetailsService`（如果是WebFlux应用程序，则为`ReactiveUserDetailsService`）bean，具有内存存储和一个具有生成密码的单一用户（关于用户的属性，见[`SecurityProperties.User`](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/autoconfigure/security/SecurityProperties.User.html)）。
* 整个应用程序（包括执行器端点，如果执行器在classpath上）基于表单的登录或HTTP Basic安全（取决于请求中的`Accept`头）。
* 一个`DefaultAuthenticationEventPublisher`用于发布认证事件。

你可以通过添加一个bean来提供一个不同的`AuthenticationEventPublisher`。

### 10.1. MVC Security

默认的安全配置在`SecurityAutoConfiguration`和`UserDetailsServiceAutoConfiguration`中实现。`SecurityAutoConfiguration`导入`SpringBootWebSecurityConfiguration`用于Web安全，`UserDetailsServiceAutoConfiguration`配置认证，这也与非Web应用有关。要完全关闭默认的Web应用安全配置，或结合多个Spring安全组件，如OAuth2客户端和资源服务器，请添加一个`SecurityFilterChain`类型的bean（这样做不会禁用`UserDetailsService`配置或Actuator的安全性）。

为了关闭 `UserDetailsService` 的配置，你可以添加一个`UserDetailsService`、`AuthenticationProvider`或`AuthenticationManager`类型的bean。

访问规则可以通过添加一个自定义的`SecurityFilterChain`或`WebSecurityConfigurerAdapter`bean来重写。Spring Boot提供了方便的方法，可以用来覆盖执行器端点和静态资源的访问规则。`EndpointRequest`可用于创建一个基于`management.endpoints.web.base-path`属性的`RequestMatcher`。`PathRequest`可以用来为常用位置的资源创建`RequestMatcher`。

### 10.2. WebFlux Security

与Spring MVC应用程序类似，你可以通过添加`spring-boot-starter-security`依赖关系来保护你的WebFlux应用程序。默认的安全配置是在`ReactiveSecurityAutoConfiguration`和`UserDetailsServiceAutoConfiguration`中实现。`ReactiveSecurityAutoConfiguration` 导入 `WebFluxSecurityConfiguration` 用于Web安全，`UserDetailsServiceAutoConfiguration` 用于配置认证，这在非Web应用中也很重要。要完全关闭默认的Web应用安全配置，你可以添加一个`WebFilterChainProxy`类型的bean（这样做不会禁用`UserDetailsService`配置或Actuator的安全）。

要关闭 `UserDetailsService` 配置，你可以添加一个 `ReactiveUserDetailsService` 或 `ReactiveAuthenticationManager` 类型的bean。

访问规则和多个Spring安全组件的使用，如OAuth 2客户端和资源服务器，可以通过添加一个自定义的`SecurityWebFilterChain`bean来配置。Spring Boot提供了方便的方法，可用于覆盖执行器端点和静态资源的访问规则。`EndpointRequest`可用于创建基于`management.endpoints.web.base-path`属性的`ServerWebExchangeMatcher`。

`PathRequest`可以用来为常用位置的资源创建一个`ServerWebExchangeMatcher`。

例如，你可以通过添加以下内容来定制你的安全配置。

```java
import org.springframework.boot.autoconfigure.security.reactive.PathRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration(proxyBeanMethods = false)
public class MyWebFluxSecurityConfiguration {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http.authorizeExchange((spec) -> {
            spec.matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll();
            spec.pathMatchers("/foo", "/bar").authenticated();
        });
        http.formLogin();
        return http.build();
    }

}
```

### 10.3. OAuth2

[OAuth2](https://oauth.net/2/)是一个被广泛使用的授权框架，由Spring支持。

#### 10.3.1. Client

如果你的classpath上有`spring-security-oauth2-client`，你可以利用一些自动配置来设置一个OAuth2/Open ID Connect客户端。这个配置利用了`OAuth2ClientProperties`下的属性。同样的属性也适用于servlet和响应式应用程序。

你可以在`spring.security.oauth2.client`前缀下注册多个OAuth2客户端和提供者，如以下例子所示。

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          my-client-1:
            client-id: "abcd"
            client-secret: "password"
            client-name: "Client for user scope"
            provider: "my-oauth-provider"
            scope: "user"
            redirect-uri: "https://my-redirect-uri.com"
            client-authentication-method: "basic"
            authorization-grant-type: "authorization-code"

          my-client-2:
            client-id: "abcd"
            client-secret: "password"
            client-name: "Client for email scope"
            provider: "my-oauth-provider"
            scope: "email"
            redirect-uri: "https://my-redirect-uri.com"
            client-authentication-method: "basic"
            authorization-grant-type: "authorization_code"

        provider:
          my-oauth-provider:
            authorization-uri: "https://my-auth-server/oauth/authorize"
            token-uri: "https://my-auth-server/oauth/token"
            user-info-uri: "https://my-auth-server/userinfo"
            user-info-authentication-method: "header"
            jwk-set-uri: "https://my-auth-server/token_keys"
            user-name-attribute: "name"
```

对于支持[OpenID Connect discovery](https://openid.net/specs/openid-connect-discovery-1_0.html)的OpenID Connect提供商，配置可以进一步简化。提供者需要配置一个`issuer-uri`，这是它作为其发行者标识符所主张的URI。例如，如果提供的`issuer-uri`是 `https://example.com`，那么将向 `https://example.com/.well-known/openid-configuration` 提出`OpenID Provider Configuration Request`。结果将是一个 `OpenID Provider Configuration Response`。下面的例子显示了如何用`issuer-uri`来配置OpenID连接提供商。

```yaml
spring:
  security:
    oauth2:
      client:
        provider:
          oidc-provider:
            issuer-uri: "https://dev-123456.oktapreview.com/oauth2/default/"
```

默认情况下，Spring Security的`OAuth2LoginAuthenticationFilter`只处理匹配`/login/oauth2/code/*`的URL。如果你想自定义`redirect-uri`以使用不同的模式，你需要提供配置来处理该自定义模式。例如，对于Servlet应用程序，你可以添加你自己的`SecurityFilterChain`，类似于以下内容。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration(proxyBeanMethods = false)
public class MyOAuthClientConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated();
        http.oauth2Login().redirectionEndpoint().baseUri("custom-callback");
        return http.build();
    }

}
```

> Spring Boot自动配置了一个`InMemoryOAuth2AuthorizedClientService`，它被Spring Security用来管理客户端注册。`InMemoryOAuth2AuthorizedClientService`的功能有限，我们建议只在开发环境中使用它。对于生产环境，请考虑使用`JdbcOAuth2AuthorizedClientService`或创建你自己的`OAuth2AuthorizedClientService`实现。

##### OAuth2 client registration for common providers

对于常见的OAuth2和OpenID提供者，包括Google、Github、Facebook和Okta，我们提供了一组提供者的默认值（分别为`google`、`github`、`facebook`和`okta`）。

如果你不需要定制这些提供者，你可以将`provider`属性设置为你需要推断默认值的那个。另外，如果客户端注册的密钥与默认支持的提供者相匹配，Spring Boot也会推断出这一点。

换句话说，下面例子中的两个配置使用的是Google提供者。

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          my-client:
            client-id: "abcd"
            client-secret: "password"
            provider: "google"
          google:
            client-id: "abcd"
            client-secret: "password"

```

#### 10.3.2. 资源服务器

如果你的classpath上有`spring-security-oauth2-resource-server`，Spring Boot可以设置一个OAuth2资源服务器。对于JWT配置，需要指定JWK Set URI或OIDC Issuer URI，如以下例子所示。

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: "https://example.com/oauth2/default/v1/keys"
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: "https://dev-123456.oktapreview.com/oauth2/default/"

```

> 如果授权服务器不支持JWK Set URI，你可以在资源服务器上配置用于验证JWT签名的公钥。这可以通过`spring.security.oauth2.resourceserver.jwt.public-key-location`属性来完成，其中的值需要指向一个包含PEM编码的x509格式的公钥的文件。

同样的属性适用于Servlet和反应式应用程序。

另外，你可以为Servlet应用程序定义你自己的`JwtDecoder`bean，或者为反应式应用程序定义`ReactiveJwtDecoder`。

在使用不透明令牌而不是JWT的情况下，你可以配置以下属性，通过自省来验证令牌。

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        opaquetoken:
          introspection-uri: "https://example.com/check-token"
          client-id: "my-client-id"
          client-secret: "my-client-secret"

```

同样，同样的属性也适用于servlet和reactive应用程序。

另外，你可以为Servlet应用程序定义你自己的`OpaqueTokenIntrospector` bean，或者为反应式应用程序定义`ReactiveOpaqueTokenIntrospector`。

#### 10.3.3. 授权服务器

目前，Spring Security并不提供对实现OAuth 2.0授权服务器的支持。然而，这个功能可以从[Spring Security OAuth](https://spring.io/projects/spring-security-oauth)项目中获得，该项目最终将被Spring Security完全取代。在此之前，你可以使用`spring-security-oauth2-autoconfigure`模块来轻松设置一个OAuth 2.0授权服务器，具体说明请参见其[document](https://docs.spring.io/spring-security-oauth2-boot/)。

### 10.4. SAML 2.0

#### 10.4.1. Relying Party

如果你的classpath上有`spring-security-saml2-service-provider`，你可以利用一些自动配置来设置一个SAML 2.0的信赖方。这个配置利用了`Saml2RelyingPartyProperties`下的属性。

依赖方注册代表了身份提供商（IDP）和服务提供商（SP）之间的配对配置。你可以在`spring.security.saml2.relyingparty`前缀下注册多个依赖方，如以下例子所示。

```yaml
spring:
  security:
    saml2:
      relyingparty:
        registration:
          my-relying-party1:
            signing:
              credentials:
              - private-key-location: "path-to-private-key"
                certificate-location: "path-to-certificate"
            decryption:
              credentials:
              - private-key-location: "path-to-private-key"
                certificate-location: "path-to-certificate"
            identityprovider:
              verification:
                credentials:
                - certificate-location: "path-to-verification-cert"
              entity-id: "remote-idp-entity-id1"
              sso-url: "https://remoteidp1.sso.url"

          my-relying-party2:
            signing:
              credentials:
              - private-key-location: "path-to-private-key"
                certificate-location: "path-to-certificate"
            decryption:
              credentials:
              - private-key-location: "path-to-private-key"
                certificate-location: "path-to-certificate"
            identityprovider:
              verification:
                credentials:
                - certificate-location: "path-to-other-verification-cert"
              entity-id: "remote-idp-entity-id2"
              sso-url: "https://remoteidp2.sso.url"
```

### 10.5. Actuator Security

为了安全起见，所有除`/health`以外的执行器都被默认禁用。`management.endpoints.web.exposure.include`属性可以用来启用执行器。

如果Spring Security在classpath上，并且没有其他`WebSecurityConfigurerAdapter`或`SecurityFilterChain` bean存在，所有`/health`以外的执行器都由Spring Boot自动配置保护。如果你定义了一个自定义的`WebSecurityConfigurerAdapter`或`SecurityFilterChain`bean，Spring Boot的自动配置就会退出，你将完全控制执行器的访问规则。

> 在设置 `management.endpoints.web.exposure.include` 之前，确保暴露的执行器不包含敏感信息，并且/或者通过将其置于防火墙或Spring Security之类的东西来确保安全。

#### 10.5.1. 跨站请求伪造保护

由于Spring Boot依赖于Spring Security的默认值，CSRF保护默认是打开的。这意味着在使用默认安全配置时，需要`POST`关闭和记录器端点）、`PUT`或`DELETE`的执行器端点会出现403禁止的错误。

> 我们建议只有在你创建的服务被非浏览器客户端使用时才完全禁用CSRF保护。

关于CSRF保护的其他信息可以在[Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.5.1/reference/html5/#csrf)中找到。

## 11. 使用SQL数据库

[Spring Framework](https://spring.io/projects/spring-framework)提供了对SQL数据库工作的广泛支持，从使用`JdbcTemplate`的直接JDBC访问到完整的 "object relational mapping" 技术，如Hibernate。[Spring Data](https://spring.io/projects/spring-data)提供了额外的功能：直接从接口创建`Repository`实现，并使用惯例从你的方法名称中生成查询。

### 11.1. 配置一个数据源

Java的`javax.sql.DataSource`接口提供了一个处理数据库连接的标准方法。传统上，`DataSource` 使用一个 `URL` 和一些凭证来建立一个数据库连接。

> 更多高级的例子请参见["How-to"部分](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-access.configure-custom-datasource)，通常是对数据源的配置进行完全控制。

#### 11.1.1. 嵌入式数据库支持

通过使用内存中的嵌入式数据库来开发应用程序通常是很方便的。很明显，内存数据库不提供持久性存储。你需要在你的应用程序开始时填充你的数据库，并准备在你的应用程序结束时丢弃数据。

> "How-to" 包括一个[关于如何初始化数据库的部分](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-initialization)。

Spring Boot可以自动配置嵌入式[H2](https://www.h2database.com/)、[HSQL](http://hsqldb.org/)和[Derby](https://db.apache.org/derby/) 数据库。你不需要提供任何连接URL。你只需要包括一个你想使用的嵌入式数据库的构建依赖。如果在classpath上有多个嵌入式数据库，设置`spring.datasource.embedded-database-connection`配置属性来控制使用哪一个。将该属性设置为 `none`，可以禁止对嵌入式数据库进行自动配置。

> 如果你在测试中使用这个功能，你可能会注意到，无论你使用多少个application context，整个测试套件都在重复使用同一个数据库。如果你想确保每个上下文都有一个单独的嵌入式数据库，你应该把`spring.datasource.generate-unique-name`设置为`true`。

例如，典型的POM依赖关系如下。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```

你需要依赖`spring-jdbc`来自动配置一个嵌入式数据库。在这个例子中，它是通过 `spring-boot-starter-data-jpa` 传递依赖到系统的。

如果出于某种原因，你确实为一个嵌入式数据库配置了连接URL，请注意确保数据库的自动关机功能被禁用。如果你使用H2，你应该使用`DB_CLOSE_ON_EXIT=FALSE`来做到这一点。如果你使用HSQLDB，你应该确保不使用`shutdown=true`。禁用数据库的自动关闭让Spring Boot控制数据库的关闭时间，从而确保一旦不再需要对数据库的访问，就会发生关闭。

#### 11.1.2. 与生产数据库的连接

生产数据库连接也可以通过使用池化数据源来自动配置。

#### 11.1.3. 数据源配置

数据源配置由`spring.datasource.*`中的外部配置属性控制。例如，你可以在`application.properties`中声明以下部分。

```yaml
spring:
  datasource:
    url: "jdbc:mysql://localhost/test"
    username: "dbuser"
    password: "dbpass"

```

你至少应该通过设置`spring.datasource.url`属性指定URL。否则，Spring Boot会尝试自动配置一个嵌入式数据库。

Spring Boot可以从URL中推断出大多数数据库的JDBC驱动类。如果你需要指定一个特定的类，你可以使用`spring.datasource.driver-class-name`属性。

为了创建一个池化的`DataSource`，我们需要能够验证一个有效的`Driver`类是可用的，所以我们在做任何事情之前都要检查。换句话说，如果你设置了`spring.datasource.driver-class-name=com.mysql.jdbc.Driver`，那么这个类必须是可加载的。

更多支持的选项请参见[`DataSourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)。这些是标准的选项，无论[实际的实现](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.sql.datasource.connection-pool)如何，都能发挥作用。也可以通过使用各自的前缀（`spring.datasource.hikari.*` , `spring.datasource.tomcat.*` , `spring.datasource.dbcp2.*` , 和`spring.datasource.oracleucp.*`）来微调具体的实现设置。请参考你所使用的连接池实现的文档以了解更多细节。

例如，如果你使用[Tomcat连接池](https://tomcat.apache.org/tomcat-9.0-doc/jdbc-pool.html#Common_Attributes)，你可以定制许多额外的设置，如以下例子所示。

```yaml
spring:
  datasource:
    tomcat:
      max-wait: 10000
      max-active: 50
      test-on-borrow: true

```

这将设置池子在没有连接可用时等待10000ms后抛出一个异常，限制最大连接数为50，并在从池子中借用连接前验证连接。

#### 11.1.4. 支持的连接池

Spring Boot使用以下算法来选择特定的实现。

1. 我们更喜欢[HikariCP](https://github.com/brettwooldridge/HikariCP)，因为其性能和并发性。如果HikariCP可用，我们总是选择它。
2. 2.否则，如果Tomcat池的`DataSource'可用，我们就使用它。
3. 否则，如果[Commons DBCP2](https://commons.apache.org/proper/commons-dbcp/)是可用的，我们就使用它。
4. 如果HikariCP、Tomcat和DBCP2都不可用，并且Oracle UCP可用，我们就使用它。

如果你使用`spring-boot-starter-jdbc`或`spring-boot-starter-data-jpa` "starters"，你会自动得到对`HikariCP`的依赖。

你可以完全绕过这个算法，通过设置`spring.datasource.type`属性来指定要使用的连接池。如果你在Tomcat容器中运行你的应用程序，这一点尤其重要，因为`tomcat-jdbc`是默认提供的。

额外的连接池总是可以手动配置的，使用`DataSourceBuilder`。如果你定义你自己的`DataSource`bean，就不会发生自动配置。`DataSourceBuilder`支持以下的连接池。

* HikariCP
* Tomcat pooling `Datasource`
* Commons DBCP2
* Oracle UCP & `OracleDataSource`
* Spring Framework’s `SimpleDriverDataSource`
* H2 `JdbcDataSource`
* PostgreSQL `PGSimpleDataSource`

#### 11.1.5. 连接到一个JNDI数据源

如果你将Spring Boot应用程序部署到应用服务器上，你可能想通过使用应用服务器的内置功能来配置和管理你的数据源，并通过使用JNDI来访问它。

`spring.datasource.jndi-name`属性可以作为`spring.datasource.url`、`spring.datasource.username`和`spring.datasource.password`属性的替代品，从特定的JNDI位置访问`DataSource`。例如，`application.properties`中的以下部分显示了如何访问JBoss AS定义的`DataSource`。

```yaml
spring:
  datasource:
    jndi-name: "java:jboss/datasources/customers"
```

### 11.2. 使用JdbcTemplate

Spring的`JdbcTemplate`和`NamedParameterJdbcTemplate`类是自动配置的，你可以将它们直接`@Autowire`到你自己的Bean中，如下例所示。

```java
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void doSomething() {
        this.jdbcTemplate ...
    }

}
```

你可以通过使用`spring.jdbc.template.*`属性来定制模板的一些属性，如以下例子所示。

```yaml
spring:
  jdbc:
    template:
      max-rows: 500
```

`NamedParameterJdbcTemplate`在幕后重复使用同一个`JdbcTemplate`实例。如果定义了一个以上的 `JdbcTemplate`，并且不存在主要的候选者，`NamedParameterJdbcTemplate` 就不会被自动配置。

### 11.3. JPA 和 Spring Data JPA

Java Persistence API是一项标准技术，它可以让你把对象 "映射" 到关系型数据库。`spring-boot-starter-data-jpa` POM提供了一个快速入门的方法。它提供了以下关键的依赖性。

* Hibernate：最流行的JPA实现之一。
* Spring Data JPA：帮助你实现基于JPA的Repository。
* Spring ORM：来自Spring框架的核心ORM支持。

> 我们在这里不谈JPA或[Spring Data](https://spring.io/projects/spring-data)的太多细节。你可以按照[spring.io](https://spring.io/)的[用JPA访问数据](https://spring.io/guides/gs/accessing-data-jpa/)指南，并阅读[Spring Data JPA](https://spring.io/projects/spring-data-jpa)和[Hibernate](https://hibernate.org/orm/documentation/)参考文档。

#### 11.3.1. Entity 类

传统上，JPA "Entity" 类是在`persistence.xml`文件中指定的。在Spring Boot中，这个文件是不必要的，而是使用 "Entity Scanning"。默认情况下，你的主配置类（用`@EnableAutoConfiguration`或`@SpringBootApplication`注释的那个）下面的所有包都会被搜索到。

任何带有`@Entity`、`@Embeddable`或`@MappedSuperclass`注释的类都会被考虑。一个典型的实体类类似于下面的例子。

```java
import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.state = state;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }

    // ... etc

}
```

> 你可以通过使用`@EntityScan`注解来定制实体扫描位置。参见[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-access.separate-entity-definitions-from-spring-configuration)的操作指南。

#### 11.3.2. Spring Data JPA Repositories

[Spring Data JPA](https://spring.io/projects/spring-data-jpa) repositories 是你可以定义的接口，用于访问数据。JPA查询是由你的方法名称自动创建的。例如，一个`CityRepository` 可能会声明一个 `findAllByState(String state)`方法，以查找某个州的所有城市。

对于更复杂的查询，你可以用Spring Data的[`Query`](https://docs.spring.io/spring-data/jpa/docs/2.5.3/api/org/springframework/data/jpa/repository/Query.html)注解来注释你的方法。

Spring Data Repositories 通常从[`Repository`](https://docs.spring.io/spring-data/commons/docs/2.5.3/api/org/springframework/data/repository/Repository.html)或[`CrudRepository`](https://docs.spring.io/spring-data/commons/docs/2.5.3/api/org/springframework/data/repository/CrudRepository.html)接口延伸。如果你使用自动配置，存储库会从包含你的主配置类（用`@EnableAutoConfiguration`或`@SpringBootApplication`注解的那个）向下搜索。

下面的例子显示了一个典型的Spring Data资源库接口定义。

```java
import org.springframework.boot.docs.features.sql.jpaandspringdata.entityclasses.City;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.repository.Repository;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

Spring Data JPA存储库支持三种不同的引导模式：default, deferred 和 lazy。要启用deferred或lazy引导，请将`spring.data.jpa.repositories.bootstrap-mode`属性分别设置为`deferred`或`lazy`。当使用deferred或lazy引导时，自动配置的`EntityManagerFactoryBuilder`将使用上下文的`AsyncTaskExecutor`，如果有的话，作为引导执行器。如果存在多个，将使用名为 `applicationTaskExecutor` 的那个。

> 当使用deferred或lazy引导时，确保在应用上下文引导阶段后推迟对JPA基础设施的任何访问。你可以使用`SmartInitializingSingleton`来调用任何需要JPA基础设施的初始化。对于作为Spring Bean创建的JPA组件（如转换器），使用`ObjectProvider`来延迟解决依赖关系（如果有）。

这一节的内容对于Spring Data JPA来说，入门都不算。关于完整的细节，请参阅[Spring Data JPA参考文档](https://docs.spring.io/spring-data/jpa/docs/2.5.3/reference/html)。

#### 11.3.3. 创建和删除JPA数据库

默认情况下，JPA数据库**只在**你使用嵌入式数据库（H2、HSQL或Derby）时自动创建。你可以通过使用`spring.jpa.*`属性明确地配置JPA设置。例如，为了创建和删除表，你可以在你的`application.properties`中添加以下一行。

```yaml
spring:
  jpa:
    hibernate.ddl-auto: "create-drop"
```

> Hibernate自己的内部属性名称（如果你碰巧记得比较清楚）是`hibernate.hbm2ddl.auto`。你可以通过使用`spring.jpa.properties.*`来设置它，以及其他Hibernate的本地属性（在将它们添加到实体管理器之前，前缀被剥离）。下面一行显示了一个为Hibernate设置JPA属性的例子。

```yaml
spring:
  jpa:
    properties:
      hibernate:
        "globally_quoted_identifiers": "true"
```

前面例子中的一行将`hibernate.global_quoted_identifiers`属性的值`true`传递给Hibernate实体管理器。

默认情况下，DDL的执行（或验证）会推迟到`ApplicationContext`启动之后。还有一个`spring.jpa.generate-ddl`标志，但如果Hibernate自动配置处于激活状态，它就不会被使用，因为`ddl-auto`的设置更加细化。

#### 11.3.4 在View中打开EntityManager

如果你正在运行一个Web应用程序，Spring Boot默认注册了[`OpenEntityManagerInViewInterceptor`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html)来应用 "Open EntityManager in View "模式，以允许在Web视图中进行延迟加载。如果你不想要这种行为，你应该在你的`application.properties`中把`spring.jpa.open-in-view`设置为`false`。

### 11.4. Spring Data JDBC

Spring Data包括对JDBC的repository支持，并将为`CrudRepository`上的方法自动生成SQL。对于更高级的查询，提供了一个`@Query`注解。

当必要的依赖在classpath上时，Spring Boot将自动配置Spring Data的JDBC资源库。它们可以通过对`spring-boot-starter-data-jdbc`的单一依赖性添加到你的项目中。如果有必要，你可以通过添加`@EnableJdbcRepositories`注解或`JdbcConfiguration`子类到你的应用程序来控制Spring Data JDBC的配置。

> 关于Spring Data JDBC的完整细节，请参考[参考文档](https://docs.spring.io/spring-data/jdbc/docs/2.2.3/reference/html/)。

### 11.5. 使用H2的Web Console

[H2数据库](https://www.h2database.com/)提供了一个[基于浏览器的控制台](https://www.h2database.com/html/quickstart.html#h2_console)，Spring Boot可以为你自动配置。当满足以下条件时，该控制台会被自动配置。

* 你正在开发一个基于Servlet的Web应用程序。
* `com.h2database:h2`位于classpath上。
* 你正在使用[Spring Boot的开发者工具](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools)。

如果你不使用Spring Boot的开发者工具，但仍想利用H2的控制台，你可以配置`spring.h2.console.enabled`属性，值为`true`。

H2控制台只在开发过程中使用，所以你应该注意确保`spring.h2.console.enabled`在生产环境中不被设置为`true`。

#### 11.5.1. 改变H2控制台的路径

默认情况下，控制台的位置是`/h2-console`。你可以通过使用`spring.h2.console.path`属性来定制控制台的路径。

### 11.6. 使用 jOOQ

jOOQ面向对象查询（[jOOQ](https://www.jooq.org/)）是[Data Geekery](https://www.datageekery.com/)的一个流行产品，它从你的数据库中生成Java代码，让你通过其流畅的API建立类型安全的SQL查询。商业版和开源版都可以与Spring Boot一起使用。

#### 11.6.1. 代码生成

为了使用jOOQ类型安全的查询，你需要从你的数据库模式中生成Java类。你可以按照[jOOQ用户手册](https://www.jooq.org/doc/3.14.13/manual-single-page/#jooq-in-7-steps-step3)中的说明进行操作。如果你使用`jooq-codegen-maven`插件，同时使用`spring-boot-starter-parent`"parent POM"，你可以安全地省略该插件的`<version>`标签。你也可以使用Spring Boot定义的版本变量（如`h2.version`）来声明该插件的数据库依赖性。下面的列表显示了一个例子。

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <executions>
        ...
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:~/yourdatabase</url>
        </jdbc>
        <generator>
            ...
        </generator>
    </configuration>
</plugin>
```

#### 11.6.2. 使用 DSLContext

jOOQ提供的流畅的API是通过`org.jooq.DSLContext`接口启动的。Spring Boot将`DSLContext`自动配置为Spring Bean，并将其连接到你的应用程序`DataSource`。要使用`DSLContext`，你可以注入它，如下面的例子所示。

```java
import java.util.GregorianCalendar;
import java.util.List;

import org.jooq.DSLContext;

import org.springframework.stereotype.Component;

import static org.springframework.boot.docs.features.sql.jooq.dslcontext.Tables.AUTHOR;

@Component
public class MyBean {

    private final DSLContext create;

    public MyBean(DSLContext dslContext) {
        this.create = dslContext;
    }


}
```

jOOQ手册倾向于使用一个名为`create`的变量来保存`DSLContext`。

然后你可以使用`DSLContext`来构建你的查询，如以下例子所示。

```java
public List<GregorianCalendar> authorsBornAfter1980() {
    return this.create.selectFrom(AUTHOR)
            .where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
            .fetch(AUTHOR.DATE_OF_BIRTH);
```

#### 11.6.3. jOOQ SQL方言

除非配置了`spring.jooq.sql-dialect`属性，否则Spring Boot会决定为你的数据源使用哪种SQL方言。如果Spring Boot无法检测到方言，它就会使用`DEFAULT`。

> Spring Boot只能自动配置jOOQ开源版本所支持的方言。

#### 11.6.4. 自定义 jOOQ

更高级的定制可以通过定义你自己的`DefaultConfigurationCustomizer`bean来实现，它将在创建`org.jooq.Configuration` `@Bean`之前被调用。这优先于任何由自动配置应用的东西。

如果你想完全控制jOOQ的配置，你也可以创建你自己的`org.jooq.Configuration` `@Bean`。

### 11.7. 使用 R2DBC

响应式关系型数据库连接（[R2DBC](https://r2dbc.io/)）项目为关系型数据库带来了响应式编程API。R2DBC的`io.r2dbc.spi.Connection`提供了一种处理非阻塞数据库连接的标准方法。连接是通过`ConnectionFactory`提供的，类似于jdbc的`DataSource`。

`ConnectionFactory`的配置由`spring.r2dbc.*`的外部配置属性控制。例如，你可以在`application.properties`中声明以下部分。

```yaml
spring:
  r2dbc:
    url: "r2dbc:postgresql://localhost/test"
    username: "dbuser"
    password: "dbpass"
```

你不需要指定驱动类名称，因为Spring Boot从R2DBC的Connection Factory discovery中获取驱动。

至少应该提供URL。在URL中指定的信息优先于单个属性，即 `name` , `username` , `password` 和池选项。

> "How-to" 部分包括一个[关于如何初始化数据库的部分](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.data-initialization.using-basic-sql-scripts)。

要定制由 `ConnectionFactory` 创建的连接，即设置你不想（或不能）在中央数据库配置中配置的特定参数，你可以使用 `ConnectionFactoryOptionsBuilderCustomizer` `@Bean`。下面的例子显示了如何手动覆盖数据库端口，而其余的选项则来自应用程序的配置。

```java
import io.r2dbc.spi.ConnectionFactoryOptions;

import org.springframework.boot.autoconfigure.r2dbc.ConnectionFactoryOptionsBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyR2dbcConfiguration {

    @Bean
    public ConnectionFactoryOptionsBuilderCustomizer connectionFactoryPortCustomizer() {
        return (builder) -> builder.option(ConnectionFactoryOptions.PORT, 5432);
    }

}
```

下面的例子显示了如何设置一些PostgreSQL的连接选项。

```java
import java.util.HashMap;
import java.util.Map;

import io.r2dbc.postgresql.PostgresqlConnectionFactoryProvider;

import org.springframework.boot.autoconfigure.r2dbc.ConnectionFactoryOptionsBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyPostgresR2dbcConfiguration {

    @Bean
    public ConnectionFactoryOptionsBuilderCustomizer postgresCustomizer() {
        Map<String, String> options = new HashMap<>();
        options.put("lock_timeout", "30s");
        options.put("statement_timeout", "60s");
        return (builder) -> builder.option(PostgresqlConnectionFactoryProvider.OPTIONS, options);
    }

}
```

当一个`ConnectionFactory`bean可用时，常规的JDBC`DataSource`自动配置就会退缩。如果你想保留JDBC的`DataSource`自动配置，并能接受在响应式应用程序中使用阻塞的JDBC API的风险，在你的应用程序中的`@Configuration`类上添加`@Import(DataSourceAutoConfiguration.class)`来重新启用它。

#### 11.7.1. 嵌入式数据库支持

与[JDBC支持](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.sql.datasource.embedded)类似，Spring Boot可以自动配置嵌入式数据库以实现响应式使用。你不需要提供任何连接URL。你只需要包括一个你想使用的嵌入式数据库的构建依赖，如下面的例子所示。

```xml
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

> 如果你在测试中使用这个功能，你可能会注意到，无论你使用多少个应用程序上下文，整个测试套件都在重复使用同一个数据库。如果你想确保每个上下文都有一个独立的嵌入式数据库，你应该把`spring.r2dbc.generate-unique-name`设置为`true`。

#### 11.7.2. 使用DatabaseClient

一个`DatabaseClient`bean是自动配置的，你可以`@Autowire`它直接进入你自己的bean，如下面的例子所示。

```java
import java.util.Map;

import reactor.core.publisher.Flux;

import org.springframework.r2dbc.core.DatabaseClient;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final DatabaseClient databaseClient;

    public MyBean(DatabaseClient databaseClient) {
        this.databaseClient = databaseClient;
    }

    public Flux<Map<String, Object>> someMethod() {
        return this.databaseClient.sql("select * from user").fetch().all();
    }

}
```

#### 11.7.3. Spring Data R2DBC Repositories

[Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc)repositories 是你可以定义访问数据的接口。查询是由你的方法名称自动创建的。例如，一个`CityRepository` 接口可能会声明一个`findAllByState(String state)`方法，以找到一个给定州的所有城市。

对于更复杂的查询，你可以用Spring Data的[`Query`](https://docs.spring.io/spring-data/r2dbc/docs/1.3.3/api/org/springframework/data/r2dbc/repository/Query.html)注解来注释你的方法。

Spring Data资源库通常从[`Repository`](https://docs.spring.io/spring-data/commons/docs/2.5.3/api/org/springframework/data/repository/Repository.html)或[`CrudRepository`](https://docs.spring.io/spring-data/commons/docs/2.5.3/api/org/springframework/data/repository/CrudRepository.html)接口延伸。如果你使用自动配置，存储库会从包含你的主配置类（用`@EnableAutoConfiguration`或`@SpringBootApplication`注解的那个）向下搜索。

下面的例子显示了一个典型的Spring Data资源库接口定义。

```java
import reactor.core.publisher.Mono;

import org.springframework.data.repository.Repository;

public interface CityRepository extends Repository<City, Long> {

    Mono<City> findByNameAndStateAllIgnoringCase(String name, String state);

}
```

> 这一节内容连Spring Data R2DBC入门都不算。有关完整的细节，请参阅[Spring Data R2DBC参考文档](https://docs.spring.io/spring-data/r2dbc/docs/1.3.3/reference/html/)。

## 12. 使用NOSQL技术

Spring Data提供了额外的项目，帮助你访问各种NoSQL技术，包括。

* [MongoDB](https://spring.io/projects/spring-data-mongodb)
* [Neo4J](https://spring.io/projects/spring-data-neo4j)
* [Elasticsearch](https://spring.io/projects/spring-data-elasticsearch)
* [Redis](https://spring.io/projects/spring-data-redis)
* [GemFire](https://spring.io/projects/spring-data-gemfire) or [Geode](https://spring.io/projects/spring-data-geode)
* [Cassandra](https://spring.io/projects/spring-data-cassandra)
* [Couchbase](https://spring.io/projects/spring-data-couchbase)
* [LDAP](https://spring.io/projects/spring-data-ldap)

Spring Boot为Redis、MongoDB、Neo4j、Solr、Elasticsearch、Cassandra、Couchbase、LDAP和InfluxDB提供自动配置。你可以利用其他项目，但你必须自己配置它们。请参考[spring.io/projects/spring-data](https://spring.io/projects/spring-data)上的相应参考文档。

### 12.1. Redis

[Redis](https://redis.io/)是一个高速缓存、消息代理和功能丰富的键值存储。Spring Boot为[Lettuce](https://github.com/lettuce-io/lettuce-core/)和[Jedis](https://github.com/xetorthio/jedis/)客户端库以及[Spring Data Redis](https://github.com/spring-projects/spring-data-redis)提供了基本自动配置。

有一个`spring-boot-starter-data-redis`的 "Starter"，用于以方便的方式收集依赖关系。默认情况下，它使用[Lettuce](https://github.com/lettuce-io/lettuce-core/)。该启动器可以处理传统的和响应式的应用程序。

> 我们还提供了一个`spring-boot-starter-data-redis-reactive`的 "Starter"，以便与其他支持reactive的stores保持一致。

#### 12.1.1. 连接到Redis

你可以像其他Spring Bean一样，注入一个自动配置的`RedisConnectionFactory`、`StringRedisTemplate`或vanilla`RedisTemplate`实例。默认情况下，该实例会尝试连接到`localhost:6379`的Redis服务器。下面的列表显示了这样一个bean的例子。

```java
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final StringRedisTemplate template;

    public MyBean(StringRedisTemplate template) {
        this.template = template;
    }

    public Boolean someMethod() {
        return this.template.hasKey("spring");
    }

}
```

> 你也可以注册任意数量的实现`LettuceClientConfigurationBuilderCustomizer`的bean，以进行更高级的定制。如果你使用Jedis，`JedisClientConfigurationBuilderCustomizer`也可用。

如果你添加你自己的任何自动配置类型的`@Bean`，它将取代默认的类型（除了在`RedisTemplate`的情况下，当排除是基于Bean的名字`redisTemplate`，而不是它的类型）。

如果`commons-pool2`在classpath上，并且至少有一个[`RedisProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/data/redis/RedisProperties.java)的`Pool`选项被设置，那么池式连接工厂将被自动配置。

### 12.2. MongoDB

[MongoDB](https://www.mongodb.com/)是一个开源的NoSQL文档数据库，使用类似JSON的模式，而不是传统的基于表格的关系数据。Spring Boot为与MongoDB合作提供了一些便利，包括`spring-boot-starter-data-mongodb`和`spring-boot-starter-data-mongodb-reactive` "Starters"。

#### 12.2.1. 连接到 MongoDB Database

为了访问MongoDB数据库，你可以注入一个自动配置的`org.springframework.data.mongodb.MongoDatabaseFactory`。默认情况下，该实例会尝试连接到位于`mongodb://localhost/test`的MongoDB服务器。下面的例子展示了如何连接到MongoDB数据库。

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import org.springframework.data.mongodb.MongoDatabaseFactory;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final MongoDatabaseFactory mongo;

    public MyBean(MongoDatabaseFactory mongo) {
        this.mongo = mongo;
    }

    public MongoCollection<Document> someMethod() {
        MongoDatabase db = this.mongo.getMongoDatabase();
        return db.getCollection("users");
    }

}
```

如果你已经定义了自己的 `MongoClient`，它将被用来自动配置一个合适的 `MongoDatabaseFactory`。

自动配置的`MongoClient`是使用`MongoClientSettings`bean创建的。如果你已经定义了你自己的`MongoClientSettings`，它将被使用而不需要修改，`spring.data.mongodb`属性将被忽略。否则，`MongoClientSettings`将被自动配置，并将有`spring.data.mongodb`属性应用到它。在这两种情况下，你可以声明一个或多个`MongoClientSettingsBuilderCustomizer` bean 来微调`MongoClientSettings`的配置。每一个都将与用于构建`MongoClientSettings`的`MongoClientSettings.Builder`依次被调用。

你可以设置`spring.data.mongodb.uri`属性来改变URL，并配置额外的设置，如*replica set*，如以下例子所示。

```yaml
spring:
  data:
    mongodb:
      uri: "mongodb://user:secret@mongo1.example.com:12345,mongo2.example.com:23456/test"
```

另外，你也可以使用分离的属性来指定连接细节。例如，你可以在你的`application.properties`中声明以下设置。

```yaml
spring:
  data:
    mongodb:
      host: "mongoserver.example.com"
      port: 27017
      database: "test"
      username: "user"
      password: "secret"
```

如果没有指定`spring.data.mongodb.port`，将使用默认的`27017`。你可以从前面的例子中删除这一行。

如果你不使用Spring Data MongoDB，你可以注入一个`MongoClient`bean，而不是使用`MongoDatabaseFactory`。如果你想完全控制建立MongoDB连接，你也可以声明你自己的`MongoDatabaseFactory`或`MongoClient`bean。

> 如果你使用的是响应式驱动，SSL需要Netty。如果使用Netty，自动配置会自动配置这个工厂。

#### 12.2.2. MongoTemplate

[Spring Data MongoDB](https://spring.io/projects/spring-data-mongodb)提供了一个[`MongoTemplate`](https://docs.spring.io/spring-data/mongodb/docs/3.2.3/api/org/springframework/data/mongodb/core/MongoTemplate.html)类，其设计与Spring的`JdbcTemplate`非常相似。与`JdbcTemplate`一样，Spring Boot为你自动配置了一个Bean来注入模板，如下所示。

```java
import com.mongodb.client.MongoCollection;
import org.bson.Document;

import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final MongoTemplate mongoTemplate;

    public MyBean(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    public MongoCollection<Document> someMethod() {
        return this.mongoTemplate.getCollection("users");
    }

}
```

完整的细节见[`MongoOperations` Javadoc](https://docs.spring.io/spring-data/mongodb/docs/3.2.3/api/org/springframework/data/mongodb/core/MongoOperations.html)。

#### 12.2.3. Spring Data MongoDB Repositories

Spring Data包括对MongoDB的repository 支持。与前面讨论的JPA repositories 一样，其基本原则是根据方法名称自动构建查询。

事实上，Spring Data JPA和Spring Data MongoDB都共享相同的公共基础设施。你可以采用前面的JPA例子，假设`City`现在是MongoDB的数据类，而不是JPA的`@Entity`，那么它的工作方式是一样的，如下例所示。

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.repository.Repository;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

你可以通过使用`@EntityScan`注释来定制document扫描位置。

> 关于Spring Data MongoDB的完整细节，包括其丰富的对象映射技术，请参考其[参考文档](https://spring.io/projects/spring-data-mongodb)。

#### 12.2.4. 嵌入式Mongo

Spring Boot为[嵌入式Mongo](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)提供自动配置。要在你的Spring Boot应用程序中使用它，需要添加对`de.flapdoodle.embed:de.flapdoodle.embed.mongo`的依赖。

Mongo监听的端口可以通过设置`spring.data.mongodb.port`属性进行配置。由`MongoAutoConfiguration`创建的`MongoClient`会自动配置为使用随机分配的端口。

> 如果你不配置自定义端口，嵌入式支持默认使用一个随机端口（而不是27017）。

如果你在classpath上有SLF4J，Mongo产生的输出会自动路由到一个名为`org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongo`的日志器。

你可以声明你自己的`IMongodConfig`和`IRuntimeConfig` Bean 来控制Mongo实例的配置和日志路由。下载配置可以通过声明`DownloadConfigBuilderCustomizer` Bean来定制。

### 12.3. Neo4j

[Neo4j](https://neo4j.com/)是一个开源的NoSQL图数据库，它使用由第一类关系连接的节点的丰富数据模型，比传统的RDBMS方法更适合连接大数据。Spring Boot为与Neo4j合作提供了一些便利，包括`spring-boot-starter-data-neo4j` "Starter"。

#### 12.3.1. 连接到 Neo4j Database

为了访问Neo4j服务器，你可以注入一个自动配置的`org.neo4j.driver.Driver`。默认情况下，实例会尝试使用Bolt协议连接到`localhost:7687`的Neo4j服务器。下面的例子显示了如何注入一个Neo4j的`Driver`，让你访问`Session`。

```java
import org.neo4j.driver.Driver;
import org.neo4j.driver.Session;
import org.neo4j.driver.Values;

import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final Driver driver;

    public MyBean(Driver driver) {
        this.driver = driver;
    }

    public String someMethod(String message) {
        try (Session session = this.driver.session()) {
            return session.writeTransaction((transaction) -> transaction
                    .run("CREATE (a:Greeting) SET a.message = $message RETURN a.message + ', from node ' + id(a)",
                            Values.parameters("message", message))
                    .single().get(0).asString());
        }
    }

}
```

你可以使用`spring.neo4j.*`属性来配置驱动程序的各个方面。下面的例子显示了如何配置要使用的URI和凭证。

```yaml
spring:
  neo4j:
    uri: "bolt://my-server:7687"
    authentication:
      username: "neo4j"
      password: "secret"
```

自动配置的 `Driver` 是用 `ConfigBuilder` 创建的。为了微调它的配置，声明一个或多个`ConfigBuilderCustomizer` Bean。每一个都将与用于创建 `Driver` 的 `ConfigBuilder` 一起被依次调用。

#### 12.3.2. Spring Data Neo4j Repositories

Spring Data包括对Neo4j的repository支持。关于Spring Data Neo4j的完整细节，请参阅[参考文档](https://docs.spring.io/spring-data/neo4j/docs/6.1.3/reference/html/)。

Spring Data Neo4j与Spring Data JPA共享共同的基础架构，就像许多其他Spring Data模块那样。你可以采用前面的JPA例子，将`City` 定义为Spring Data Neo4j的`@Node`，而不是JPA的`@Entity`，资源库的抽象也以同样的方式工作，如下面的例子中所示。

```java
import java.util.Optional;

import org.springframework.data.neo4j.repository.Neo4jRepository;

public interface CityRepository extends Neo4jRepository<City, Long> {

    Optional<City> findOneByNameAndState(String name, String state);

}
```

`spring-boot-starter-data-neo4j` "Starter" 实现了repository支持和事务管理。Spring Boot使用`Neo4jTemplate`或`ReactiveNeo4jTemplate` Bean，支持经典和响应式Neo4j仓库。当Project Reactor在classpath上可用时，响应式也是自动配置的。

你可以通过使用`@EnableNeo4jRepositories`和`@EntityScan`分别在`@Configuration`-bean上自定义查找存储库和实体的位置。

在使用响应式的应用程序中，`ReactiveTransactionManager`不是自动配置的。要启用事务管理，必须在你的配置中定义以下Bean。

```java
import org.neo4j.driver.Driver;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.neo4j.core.ReactiveDatabaseSelectionProvider;
import org.springframework.data.neo4j.core.transaction.ReactiveNeo4jTransactionManager;

@Configuration(proxyBeanMethods = false)
public class MyNeo4jConfiguration {

    @Bean
    public ReactiveNeo4jTransactionManager reactiveTransactionManager(Driver driver,
            ReactiveDatabaseSelectionProvider databaseNameProvider) {
        return new ReactiveNeo4jTransactionManager(driver, databaseNameProvider);
    }

}
```

### 12.4. Solr

[Apache Solr](https://lucene.apache.org/solr/)是一个搜索引擎。Spring Boot为Solr 5客户端库提供了基本的自动配置功能。

#### 12.4.1. 连接到 Solr

你可以像其他Spring Bean一样注入一个自动配置的`SolrClient`实例。默认情况下，该实例会尝试连接到`localhost:8983/solr`的服务器上。下面的例子展示了如何注入一个Solr Bean。

```java
import java.io.IOException;

import org.apache.solr.client.solrj.SolrClient;
import org.apache.solr.client.solrj.SolrServerException;
import org.apache.solr.client.solrj.response.SolrPingResponse;

import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final SolrClient solr;

    public MyBean(SolrClient solr) {
        this.solr = solr;
    }

    public SolrPingResponse someMethod() throws SolrServerException, IOException {
        return this.solr.ping("users");
    }

}
```

如果你添加了你自己的`@Bean` 类型的`SolrClient`，它将取代默认的。

### 12.5. Elasticsearch

[Elasticsearch](https://www.elastic.co/products/elasticsearch)是一个开源、分布式、RESTful搜索和分析引擎。Spring Boot为Elasticsearch提供了基本的自动配置功能。

Spring Boot支持几个客户端。

* 官方Java "Low Level" 和 "High Level" REST客户端
* Spring Data Elasticsearch提供的`ReactiveElasticsearchClient`。

Spring Boot提供了一个专门的 "Starter"，`spring-boot-starter-data-elasticsearch`。

#### 12.5.1. 使用REST clients连接到Elasticsearch

Elasticsearch提供了[两个不同的REST客户端](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)，你可以用它们来查询集群："低级"客户端和 "高级"客户端。Spring Boot为 "高级" 客户端提供了支持，它与`org.elasticsearch.client:elasticsearch-rest-high-level-client`一起发布。

如果你在classpath上有这个依赖，Spring Boot会自动配置并注册一个`RestHighLevelClient`bean，默认目标是`localhost:9200`。你可以进一步调整`RestHighLevelClient`的配置方式，如以下例子所示。

```yaml
spring:
  elasticsearch:
    rest:
      uris: "https://search.example.com:9200"
      read-timeout: "10s"
      username: "user"
      password: "secret"
```

你也可以注册任意数量的实现了`RestClientBuilderCustomizer`的Bean，以便进行更高级的定制。要完全控制注册，请定义一个`RestClientBuilder`bean。

> 如果你的应用程序需要访问一个 "低级"的 `RestClient`，你可以通过调用`client.getLowLevelClient()` 在自动配置的`RestHighLevelClient`上得到它。

此外，如果`elasticsearch-rest-client-sniffer`在classpath上，`Sniffer`会被自动配置为自动从运行中的Elasticsearch集群中发现节点，并将它们设置为`RestHighLevelClient`bean。你可以进一步调整`Sniffer`的配置方式，如以下例子所示。

```yaml
spring:
  elasticsearch:
    rest:
      sniffer:
        interval: 10m
        delay-after-failure: 30s
```

#### 12.5.2. 使用Reactive REST客户端连接到Elasticsearch

[Spring Data Elasticsearch](https://spring.io/projects/spring-data-elasticsearch)提供了`ReactiveElasticsearchClient`，用于以响应的方式查询Elasticsearch实例。它建立在WebFlux的`WebClient`之上，所以`spring-boot-starter-elasticsearch`和`spring-boot-starter-webflux`这两个依赖项对于启用这种支持很有用。

默认情况下，Spring Boot会自动配置和注册一个`ReactiveElasticsearchClient`bean，目标是`localhost:9200`。你可以进一步调整它的配置方式，如下面的例子所示。

```yaml
spring:
  data:
    elasticsearch:
      client:
        reactive:
          endpoints: "search.example.com:9200"
          use-ssl: true
          socket-timeout: "10s"
          username: "user"
          password: "secret"
```

如果配置属性还不够，你想完全控制客户端配置，你可以注册一个自定义的`ClientConfiguration`bean。

#### 12.5.3. 通过使用Spring Data连接到Elasticsearch

要连接到Elasticsearch，必须定义一个`RestHighLevelClient`bean，由Spring Boot自动配置或由应用程序手动提供（见前几节）。有了这个配置，就可以像其他Spring Bean一样注入`ElasticsearchRestTemplate`，如下例所示。

```java
import org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final ElasticsearchRestTemplate template;

    public MyBean(ElasticsearchRestTemplate template) {
        this.template = template;
    }

    public boolean someMethod(String id) {
        return this.template.exists(id, User.class);
    }

}
```

在存在`spring-data-elasticsearch`和使用`WebClient`（通常是`spring-boot-starter-webflux`）所需的依赖时，Spring Boot还可以自动配置一个[ReactiveElasticsearchClient](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.elasticsearch.connecting-using-reactive-rest)和一个`ReactiveElasticsearchTemplate`作为bean。它们相当于其他REST客户端的响应式。

#### 12.5.4. Spring Data Elasticsearch Repositories

Spring Data包括对Elasticsearch的资源库支持。与前面讨论的JPA资源库一样，其基本原理是根据方法名称自动为你构建查询。

事实上，Spring Data JPA和Spring Data Elasticsearch都共享相同的公共基础设施。你可以用前面的JPA例子，假设`City`现在是Elasticsearch的`@Document`类，而不是JPA的`@Entity`，它的工作方式是一样的。

> 关于Spring Data Elasticsearch的完整细节，请参考[参考文档](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/)。

Spring Boot使用`ElasticsearchRestTemplate`或`ReactiveElasticsearchTemplate` Bean，支持经典和反应式Elasticsearch存储库。鉴于所需的依赖关系存在，这些 Bean子很可能是由Spring Boot自动配置的。

如果你想使用自己的模板来支持Elasticsearch repositories，你可以添加自己的`ElasticsearchRestTemplate`或`ElasticsearchOperations` `@Bean`，只要它被命名为`"elasticsearchTemplate"`。同样适用于`ReactiveElasticsearchTemplate`和`ReactiveElasticsearchOperations`，bean的名字为`"reactiveElasticsearchTemplate"`。

你可以通过以下属性选择禁用存储库支持。

```yaml
spring:
  data:
    elasticsearch:
      repositories:
        enabled: false
```

### 12.6. Cassandra

[Cassandra](https://cassandra.apache.org/)是一个开源的分布式数据库管理系统，旨在处理许多商品服务器上的大量数据。Spring Boot为Cassandra和[Spring Data Cassandra](https://github.com/spring-projects/spring-data-cassandra)提供的上面的抽象提供自动配置。有一个`spring-boot-starter-data-cassandra`的 "Starter"，用于以方便的方式收集依赖项。

#### 12.6.1. 连接到 Cassandra

你可以像对待其他Spring Bean一样，注入一个自动配置的`CassandraTemplate`或Cassandra`CqlSession`实例。`spring.data.cassandra.*`属性可用于定制连接。一般来说，你提供`keyspace-name`和`contact-points`以及本地数据中心名称，如下例所示。

```yaml
spring:
  data:
    cassandra:
      keyspace-name: "mykeyspace"
      contact-points: "cassandrahost1:9042,cassandrahost2:9042"
      local-datacenter: "datacenter1"
```

如果你的所有contact points的端口都是一样的，你可以使用一个快捷方式，只指定主机名，如下例所示。

```yaml
spring:
  data:
    cassandra:
      keyspace-name: "mykeyspace"
      contact-points: "cassandrahost1,cassandrahost2"
      local-datacenter: "datacenter1"
```

> 这两个例子是相同的，因为端口默认为`9042`。如果你需要配置端口，使用`spring.data.cassandra.port`。

Cassandra驱动有自己的配置基础结构，在classpath的根部加载一个`application.conf`。

Spring Boot默认不寻找这样的文件，但可以使用`spring.data.cassandra.config`加载一个。如果一个属性同时存在于`spring.data.cassandra.*`和配置文件中，则以`spring.data.cassandra.*`中的值为优先。

对于更高级的驱动定制，你可以注册任意数量的Bean来实现`DriverConfigLoaderBuilderCustomizer`。`CqlSession`可以用`CqlSessionBuilderCustomizer`类型的Bean来定制。

如果你使用`CqlSessionBuilder`来创建多个`CqlSession` bean，请记住该构建器是可变的，所以确保为每个会话注入一个新Session。

下面的代码列表显示了如何注入一个Cassandra Bean。

```java
import org.springframework.data.cassandra.core.CassandraTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final CassandraTemplate template;

    public MyBean(CassandraTemplate template) {
        this.template = template;
    }

    public long someMethod() {
        return this.template.count(User.class);
    }

}
```

如果你添加了你自己的`@Bean`类型的`CassandraTemplate`，它将取代默认的。

#### 12.6.2. Spring Data Cassandra Repositories

Spring Data包括对Cassandra的基本repository支持。目前，这比前面讨论的JPA repositories更为有限，需要用`@Query`来注释查找方法。

> 关于Spring Data Cassandra的完整细节，请参考[参考文档](https://docs.spring.io/spring-data/cassandra/docs/)。

### 12.7. Couchbase

[Couchbase](https://www.couchbase.com/)是一个开源的、分布式的、多模型的NoSQL面向文档的数据库，为交互式应用进行了优化。Spring Boot为Couchbase和[Spring Data Couchbase](https://github.com/spring-projects/spring-data-couchbase)提供的上面的抽象提供自动配置。有`spring-boot-starter-data-couchbase`和`spring-boot-starter-data-couchbase-reactive` "Starters"，以方便的方式收集依赖关系。

#### 12.7.1. 连接到 Couchbase

你可以通过添加Couchbase SDK和一些配置得到一个`Cluster`。`spring.couchbase.*`属性可以用来定制连接。一般来说，你提供[连接字符串](https://github.com/couchbaselabs/sdk-rfcs/blob/master/rfc/0011-connection-string.md)、username和password，如下例所示。

```yaml
spring:
  couchbase:
    connection-string: "couchbase://192.168.1.123"
    username: "user"
    password: "secret"
```

也可以定制一些 `ClusterEnvironment` 的设置。例如，下面的配置改变了打开一个新的 `Bucket` 所使用的超时时间，并启用了SSL支持。

```yaml
spring:
  couchbase:
    env:
      timeouts:
        connect: "3s"
      ssl:
        key-store: "/location/of/keystore.jks"
        key-store-password: "secret"
```

检查`spring.couchbase.env.*`属性以了解更多细节。为了获得更多的控制权，可以使用一个或多个`ClusterEnvironmentBuilderCustomizer` Bean。

#### 12.7.2. Spring Data Couchbase Repositories

Spring Data包括对Couchbase的存储库支持。关于Spring Data Couchbase的完整细节，请参考[参考文档](https://docs.spring.io/spring-data/couchbase/docs/4.2.3/reference/html/)。

你可以像对待其他Spring Bean一样注入一个自动配置的 `CouchbaseTemplate` 实例，前提是有一个 `CouchbaseClientFactory` Bean。这发生在如上所述的 `Cluster` 可用的情况下，并且已经指定了一个bucket的名称。

```yaml
spring:
  data:
    couchbase:
      bucket-name: "my-bucket"
```

下面的例子显示了如何注入一个`CouchbaseTemplate`bean。

```java
import org.springframework.data.couchbase.core.CouchbaseTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final CouchbaseTemplate template;

    public MyBean(CouchbaseTemplate template) {
        this.template = template;
    }

    public String someMethod() {
        return this.template.getBucketName();
    }

}
```

你可以在自己的配置中定义一些Bean，以覆盖自动配置所提供的Bean。

* 一个`CouchbaseMappingContext` `@Bean`，名称为`couchbaseMappingContext`。
* 一个`CustomConversions` `@Bean`，名称为`couchbaseCustomConversions`。
* 一个`CouchbaseTemplate` `@Bean`的名字是`couchbaseTemplate`。

为了避免在你自己的配置中硬编码这些名字，你可以重用Spring Data Couchbase提供的`BeanNames`。例如，你可以自定义要使用的转换器，如下所示。

```java
import org.assertj.core.util.Arrays;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.couchbase.config.BeanNames;
import org.springframework.data.couchbase.core.convert.CouchbaseCustomConversions;

@Configuration(proxyBeanMethods = false)
public class MyCouchbaseConfiguration {

    @Bean(BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
    public CouchbaseCustomConversions myCustomConversions() {
        return new CouchbaseCustomConversions(Arrays.asList(new MyConverter()));
    }

}
```

### 12.8. LDAP

[LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol) (Lightweight Directory Access Protocol)是一个开放的、供应商中立的、行业标准的应用协议，用于通过IP网络访问和维护分布式目录信息服务。Spring Boot为任何兼容的LDAP服务器提供自动配置，并支持[UnboundID](https://ldap.com/unboundid-ldap-sdk-for-java/)的嵌入式内存LDAP服务器。

LDAP的抽象由[Spring Data LDAP](https://github.com/spring-projects/spring-data-ldap)提供。有一个`spring-boot-starter-data-ldap`的 "Starter"，可以方便地收集依赖性。

#### 12.8.1. 连接到 LDAP Server

要连接到LDAP服务器，确保你声明对`spring-boot-starter-data-ldap` "Starter "或`spring-ldap-core`的依赖，然后在application.properties中声明服务器的URL，如下面的例子所示。

```yaml
spring:
  ldap:
    urls: "ldap://myserver:1235"
    username: "admin"
    password: "secret"
```

如果你需要定制连接设置，你可以使用`spring.ldap.base`和`spring.ldap.base-environment`属性。

`LdapContextSource`是基于这些设置自动配置的。如果`DirContextAuthenticationStrategy` Bean可用，它将与自动配置的`LdapContextSource`关联。如果你需要定制它，例如使用`PooledContextSource`，你仍然可以注入自动配置的`LdapContextSource`。确保将你定制的`ContextSource`标记为`@Primary`，以便自动配置的`LdapTemplate`使用它。

#### 12.8.2. Spring Data LDAP Repositories

Spring Data包括对LDAP的repository支持。关于Spring Data LDAP的完整细节，请参考[参考文档](https://docs.spring.io/spring-data/ldap/docs/1.0.x/reference/html/)。

你也可以像对待其他Spring Bean一样，注入一个自动配置的`LdapTemplate`实例，如下例所示。

```java
import java.util.List;

import org.springframework.ldap.core.LdapTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final LdapTemplate template;

    public MyBean(LdapTemplate template) {
        this.template = template;
    }

    public List<User> someMethod() {
        return this.template.findAll(User.class);
    }

}
```

#### 12.8.3. 嵌入式内存中的LDAP服务器

为测试目的，Spring Boot支持从[UnboundID](https://ldap.com/unboundid-ldap-sdk-for-java/)自动配置内存中的LDAP服务器。要配置该服务器，请添加对`com.unboundid:unboundid-ldapsdk`的依赖，并声明`spring.ldap.embedded.base-dn`属性，如下所示。

```yaml
spring:
  ldap:
    embedded:
      base-dn: "dc=spring,dc=io"
```

可以定义多个base-dn值，但是，由于区分的名称通常包含逗号，所以必须使用正确的符号来定义。

在yaml文件中，你可以使用yaml列表符号。在属性文件中，你必须将索引作为属性名称的一部分。

```yaml
spring.ldap.embedded.base-dn:
  - dc=spring,dc=io
  - dc=pivotal,dc=io
```

默认情况下，服务器在一个随机端口启动，并触发常规的LDAP支持。没有必要指定`spring.ldap.urls`属性。

如果你的classpath上有一个`schema.ldif`文件，它被用来初始化服务器。如果你想从一个不同的资源加载初始化脚本，你也可以使用`spring.ldap.embedded.ldif`属性。

默认情况下，一个标准的模式被用来验证`LDIF`文件。你可以通过设置`spring.ldap.embedded.validation.enabled`属性来完全关闭验证功能。如果你有自定义属性，你可以使用`spring.ldap.embedded.validation.schema`来定义你的自定义属性类型或对象类。

### 12.9. InfluxDB

[InfluxDB](https://www.influxdata.com/)是一个开源的时间序列数据库，为快速、高可用性地存储和检索运营监测、应用指标、物联网传感器数据和实时分析等领域的时间序列数据而优化。

#### 12.9.1. Connecting to InfluxDB

只要`influxdb-java`客户端在classpath上，并且设置了数据库的URL，Spring Boot就会自动配置一个`InfluxDB`实例，如下例所示。

```yaml
spring:
  influx:
    url: "https://172.0.0.1:8086"
```

如果与InfluxDB的连接需要用户和密码，你可以相应地设置`spring.influx.user`和`spring.influx.password`属性。

InfluxDB依赖于OkHttp。如果你需要调整`InfluxDB`在幕后使用的http客户端，你可以注册一个`InfluxDbOkHttpClientBuilderProvider`bean。

如果你需要对配置进行更多的控制，可以考虑注册一个`InfluxDbCustomizer`bean。

## 13 缓存

Spring框架提供了对透明地添加缓存到应用程序的支持。在其核心部分，该抽象将缓存应用于方法，从而根据缓存中的可用信息减少执行的次数。缓存逻辑的应用是透明的，对调用者没有任何干扰。只要通过 `@EnableCaching` 注解启用缓存支持，Spring Boot就会自动配置缓存基础设施。

> 请查看Spring框架参考文献中的[相关章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/integration.html#cache)以了解更多细节。

简而言之，为了给你的服务的某个操作添加缓存，请在其方法中添加相关的注解，如下面的例子所示。

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Component;

@Component
public class MyMathService {

    @Cacheable("piDecimals")
    public int computePiDecimal(int precision) {
        ...
    }

}
```

这个例子演示了在一个潜在的高成本操作中使用缓存。在调用 `computeDecimal` 之前，该抽象概念在 `piDecimals` 缓存中寻找与 `i` 参数匹配的条目。如果找到一个条目，缓存中的内容将立即返回给调用者，并且不调用该方法。否则，该方法被调用，并在返回值之前更新缓存。

> 你也可以透明地使用标准的JSR-107（JCache）注解（如`@CacheResult`）。然而，我们强烈建议你不要混合使用Spring Cache和JCache注解。

如果你没有添加任何特定的缓存库，Spring Boot会自动配置一个[simple provider](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.simple)，在内存中concurrent map。当需要缓存时（比如前面例子中的`piDecimals`），这个提供者会为你创建缓存。简单的提供者并不推荐在生产中使用，但它对于开始使用并确保你理解这些功能是非常好的。当你决定使用哪个缓存提供程序时，请确保阅读它的文档，以弄清楚如何配置你的应用程序使用的缓存。几乎所有的提供者都要求你明确地配置你在应用程序中使用的每一个缓冲区。有些提供了一种方法来定制由`spring.cache.cache-names`属性定义的默认缓存。

> 也可以透明地从缓存中[更新](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/integration.html#cache-annotations-put)或[驱逐](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/integration.html#cache-annotations-evict)数据。

### 13.1. 支持的缓存

缓存抽象不提供实际的存储，而是依赖于由`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`接口物化的抽象。

如果你没有定义一个`CacheManager`类型的bean或一个名为`cacheResolver`的`CacheResolver`（见[`CachingConfigurer`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/cache/annotation/CachingConfigurer.html)），Spring Boot会尝试检测以下提供者（按所示顺序）。

1. [Generic](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.generic)
2. [JCache (JSR-107)](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.jcache) (EhCache 3, Hazelcast, Infinispan, and others)
3. [EhCache 2.x](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.ehcache2)
4. [Hazelcast](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.hazelcast)
5. [Infinispan](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.infinispan)
6. [Couchbase](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.couchbase)
7. [Redis](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.redis)
8. [Caffeine](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.caffeine)
9. [Simple](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.simple)

> 也可以通过设置`spring.cache.type`属性来*强制*一个特定的缓存提供者。如果你需要在某些环境下（如测试）[完全禁用缓存](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.none)，请使用该属性。

使用`spring-boot-starter-cache` "Starter "来快速添加基本的缓存依赖项。该启动器带来了`spring-context-support`。如果你手动添加依赖项，你必须包括`spring-context-support`才能使用JCache、EhCache 2.x或Caffeine支持。

如果 `CacheManager` 是由Spring Boot自动配置的，你可以通过暴露一个实现 `CacheManagerCustomizer` 接口的bean，在它完全初始化之前进一步调整其配置。下面的例子设置了一个标志，表示 `null` 值应该被传递给底层Map。

```java
import org.springframework.boot.autoconfigure.cache.CacheManagerCustomizer;
import org.springframework.cache.concurrent.ConcurrentMapCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyCacheManagerConfiguration {

    @Bean
    public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
        return (cacheManager) -> cacheManager.setAllowNullValues(false);
    }

}
```

> 在前面的例子中，预计会有一个自动配置的`ConcurrentMapCacheManager`。如果不是这样（要么是你提供了自己的配置，要么是自动配置了不同的缓存提供者），定制器根本就不会被调用。你可以有任意多的自定义器，你也可以通过使用`@Order`或`Ordered`对它们进行排序。

#### 13.1.1. 通用的

如果上下文至少定义了一个`org.springframework.cache.Cache` bean，就会使用通用缓存。一个包裹所有该类型的Bean的`CacheManager`被创建。

#### 13.1.2. JCache (JSR-107)

[JCache](https://jcp.org/en/jsr/detail?id=107)通过classpath上存在的`javax.cache.spi.CachingProvider`进行引导（也就是说，classpath上存在一个符合JSR-107的缓存库），`JCacheCacheManager`由`spring-boot-starter-cache` "Starter "提供。有各种兼容的库，Spring Boot为Ehcache 3、Hazelcast和Infinispan提供依赖性管理。也可以添加任何其他兼容的库。

可能会出现不止一个提供者的情况，在这种情况下，必须明确指定提供者。即使JSR-107标准没有强制要求以标准化的方式定义配置文件的位置，Spring Boot也会尽力适应设置缓存的实现细节，如下面的例子所示。

```yaml
# Only necessary if more than one provider is present
spring:
  cache:
    jcache:
      provider: "com.example.MyCachingProvider"
      config: "classpath:example.xml"
```

当一个缓存库同时提供本地实现和JSR-107支持时，Spring Boot更倾向于JSR-107支持，这样，如果你切换到不同的JSR-107实现，同样的功能也可以使用。

> Spring Boot有[对Hazelcast的一般支持](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.hazelcast)。如果有一个`HazelcastInstance`，它也会自动被`CacheManager`重用，除非指定`spring.cache.jcache.config`属性。

有两种方法来定制底层的`javax.cache.cacheManager`。

* 通过设置`spring.cache.cache-names`属性，可以在启动时创建缓存。如果定义了一个自定义的`javax.cache.configuration.Configuration` Bean，就可以用来定制它们。
* `org.springframework.boot.autoconfigure.cache.JCacheManagerCustomizer` Bean 被调用，并引用了`CacheManager`的引用，以实现完全定制。

如果定义了一个标准的`javax.cache.CacheManager` Bean，它将被自动包装在抽象所期望的`org.springframework.cache.CacheManager`实现中。没有进一步的定制应用于它。

#### 13.1.3. EhCache 2.x

[EhCache](https://www.ehcache.org/) 2.x被使用，如果在classpath的根部能找到一个名为`ehcache.xml`的文件。如果找到EhCache 2.x，则使用`spring-boot-starter-cache` "Starter "提供的`EhCacheCacheManager`来引导 cache manager。也可以提供一个备用的配置文件，如下面的例子所示。

```yaml
spring:
  cache:
    ehcache:
      config: "classpath:config/another-config.xml"
```

#### 13.1.4. Hazelcast

Spring Boot有[对Hazelcast的一般支持](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.hazelcast)。如果一个 `HazelcastInstance` 被自动配置了，它就会被自动包装成一个 `CacheManager`。

#### 13.1.5. Infinispan

[Infinispan](https://infinispan.org/)没有默认的配置文件位置，所以必须明确地指定它。否则，将使用默认的引导文件。

```yaml
spring:
  cache:
    infinispan:
      config: "infinispan.xml"
```

缓存可以通过设置 `spring.cache.cache-names` 属性在启动时创建。如果定义了一个自定义的`ConfigurationBuilder`bean，它将被用来定制缓存。

> Spring Boot中对Infinispan的支持仅限于嵌入式模式，而且是相当基本的。如果你想要更多的选项，你应该使用官方的Infinispan Spring Boot启动器来代替。更多细节请参见[Infinispan的文档](https://github.com/infinispan/infinispan-spring-boot)。

#### 13.1.6. Couchbase

如果Spring Data Couchbase可用并且Couchbase被[配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.couchbase)，则会自动配置一个`CouchbaseCacheManager`。通过设置`spring.cache.cache-names`属性可以在启动时创建额外的缓存，并且可以通过使用`spring.cache.couchbase.*`属性配置缓存默认值。例如，下面的配置创建了`cache1`和`cache2`缓存，条目*有效期*为10分钟。

```yaml
spring:
  cache:
    cache-names: "cache1,cache2"
    couchbase:
      expiration: "10m"
```

如果你需要对配置进行更多的控制，可以考虑注册一个`CouchbaseCacheManagerBuilderCustomizer` Bean。下面的例子显示了一个自定义器，它为`cache1`和`cache2`配置了一个特定的条目到期时间。

```java
import java.time.Duration;

import org.springframework.boot.autoconfigure.cache.CouchbaseCacheManagerBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.couchbase.cache.CouchbaseCacheConfiguration;

@Configuration(proxyBeanMethods = false)
public class MyCouchbaseCacheManagerConfiguration {

    @Bean
    public CouchbaseCacheManagerBuilderCustomizer myCouchbaseCacheManagerBuilderCustomizer() {
        return (builder) -> builder
                .withCacheConfiguration("cache1", CouchbaseCacheConfiguration
                        .defaultCacheConfig().entryExpiry(Duration.ofSeconds(10)))
                .withCacheConfiguration("cache2", CouchbaseCacheConfiguration
                        .defaultCacheConfig().entryExpiry(Duration.ofMinutes(1)));

    }

}
```

#### 13.1.7. Redis

如果[Redis](https://redis.io/)可用且已配置，则会自动配置一个`RedisCacheManager`。可以通过设置`spring.cache.cache-names`属性在启动时创建额外的缓存，缓存默认值可以通过使用`spring.cache.redis.*`属性来配置。例如，下面的配置创建了`cache1`和`cache2`缓存，*生存时间*为10分钟。

```yaml
spring:
  cache:
    cache-names: "cache1,cache2"
    redis:
      time-to-live: "10m"
```

默认情况下，会添加一个键的前缀，这样，如果两个独立的缓存使用同一个键，Redis就不会有重叠的键，也不能返回无效的值。如果你创建了自己的 `RedisCacheManager`，我们强烈建议保持这个设置。

> 你可以通过添加你自己的`RedisCacheConfiguration` `@Bean`来完全控制默认配置。如果你想定制默认的序列化策略，这很有用。

如果你需要对配置进行更多的控制，可以考虑注册一个`RedisCacheManagerBuilderCustomizer`bean。下面的例子显示了一个自定义器，它为`cache1`和`cache2`配置了一个特定的生存时间。

```java
import java.time.Duration;

import org.springframework.boot.autoconfigure.cache.RedisCacheManagerBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;

@Configuration(proxyBeanMethods = false)
public class MyRedisCacheManagerConfiguration {

    @Bean
    public RedisCacheManagerBuilderCustomizer myRedisCacheManagerBuilderCustomizer() {
        return (builder) -> builder
                .withCacheConfiguration("cache1", RedisCacheConfiguration
                        .defaultCacheConfig().entryTtl(Duration.ofSeconds(10)))
                .withCacheConfiguration("cache2", RedisCacheConfiguration
                        .defaultCacheConfig().entryTtl(Duration.ofMinutes(1)));

    }

}
```

#### 13.1.8. Caffeine

[Caffeine](https://github.com/ben-manes/caffeine)是Java 8对Guava缓存的重写，取代了对Guava的支持。如果Caffeine存在，一个`CaffeineCacheManager`（由`spring-boot-starter-cache` "Starter "提供）会自动配置。缓存可以通过设置`spring.cache.cache-names`属性在启动时创建，并可以通过以下方式之一进行定制（按指定顺序）。

1. 一个由`spring.cache.caffeine.spec`定义的缓存规范。
2. 定义了一个`com.github.benmanes.caffeine.cache.CaffeineSpec` Beam
3. 定义了一个`com.github.benmanes.caffeine.cache.Caffeine` Bean

例如，以下配置创建了 `cache1` 和 `cache2` 缓存，最大容量为500，*生存时间*为10分钟

```yaml
spring:
  cache:
    cache-names: "cache1,cache2"
    caffeine:
      spec: "maximumSize=500,expireAfterAccess=600s"
```

如果定义了一个`com.github.benmanes.caffeine.cache.CacheLoader` Bean，它将自动与`CaffeineCacheManager`相关联。由于 `CacheLoader` 将与缓存管理器管理的所有缓存相关联，它必须被定义为 `CacheLoader<Object, Object>`。自动配置会忽略任何其他的通用类型。

#### 13.1.9. 简单的实现

如果找不到其他的提供者，就配置一个简单的实现，使用`ConcurrentHashMap`作为缓存存储。如果你的应用程序中没有缓存库，这就是默认的。默认情况下，缓存是根据需要创建的，但你可以通过设置`cache-names`属性来限制可用的缓存列表。例如，如果你只想要`cache1`和`cache2`缓存，设置`cache-names`属性如下。

```yaml
spring:
  cache:
    cache-names: "cache1,cache2"
```

如果你这样做了，而你的应用程序使用了一个没有列出的缓存，那么在运行时需要缓存时就会失败，但在启动时不会。这与 "真正的" 缓存提供者在你使用未声明的缓存时的行为类似。

#### 13.1.10. None

当`@EnableCaching` 出现在你的配置中时，预计也会有一个合适的缓存配置。如果你需要在某些环境中完全禁用缓存，请将缓存类型强制为`none`，以使用一个无操作的实现，如下例所示。

```yaml
spring:
  cache:
    type: "none"
```

## 14. Messaging

Spring框架为集成消息系统提供了广泛的支持，从使用`JmsTemplate`简化JMS API的使用到异步接收消息的完整基础设施。Spring AMQP为高级消息队列协议提供了一个类似的功能集。Spring Boot还为`RabbitTemplate`和RabbitMQ提供了自动配置选项。Spring WebSocket原生包括对STOMP消息传递的支持，Spring Boot通过启动器和少量的自动配置对其进行支持。Spring Boot还支持Apache Kafka。

### 14.1. JMS

`javax.jms.ConnectionFactory`接口提供了一个创建`javax.jms.Connection`的标准方法，用于与JMSbroker进行交互。尽管Spring需要一个`ConnectionFactory`来与JMS一起工作，但你一般不需要自己直接使用它，而是可以依赖更高级别的消息传递抽象。(详见Spring框架参考文档的[相关部分](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/integration.html#jms))。Spring Boot还自动配置了必要的基础设施来发送和接收消息。

#### 14.1.1. ActiveMQ Support

当[ActiveMQ](https://activemq.apache.org/)在classpath上可用时，Spring Boot也可以配置一个`ConnectionFactory`。如果broker存在，就会自动启动和配置一个嵌入式broker（前提是没有通过配置指定broker 的URL）。

> 如果你使用`spring-boot-starter-activemq`，就会提供连接或嵌入ActiveMQ实例的必要依赖，以及与JMS集成的Spring基础设施。

ActiveMQ的配置是由`spring.activemq.*`中的外部配置属性控制。例如，你可以在`application.properties`中声明以下部分。

```yaml
spring:
  activemq:
    broker-url: "tcp://192.168.1.210:9876"
    user: "admin"
    password: "secret"
```

默认情况下，`CachingConnectionFactory`用合理的设置包装了本地的`ConnectionFactory`，你可以通过`spring.jms.*`的外部配置属性来控制。

```yaml
spring:
  jms:
    cache:
      session-cache-size: 5
```

如果你想使用native pooling，你可以通过添加`org.messaginghub:pooled-jms`的依赖关系，并相应配置`JmsPoolConnectionFactory`来实现，如下面的例子所示。

```yaml
spring:
  activemq:
    pool:
      enabled: true
      max-connections: 50
```

> 更多支持的选项请参见[`ActiveMQProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java)。你也可以注册任意数量的实现了`ActiveMQConnectionFactoryCustomizer`的Bean，以便进行更高级的定制。

默认情况下，如果destination还不存在，ActiveMQ会创建一个destination，以便根据其提供的名称来解析destination。

#### 14.1.2. ActiveMQ Artemis支持

当Spring Boot检测到classpath上有[ActiveMQ Artemis](https://activemq.apache.org/components/artemis/)时，它可以自动配置一个`ConnectionFactory`。如果broker 存在，就会自动启动和配置一个嵌入式broker （除非明确设置了模式属性）。支持的模式有`embedded`（明确指出需要嵌入式broker，如果broker在classpath上不可用，就会发生错误）和`native`（使用`netty`传输协议连接到broker）。当配置了后者时，Spring Boot会配置一个`ConnectionFactory`，以默认设置连接到本地机器上运行的broker。

> 如果你使用`spring-boot-starter-artemis`，就会提供连接到现有的ActiveMQ Artemis实例的必要依赖，以及与JMS集成的Spring基础设施。添加`org.apache.activemq:artemis-jms-server`到你的应用程序，可以使用嵌入式模式。

ActiveMQ Artemis的配置是由`spring.artemis.*`中的外部配置属性控制。例如，你可以在`application.properties`中声明以下部分。

```yaml
spring:
  artemis:
    mode: native
    broker-url: "tcp://192.168.1.210:9876"
    user: "admin"
    password: "secret"
```

在嵌入broker时，你可以选择是否要启用持久性，并列出应该被提供的目的地。这些可以指定为逗号分隔的列表，用默认选项创建，或者你可以定义`org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration`或`org.apache.activemq.artemis.jms.server.config.TopicConfiguration`类型的bean，分别用于高级队列和主题配置。

默认情况下，`CachingConnectionFactory`用合理的设置包装了本地的`ConnectionFactory`，你可以通过`spring.jms.*`中的外部配置属性控制。

```yaml
spring:
  jms:
    cache:
      session-cache-size: 5
```

如果你想使用native pooling，你可以通过添加`org.messaginghub:pooled-jms`的依赖关系，并相应配置`JmsPoolConnectionFactory`来实现，如下面的例子所示。

```yaml
spring:
  artemis:
    pool:
      enabled: true
      max-connections: 50
```

更多支持的选项请参见[`ArtemisProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/artemis/ArtemisProperties.java)。

不涉及JNDI查询，目的地是根据它们的名字来解决的，使用Artemis配置中的`name`属性或通过配置提供的名字。

#### 14.1.3. 使用JNDI ConnectionFactory

如果你在应用服务器中运行你的应用程序，Spring Boot会尝试使用JNDI来定位JMS `ConnectionFactory`。默认情况下，会检查`java:/JmsXA`和`java:/XAConnectionFactory`位置。如果你需要指定另一个位置，你可以使用`spring.jms.jndi-name`属性，如下面的例子所示。

```yaml
spring:
  jms:
    jndi-name: "java:/MyConnectionFactory"
```

#### 14.1.4. 发送消息

Spring的`JmsTemplate`是自动配置的，你可以将其直接自动连接到你自己的Bean中，如下例所示。

```java
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JmsTemplate jmsTemplate;

    public MyBean(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }

    public void someMethod() {
        this.jmsTemplate.convertAndSend("hello");
    }

}
```

[`JmsMessagingTemplate`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/jms/core/JmsMessagingTemplate.html)可以以类似方式注入。如果定义了`DestinationResolver`或`MessageConverter`bean，它将自动与自动配置的`JmsTemplate`关联。

#### 14.1.5. 接收消息

当JMS基础设施存在时，任何Bean都可以用`@JmsListener`来注释，以创建一个监听器端点。如果没有定义`JmsListenerContainerFactory`，会自动配置一个默认的。如果定义了`DestinationResolver`、`MessageConverter`或`javax.jms.ExceptionListener` Bean，它们会自动与默认工厂关联。

默认情况下，默认工厂是事务性的。如果你运行在一个有`JtaTransactionManager`的基础设施中，它就会默认与监听器容器相关联。如果没有，`sessionTransacted` 标志将被启用。在后一种情况下，你可以通过在你的监听器方法（或其委托）上添加`@Transactional`，将你的本地数据存储事务与传入消息的处理联系起来。这可以确保在本地事务完成后，传入的消息被确认。这也包括发送在同一JMS会话中执行的响应消息。

下面的组件在`someQueue`目标上创建了一个监听器端点。

```java
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @JmsListener(destination = "someQueue")
    public void processMessage(String content) {
        // ...
    }

}
```

> 更多细节请参见[`@EnableJms`的Javadoc](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/jms/annotation/EnableJms.html)。

如果你需要创建更多的`JmsListenerContainerFactory`实例，或者你想覆盖默认值，Spring Boot提供了一个`DefaultJmsListenerContainerFactoryConfigurer`，你可以用它来初始化一个`DefaultJmsListenerContainerFactory`，设置与自动配置的那个相同。

例如，下面的例子暴露了另一个使用特定`MessageConverter`的工厂。

```java
import javax.jms.ConnectionFactory;

import org.springframework.boot.autoconfigure.jms.DefaultJmsListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;

@Configuration(proxyBeanMethods = false)
public class MyJmsConfiguration {

    @Bean
    public DefaultJmsListenerContainerFactory myFactory(DefaultJmsListenerContainerFactoryConfigurer configurer) {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        ConnectionFactory connectionFactory = getCustomConnectionFactory();
        configurer.configure(factory, connectionFactory);
        factory.setMessageConverter(new MyMessageConverter());
        return factory;
    }

    private ConnectionFactory getCustomConnectionFactory() {
        return ...
    }

}
```

然后你可以在任何`@JmsListener`注释的方法中使用该工厂，如下所示。

```java
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @JmsListener(destination = "someQueue", containerFactory = "myFactory")
    public void processMessage(String content) {
        // ...
    }

}
```

### 14.2. AMQP

高级消息队列协议（AMQP）是一个平台中立、面向消息的中间件的线级协议。Spring AMQP项目将Spring的核心概念应用于基于AMQP的消息传递解决方案的开发。Spring Boot为通过RabbitMQ与AMQP合作提供了一些便利，包括`spring-boot-starter-amqp` "Starter"。

#### 14.2.1. RabbitMQ support

[RabbitMQ](https://www.rabbitmq.com/)是一个基于AMQP协议的轻量级、可靠、可扩展、可移植的消息broker。Spring使用`RabbitMQ`来通过AMQP协议进行通信。

RabbitMQ 的配置由 `spring.rabbitmq.*` 的外部配置属性控制。例如，你可以在`application.properties`中声明以下部分。

```yaml
spring:
  rabbitmq:
    host: "localhost"
    port: 5672
    username: "admin"
    password: "secret"
```

另外，你也可以使用`addresses`属性配置相同的连接。

```yaml
spring:
  rabbitmq:
    addresses: "amqp://admin:secret@localhost"
```

> 当以这种方式指定地址时，`host`和`port`属性会被忽略。如果地址使用`amqps`协议，SSL支持将自动启用。

请参阅 [`RabbitProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java) 以了解更多支持的基于属性的配置选项。要配置 Spring AMQP 使用的 RabbitMQ `ConnectionFactory` 的低级细节，请定义一个`ConnectionFactoryCustomizer` Bean。

如果上下文中存在 `ConnectionNameStrategy` Bean，它将被自动用于命名由自动配置的 `CachingConnectionFactory` 创建的连接。

> 请参阅 [Understanding AMQP, the protocol used by RabbitMQ](https://spring.io/blog/2010/06/14/understanding-amqp-the-protocol-used-by-rabbitmq/) 以了解更多细节。

#### 14.2.2. 发送消息

Spring的`AmqpTemplate`和`AmqpAdmin`是自动配置的，你可以将它们直接自动连接到你自己的Bean中，如下例所示。

```java
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final AmqpAdmin amqpAdmin;

    private final AmqpTemplate amqpTemplate;

    public MyBean(AmqpAdmin amqpAdmin, AmqpTemplate amqpTemplate) {
        this.amqpAdmin = amqpAdmin;
        this.amqpTemplate = amqpTemplate;
    }

    public void someMethod() {
        this.amqpAdmin.getQueueInfo("someQueue");
    }

    public void someOtherMethod() {
        this.amqpTemplate.convertAndSend("hello");
    }

}
```

> [`RabbitMessagingTemplate`](https://docs.spring.io/spring-amqp/docs/2.3.10/api/org/springframework/amqp/rabbit/core/RabbitMessagingTemplate.html)可以以类似方式注入。如果定义了一个`MessageConverter` bean，它就会自动关联到自动配置的`AmqpTemplate`。

如果有必要，任何定义为 Bean 的 `org.springframework.amqp.core.Queue` 都会自动用于在 RabbitMQ 实例上声明一个相应的队列。

为了重试操作，您可以在 `AmqpTemplate` 上启用重试（例如，在broker连接丢失的情况下）。

```yaml
spring:
  rabbitmq:
    template:
      retry:
        enabled: true
        initial-interval: "2s"
```

默认情况下，重试是禁用的。你也可以通过声明一个`RabbitRetryTemplateCustomizer`bean，以编程方式定制`RetryTemplate`。

如果你需要创建更多的`RabbitTemplate`实例，或者你想覆盖默认值，Spring Boot提供了一个`RabbitTemplateConfigurer`bean，你可以用它来初始化`RabbitTemplate`，设置与自动配置使用的工厂相同。

#### 14.2.3. 接收消息

当Rabbit基础设施存在时，任何Bean都可以用`@RabbitListener`来注释以创建一个监听器端点。如果没有定义`RabbitListenerContainerFactory`，就会自动配置一个默认的`SimpleRabbitListenerContainerFactory`，你可以使用`spring.rabbitmq.listener.type`属性切换到一个直接容器。如果定义了一个`MessageConverter`或`MessageRecoverer` bean，它将自动与默认工厂相关联。

下面的示例组件在`someQueue`队列上创建了一个监听器端点。

```java
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @RabbitListener(queues = "someQueue")
    public void processMessage(String content) {
        // ...
    }

}
```

> 更多细节请参见[`@EnableRabbit`的Javadoc](https://docs.spring.io/spring-amqp/docs/2.3.10/api/org/springframework/amqp/rabbit/annotation/EnableRabbit.html)。

如果你需要创建更多的`RabbitListenerContainerFactory`实例，或者你想覆盖默认值，Spring Boot提供了一个`SimpleRabbitListenerContainerFactoryConfigurer`和`DirectRabbitListenerContainerFactoryConfigurer`，你可以用它来初始化`SimpleRabbitListenerContainerFactory`和`DirectRabbitListenerContainerFactory`，设置与自动配置使用的工厂一样。

> 你选择哪种容器类型并不重要。这两个Bean是由自动配置暴露出来的。

例如，下面的配置类暴露了另一个使用特定`MessageConverter`的工厂。

```java
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyRabbitConfiguration {

    @Bean
    public SimpleRabbitListenerContainerFactory myFactory(SimpleRabbitListenerContainerFactoryConfigurer configurer) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        ConnectionFactory connectionFactory = getCustomConnectionFactory();
        configurer.configure(factory, connectionFactory);
        factory.setMessageConverter(new MyMessageConverter());
        return factory;
    }

    private ConnectionFactory getCustomConnectionFactory() {
        return ...
    }

}
```

你可以启用重试以处理你的监听器抛出异常的情况。默认情况下，使用`RejectAndDontRequeueRecoverer`，但你可以自己定义一个`MessageRecoverer`。当重试用尽时，消息会被拒绝，或者被丢弃，或者被路由到一个死信交换点，如果broker被配置为这样做的话。默认情况下，重试被禁用。你也可以通过声明一个`RabbitRetryTemplateCustomizer`bean，以编程方式定制`RetryTemplate`。

默认情况下，如果重试被禁用，并且监听器抛出了一个异常，那么将无限期地重试交付。你可以通过两种方式修改这种行为。将 `defaultRequeueRejected` 属性设置为 `false`，以便尝试零次重发，或者抛出一个 `AmqpRejectAndDontRequeueException`，以示该消息应被拒绝。后者是在启用重试和达到最大发送尝试次数时使用的机制。

### 14.3. Apache Kafka Support

[Apache Kafka](https://kafka.apache.org/)通过提供`spring-kafka`项目的自动配置来支持。

Kafka配置是由`spring.kafka.*`中的外部配置属性控制的。例如，你可以在`application.properties`中声明以下部分。

```yaml
spring:
  kafka:
    bootstrap-servers: "localhost:9092"
    consumer:
      group-id: "myGroup"
```

要在启动时创建一个主题，需要添加一个`NewTopic`类型的Bean。如果该主题已经存在，该Bean将被忽略。

> 更多支持的选项见[`KafkaProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/kafka/KafkaProperties.java)。

#### 14.3.1. 发送消息

Spring的`KafkaTemplate`是自动配置的，你可以在自己的Bean中直接自动连接它，如下例所示。

```java
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final KafkaTemplate<String, String> kafkaTemplate;

    public MyBean(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void someMethod() {
        this.kafkaTemplate.send("someTopic", "Hello");
    }

}
```

如果定义了`spring.kafka.producer.transaction-id-prefix`属性，就会自动配置一个`KafkaTransactionManager`。此外，如果定义了`RecordMessageConverter`bean，它将自动与自动配置的`KafkaTemplate`相关联。

#### 14.3.2. 接收消息

当Apache Kafka基础设施存在时，任何Bean都可以用`@KafkaListener`来注释，以创建一个监听器端点。如果没有定义`KafkaListenerContainerFactory`，就会自动配置一个默认的、在`spring.kafka.listener.*`中定义的键。

下面的组件在`someTopic`主题上创建了一个监听器端点。

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @KafkaListener(topics = "someTopic")
    public void processMessage(String content) {
        // ...
    }

}
```

如果定义了 `KafkaTransactionManager` Bean，它将自动与容器工厂相关联。同样，如果定义了`RecordFilterStrategy`、`ErrorHandler`、`AfterRollbackProcessor`或`ConsumerAwareRebalanceListener` bean，它将自动与默认工厂相关联。

根据监听器的类型，`RecordMessageConverter` 或 `BatchMessageConverter` bean被关联到默认工厂。如果批处理监听器只有一个`RecordMessageConverter` bean，它将被包裹在一个`BatchMessageConverter`中。

> 自定义的`ChainedKafkaTransactionManager`必须被标记为`@Primary`，因为它通常引用自动配置的`KafkaTransactionManager`bean。

#### 14.3.3. Kafka Streams

Spring for Apache Kafka提供了一个工厂bean来创建`StreamsBuilder`对象并管理其流的生命周期。只要`kafka-streams`在classpath上并且通过`@EnableKafkaStreams`注解启用Kafka Streams，Spring Boot就会自动配置所需的`KafkaStreamsConfiguration`bean。

启用Kafka Streams意味着必须设置应用ID和引导服务器。前者可以使用`spring.kafka.streams.application-id`进行配置，如果没有设置，则默认为`spring.application.name`。后者可以全局设置，也可以只对流进行特别重写。

一些额外的属性可以使用专用属性；其他任意的Kafka属性可以使用`spring.kafka.streams.properties`命名空间来设置。更多信息请参见[Additional Kafka Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.messaging.kafka.additional-properties)。

要使用工厂Bean，请将`StreamsBuilder`接入你的`@Bean`，如下例所示。

```java
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.Produced;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafkaStreams;
import org.springframework.kafka.support.serializer.JsonSerde;

@Configuration(proxyBeanMethods = false)
@EnableKafkaStreams
public class MyKafkaStreamsConfiguration {

    @Bean
    public KStream<Integer, String> kStream(StreamsBuilder streamsBuilder) {
        KStream<Integer, String> stream = streamsBuilder.stream("ks1In");
        stream.map(this::uppercaseValue).to("ks1Out", Produced.with(Serdes.Integer(), new JsonSerde<>()));
        return stream;
    }

    private KeyValue<Integer, String> uppercaseValue(Integer key, String value) {
        return new KeyValue<>(key, value.toUpperCase());
    }

}
```

默认情况下，其创建的`StreamBuilder`对象所管理的流会自动启动。你可以使用`spring.kafka.streams.auto-startup`属性来定制这一行为。

#### 14.3.4. 额外的Kafka属性

自动配置支持的属性显示在[application-perties.html](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties)。注意，在大多数情况下，这些属性（连字符或camelCase）直接映射到Apache Kafka的点状属性。详情请参考Apache Kafka文档。

这些属性中的前几个适用于所有组件（生产者、消费者、管理员和流），但如果你想使用不同的值，可以在组件级别指定。Apache Kafka将属性的重要性指定为高、中、低。Spring Boot自动配置支持所有高重要性属性、一些选定的中度和低度属性，以及没有默认值的任何属性。

只有Kafka支持的一部分属性可以通过`KafkaProperties`类直接使用。如果你想用不直接支持的额外属性来配置生产者或消费者，请使用以下属性。

```yaml
spring:
  kafka:
    properties:
      "[prop.one]": "first"
    admin:
      properties:
        "[prop.two]": "second"
    consumer:
      properties:
        "[prop.three]": "third"
    producer:
      properties:
        "[prop.four]": "fourth"
    streams:
      properties:
        "[prop.five]": "fifth"
```

这将普通的`prop.one`Kafka属性设置为`first`（适用于生产者、消费者和管理员），`prop.two`管理员属性为`second`，`prop.three`消费者属性为`third`，`prop.four`生产者属性为`fourth`，`prop.five`streams属性为`fifth`。

你也可以按以下方式配置Spring Kafka的`JsonDeserializer`。

```yaml
spring:
  kafka:
    consumer:
      value-deserializer: "org.springframework.kafka.support.serializer.JsonDeserializer"
      properties:
        "[spring.json.value.default.type]": "com.example.Invoice"
        "[spring.json.trusted.packages]": "com.example.main,com.example.another"
```

同样地，你可以禁用`JsonSerializer`默认行为，即在头文件中发送类型信息。

```yaml
spring:
  kafka:
    producer:
      value-serializer: "org.springframework.kafka.support.serializer.JsonSerializer"
      properties:
        "[spring.json.add.type.headers]": false
```

> 以这种方式设置的属性将覆盖Spring Boot明确支持的任何配置项目。

#### 14.3.5. 使用嵌入式Kafka测试

Spring for Apache Kafka提供了一种方便的方式来测试带有嵌入式Apache Kafka broker的项目。要使用这个功能，请用`spring-kafka-test`模块中的`@EmbeddedKafka`注释一个测试类。欲了解更多信息，请参见Spring for Apache Kafka [参考手册](https://docs.spring.io/spring-kafka/docs/2.7.4/reference/html/#embedded-kafka-annotation)。

为了让Spring Boot自动配置与上述嵌入式Apache Kafka broker一起工作，你需要将嵌入式 broker地址的系统属性（由`EmbeddedKafkaBroker`填充）重新映射为Apache Kafka的Spring Boot配置属性。有几种方法可以做到这一点。

* 提供一个系统属性，将嵌入式代理地址映射到测试类中的`spring.kafka.bootstrap-servers`。

  ```java
  static {
      System.setProperty(EmbeddedKafkaBroker.BROKER_LIST_PROPERTY, "spring.kafka.bootstrap-servers");
  }
  ```

* 在`@EmbeddedKafka`注解上配置一个属性名称。

  ```java
  @SpringBootTest
  @EmbeddedKafka(topics = "someTopic", bootstrapServersProperty = "spring.kafka.bootstrap-servers")
  class MyTest {

      // ...

  }
  ```

* 在配置属性中使用一个占位符。

  ```yaml
  spring:
    kafka:
      bootstrap-servers: "${spring.embedded.kafka.brokers}"
  ```

## 15. 用RestTemplate调用REST服务

如果你需要从你的应用程序中调用远程REST服务，你可以使用Spring框架的[`RestTemplate`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/web/client/RestTemplate.html)类。由于`RestTemplate`实例在使用前通常需要定制，Spring Boot没有提供任何单一的自动配置的`RestTemplate` Bean。然而，它确实自动配置了一个`RestTemplateBuilder`，在需要时可以用来创建`RestTemplate`实例。自动配置的`RestTemplateBuilder`确保合理的`HttpMessageConverters`被应用到`RestTemplate`实例中。

下面的代码显示了一个典型的例子。

```java
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class MyService {

    private final RestTemplate restTemplate;

    public MyService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public Details someRestCall(String name) {
        return this.restTemplate.getForObject("/{name}/details", Details.class, name);
    }

}
```

> `RestTemplateBuilder`包括一些有用的方法，可以用来快速配置一个`RestTemplate`。例如，要添加BASIC auth支持，你可以使用`builder.basicAuthentication("user", "password").build()` 。

### 15.1. RestTemplate定制

`RestTemplate`的定制有三种主要方法，取决于你希望定制的适用范围有多广。

为了使任何定制的范围尽可能的窄，注入自动配置的`RestTemplateBuilder`，然后根据需要调用其方法。每个方法的调用都会返回一个新的`RestTemplateBuilder`实例，所以自定义只影响构建器的这个用途。

要进行全应用的、附加的定制，请使用`RestTemplateCustomizer` Bean。所有这样的Bean都会自动注册到自动配置的`RestTemplateBuilder`上，并应用到用它构建的任何模板上。

下面的例子显示了一个自定义器，它配置了对除192.168.0.5以外的所有主机使用代理。

```java
import org.apache.http.HttpException;
import org.apache.http.HttpHost;
import org.apache.http.HttpRequest;
import org.apache.http.client.HttpClient;
import org.apache.http.conn.routing.HttpRoutePlanner;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.conn.DefaultProxyRoutePlanner;
import org.apache.http.protocol.HttpContext;

import org.springframework.boot.web.client.RestTemplateCustomizer;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

public class MyRestTemplateCustomizer implements RestTemplateCustomizer {

    @Override
    public void customize(RestTemplate restTemplate) {
        HttpRoutePlanner routePlanner = new CustomRoutePlanner(new HttpHost("proxy.example.com"));
        HttpClient httpClient = HttpClientBuilder.create().setRoutePlanner(routePlanner).build();
        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory(httpClient));
    }

    static class CustomRoutePlanner extends DefaultProxyRoutePlanner {

        CustomRoutePlanner(HttpHost proxy) {
            super(proxy);
        }

        @Override
        public HttpHost determineProxy(HttpHost target, HttpRequest request, HttpContext context) throws HttpException {
            if (target.getHostName().equals("192.168.0.5")) {
                return null;
            }
            return super.determineProxy(target, request, context);
        }

    }

}
```

最后，你也可以创建你自己的`RestTemplateBuilder` Bean。为了防止关闭`RestTemplateBuilder`的自动配置，防止任何`RestTemplateCustomizer` Bean 被使用，确保用`RestTemplateBuilderConfigurer`来配置你的自定义实例。下面的例子暴露了一个`RestTemplateBuilder`，它具有Spring Boot自动配置的功能，只是还指定了自定义连接和读取超时。

```java
import java.time.Duration;

import org.springframework.boot.autoconfigure.web.client.RestTemplateBuilderConfigurer;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
public class MyRestTemplateBuilderConfiguration {

    @Bean
    public RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer configurer) {
        return configurer.configure(new RestTemplateBuilder()).setConnectTimeout(Duration.ofSeconds(5))
                .setReadTimeout(Duration.ofSeconds(2));
    }

}
```

最极端（也很少使用）的选择是创建你自己的`RestTemplateBuilder` Bean，而不使用configurer。这样做可以关闭 `RestTemplateBuilder` 的自动配置，并防止任何 `RestTemplateCustomizer` Bean被使用。

## 16. 用WebClient调用REST服务

如果你的classpath上有Spring WebFlux，你也可以选择使用`WebClient`来调用远程REST服务。与`RestTemplate`相比，这个客户端更具备功能性，并且是完全响应式的。你可以在Spring Framework docs的专门[章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web-reactive.html#webflux-client)中了解更多关于`WebClient`的信息。

Spring Boot为你创建并预先配置了一个`WebClient.Builder`。强烈建议在你的组件中注入它，用它来创建`WebClient`实例。Spring Boot正在配置该生成器，以共享HTTP资源，以与服务器相同的方式反映编解码器的设置（见[WebFlux HTTP编解码器的自动配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-webflux.httpcodecs)），等等。

下面的代码显示了一个典型的例子。

```java
import org.neo4j.cypherdsl.core.Relationship.Details;
import reactor.core.publisher.Mono;

import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;

@Service
public class MyService {

    private final WebClient webClient;

    public MyService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://example.org").build();
    }

    public Mono<Details> someRestCall(String name) {
        return this.webClient.get().uri("/{name}/details", name).retrieve().bodyToMono(Details.class);
    }

}
```

### 16.1. WebClient运行时

Spring Boot将自动检测哪个`ClientHttpConnector`用来驱动`WebClient`，这取决于应用程序classpath上的可用库。目前，支持Reactor Netty和Jetty RS客户端。

`spring-boot-starter-webflux`启动器默认依赖`io.projectreactor.netty:reactor-netty`，它同时带来了服务器和客户端的实现。如果你选择使用Jetty作为反应式服务器，你应该添加对Jetty反应式HTTP客户端库的依赖，`org.eclipse.jetty:jetty-reactive-httpclient`。在服务器和客户端使用相同的技术有它的优势，因为它将自动在客户端和服务器之间共享HTTP资源。

开发者可以通过提供一个自定义的`ReactorResourceFactory`或`JettyResourceFactory`bean来覆盖Jetty和Reactor Netty的资源配置--这将同时应用于客户端和服务器。

如果你想覆盖客户端的选择，你可以定义你自己的`ClientHttpConnector`bean，并完全控制客户端的配置。

你可以在Spring框架参考文档中了解更多关于[`WebClient`配置选项](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web-reactive.html#webflux-client-builder)。

### 16.2. WebClient的定制

`WebClient`定制有三种主要方法，取决于你希望定制的范围有多大。

为了使任何定制的范围尽可能的窄，注入自动配置的`WebClient.Builder`，然后根据需要调用其方法。`WebClient.Builder`实例是有状态的。构建器上的任何变化都会反映在随后用它创建的所有客户端上。如果你想用同一个生成器创建多个客户端，你也可以考虑用`WebClient.Builder other = builder.clone();`来克隆生成器。

要对所有的`WebClient.Builder`实例进行全应用程序的附加定制，你可以声明`WebClientCustomizer` Bean，并在注入点本地改变`WebClient.Builder`。

最后，你可以回到原来的API，使用`WebClient.create()`。在这种情况下，没有自动配置或`WebClientCustomizer` 被应用。

## 17. 校验

只要在classpath上有JSR-303实现（如Hibernate验证器），Bean Validation 1.1所支持的方法验证功能就会自动启用。这让Bean方法在其参数和/或返回值上被注释为`javax.validation`约束。有这种注解的方法的目标类需要在类型级别上用`@Validated`注解进行注解，以使其方法被搜索到内联约束注解。

例如，下面的服务触发了对第一个参数的验证，确保其大小在8和10之间。

```java
import javax.validation.constraints.Size;

import org.springframework.stereotype.Service;
import org.springframework.validation.annotation.Validated;

@Service
@Validated
public class MyBean {

    public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code, Author author) {
        return ...
    }

}
```

## 18. 发送电子邮件

Spring框架通过使用 `JavaMailSender` 接口为发送电子邮件提供了一个抽象，Spring Boot为其提供了自动配置以及一个starter模块。

> 关于如何使用`JavaMailSender`的详细解释，请参见[参考文档](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/integration.html#mail)。

如果`spring.mail.host`和相关的库（如`spring-boot-starter-mail`所定义）是可用的，如果不存在，就会创建一个默认的`JavaMailSender`。发件人可以通过`spring.mail`命名空间中的配置项进一步定制。参见[`MailProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java)了解更多细节。

特别是，某些默认的超时值是无限的，你可能想改变它，以避免线程被无响应的邮件服务器阻塞，如下面的例子所示。

```yaml
spring:
  mail:
    properties:
      "[mail.smtp.connectiontimeout]": 5000
      "[mail.smtp.timeout]": 3000
      "[mail.smtp.writetimeout]": 5000
```

也可以用JNDI中现有的 `Session` 来配置 `JavaMailSender`。

```yaml
spring:
  mail:
    jndi-name: "mail/Session"
```

当 `jndi-name` 被设置时，它优先于所有其他与会话相关的设置。

## 19. 使用JTA的分布式事务

Spring Boot通过使用[Atomikos](https://www.atomikos.com/)嵌入式事务管理器，支持跨多个XA资源的分布式JTA事务。在部署到合适的Java EE应用服务器时，也支持JTA事务。

当检测到JTA环境时，Spring的`JtaTransactionManager`被用来管理事务。自动配置的JMS、DataSource和JPA Bean被升级以支持XA事务。你可以使用标准的Spring成语，如`@Transactional`，来参与分布式事务。如果你在JTA环境中，仍然想使用本地事务，你可以将`spring.jta.enabled`属性设置为`false`以禁用JTA自动配置。

### 19.1. 使用Atomikos事务管理器

[Atomikos](https://www.atomikos.com/)是一个流行的开源事务管理器，可以嵌入到你的Spring Boot应用程序。你可以使用`spring-boot-starter-jta-atomikos`启动器来拉入适当的Atomikos库。Spring Boot自动配置Atomikos，并确保适当的`depends-on`设置应用于你的Spring Bean，以便正确启动和关闭。

默认情况下，Atomikos交易日志被写入应用程序主目录（应用程序jar文件所在的目录）中的`transaction-logs`目录中。你可以通过在你的`application.properties`文件中设置`spring.jta.log-dir`属性来定制这个目录的位置。以`spring.jta.atomikos.properties`开头的属性也可以用来定制Atomikos的`UserTransactionServiceImp`。请参阅[`AtomikosProperties` Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html)了解完整的细节。

> 为了确保多个事务管理器能够安全地协调相同的资源管理器，每个Atomikos实例必须配置一个唯一的ID。默认情况下，这个ID是运行Atomikos的机器的IP地址。为了确保生产中的唯一性，你应该为你的应用程序的每个实例配置`spring.jta.transaction-manager-id`属性的不同值。

### 19.2. 使用Java EE管理的Transaction Manager

如果你将Spring Boot应用程序打包成`war`或`ear`文件，并将其部署到Java EE应用服务器上，你可以使用应用服务器的内置事务管理器。Spring Boot试图通过查看常见的JNDI位置（`java:comp/UserTransaction`、`java:comp/TransactionManager`等）来自动配置一个事务管理器。如果你使用应用服务器提供的事务服务，你一般也要确保所有资源由服务器管理并通过JNDI暴露。Spring Boot试图通过在JNDI路径（`java:/JmsXA`或`java:/XAConnectionFactory`）寻找一个`ConnectionFactory`来自动配置JMS，你可以使用[`spring.datasource.jndi-name`属性](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.sql.datasource.jndi)来配置你的`DataSource`。

### 19.3. 混合XA和非XA的JMS连接

当使用JTA时，主要的JMS`ConnectionFactory` Bean是XA-aware，并参与分布式事务。你可以注入你的Bean而不需要使用任何`@Qualifier`。

```java
public MyBean(ConnectionFactory connectionFactory) {
    // ...
}
```

在某些情况下，你可能想通过使用非XA的`ConnectionFactory`来处理某些JMS消息。例如，你的JMS处理逻辑可能需要比XA的超时时间更长。

如果你想使用一个非XA的`ConnectionFactory`，你可以使用`nonXaJmsConnectionFactory` Bean。

```java
public MyBean(@Qualifier("nonXaJmsConnectionFactory") ConnectionFactory connectionFactory) {
    // ...
}
```

为了保持一致性，`jmsConnectionFactory` bean也通过使用bean别名 `xaJmsConnectionFactory` 来提供。

```java
public MyBean(@Qualifier("xaJmsConnectionFactory") ConnectionFactory connectionFactory) {
    // ...
}
```

### 19.4. 支持另一种嵌入式Transaction Manager

[`XAConnectionFactoryWrapper`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jms/XAConnectionFactoryWrapper.java)和[`XADataSourceWrapper`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/XADataSourceWrapper.java)接口可以用来支持其他的嵌入式事务管理器。这些接口负责包装 `XAConnectionFactory` 和 `XADataSource` Bean，并将其作为常规的 `ConnectionFactory` 和 `DataSource` Bean公开，这些Bean可以透明地加入分布式事务。数据源和JMS自动配置使用JTA变体，只要你有一个`JtaTransactionManager` Bean和适当的XA包装Bean在你的`ApplicationContext`中注册。

[AtomikosXAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/atomikos/AtomikosXAConnectionFactoryWrapper.java)和[AtomikosXADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/atomikos/AtomikosXADataSourceWrapper.java)提供了如何编写XA包装器的好例子。

## 20. Hazelcast

如果[Hazelcast](https://hazelcast.com/)在classpath上，并且找到了合适的配置，Spring Boot就会自动配置一个`HazelcastInstance`，你可以把它注入你的应用程序。

Spring Boot首先尝试通过检查以下配置选项来创建一个客户端。

* 存在一个`com.hazelcast.client.config.ClientConfig`Bean。
* 一个由`spring.hazelcast.config`属性定义的配置文件。
* 存在`hazelcast.client.config`系统属性。
* 工作目录中或classpath根目录中的`hazelcast-client.xml`。
* 在工作目录或classpath的根目录下有`hazelcast-client.yaml`。

> Spring Boot同时支持Hazelcast 4和Hazelcast 3。如果你降级到Hazelcast 3，应将`hazelcast-client`添加到classpath中以配置客户端。

如果不能创建一个客户端，Spring Boot会尝试配置一个嵌入式服务器。如果你定义了一个`com.hazelcast.config.Config` Bean，Spring Boot就会使用这个Bean。如果你的配置定义了一个实例名称，Spring Boot会尝试定位一个现有的实例，而不是创建一个新的。

你也可以通过配置指定要使用的Hazelcast配置文件，如下面的例子所示。

```yaml
spring:
  hazelcast:
    config: "classpath:config/my-hazelcast.xml"
```

否则，Spring Boot会尝试从默认位置找到Hazelcast配置。工作目录或classpath根部的`hazelcast.xml`，或相同位置的`.yaml`对应物。我们还检查`hazelcast.config`系统属性是否被设置。更多细节请参见[Hazelcast文档](https://docs.hazelcast.org/docs/latest/manual/html-single/)。

> Spring Boot也有[对Hazelcast的明确缓存支持](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.caching.provider.hazelcast)。如果启用了缓存，`HazelcastInstance` 会被自动包裹在 `CacheManager` 实现中。

## 21. Quartz Scheduler

Spring Boot为与[Quartz Scheduler](https://www.quartz-scheduler.org/)合作提供了一些便利，包括`spring-boot-starter-quartz` "Starter"。如果Quartz可用，就会自动配置一个`Scheduler`（通过`SchedulerFactoryBean`抽象）。

以下类型的Bean被自动拾取并与`Scheduler`相关联。

* `JobDetail`: 定义了一个特定的工作。`JobDetail`的实例可以通过`JobBuilder` API建立。
* Calendar
* `Trigger`：定义一个特定工作何时被触发。

默认情况下，使用内存中的`JobStore`。然而，如果你的应用程序中有一个`DataSource` Bean，并且相应地配置了`spring.quartz.job-store-type`属性，也可以配置一个基于JDBC的存储，如下面的例子所示。

```yaml
spring:
  quartz:
    job-store-type: "jdbc"
```

当使用JDBC存储时，模式可以在启动时被初始化，如下面的例子中所示。

```yaml
spring:
  quartz:
    jdbc:
      initialize-schema: "always"
```

> 默认情况下，数据库是通过使用Quartz库提供的标准脚本来检测和初始化的。这些脚本会删除现有的表，在每次重启时删除所有触发器。也可以通过设置`spring.quartz.jdbc.schema`属性来提供一个自定义脚本。

要让Quartz使用除应用程序的主 `DataSource` 以外的 `DataSource`，请声明一个 `DataSource` bean，用 `@QuartzDataSource` 注释其`@Bean` "方法。这样做可以确保Quartz专用的`DataSource`被`SchedulerFactoryBean`和模式初始化所使用。同样地，为了让Quartz使用应用程序的主`TransactionManager`以外的`TransactionManager`，需要声明一个`TransactionManager`bean，用`@QuartzTransactionManager`来注释其`@Bean`方法。

默认情况下，通过配置创建的作业不会覆盖已经注册的作业，这些作业已经从持久性作业store中读取。要启用覆盖现有作业定义，请设置`spring.quartz.overwrite-existing-jobs`属性。

Quartz Scheduler的配置可以使用`spring.quartz`属性和`SchedulerFactoryBeanCustomizer` Bean来定制，它允许以编程方式定制`SchedulerFactoryBean`。高级Quartz配置属性可以使用`spring.quartz.properties.*`来定制。

> 特别是，`Executor` Bean与调度器没有关联，因为Quartz提供了一种通过`spring.quartz.properties`来配置调度器的方法。如果你需要定制任务执行器，可以考虑实现`SchedulerFactoryBeanCustomizer`。

Job可以定义setter来注入data map properties。常规的Bean也可以用类似的方式注入，如下面的例子中所示。

```java
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

import org.springframework.scheduling.quartz.QuartzJobBean;

public class MySampleJob extends QuartzJobBean {

    private MyService myService;

    private String name;

    // Inject "MyService" bean
    public void setMyService(MyService myService) {
        this.myService = myService;
    }

    // Inject the "name" job data property
    public void setName(String name) {
        this.name = name;
    }

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        this.myService.someMethod(context.getFireTime(), this.name);
    }

}
```

## 22. Task Execution 和 Scheduling

如果上下文中没有`Executor` Bean，Spring Boot会自动配置一个`ThreadPoolTaskExecutor`，它具有合理的默认值，可以自动与异步任务执行（`@EnableAsync`）和Spring MVC的异步请求处理相关联。

> 如果你在上下文中定义了一个自定义的`Executor`，常规的任务执行（即`@EnableAsync`）将透明地使用它，但Spring MVC支持将不会被配置，因为它需要一个`AsyncTaskExecutor`的实现（名为`applicationTaskExecutor`）。根据你的目标安排，你可以将你的 `Executor` 改为 `ThreadPoolTaskExecutor` `ThreadPoolTaskExecutor`和一个 `AsyncConfigurer` 来包装你的自定义 `Executor`。
>
>自动配置的`TaskExecutorBuilder`允许你轻松地创建实例，复制自动配置的默认操作。

线程池使用8个核心线程，可以根据负载增长和缩减。这些默认设置可以使用`spring.task.execution`命名空间进行微调，如以下例子所示。

```yaml
spring:
  task:
    execution:
      pool:
        max-size: 16
        queue-capacity: 100
        keep-alive: "10s"
```

这将线程池改为使用有界队列，因此当队列满了（100个任务），线程池增加到最大16个线程。线程池的收缩更加积极，因为当线程闲置10秒（而不是默认的60秒）时就会被回收。

如果需要与计划任务的执行相关联，也可以自动配置一个 `ThreadPoolTaskScheduler`（例如，`@EnableScheduling`）。线程池默认使用一个线程，它的设置可以使用`spring.task.scheduling`命名空间进行微调，如以下例子所示。

```yaml
spring:
  task:
    scheduling:
      thread-name-prefix: "scheduling-"
      pool:
        size: 2
```

如果需要创建一个自定义的执行器或调度器，上下文中的 `TaskExecutorBuilder` Bean和 `TaskSchedulerBuilder` Bean都是可用的。

## 23. Spring Integration

Spring Boot为与[Spring Integration](https://spring.io/projects/spring-integration)合作提供了一些便利，包括`spring-boot-starter-integration` "Starter"。Spring Integration提供了对消息传递的抽象，也提供了其他传输方式，如HTTP、TCP和其他。如果Spring Integration在你的classpath上可用，它将通过`@EnableIntegration`注解被初始化。

Spring Integration的轮询逻辑依赖于[自动配置的`TaskScheduler`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.task-execution-and-scheduling)。

Spring Boot还配置了一些由额外的Spring Integration模块的存在所触发的功能。如果`spring-integration-jmx`也在classpath上，则消息处理的统计数据会通过JMX发布。如果`spring-integration-jdbc`可用，可以在启动时创建默认的数据库模式，如下面一行所示。

```yaml
spring:
  integration:
    jdbc:
      initialize-schema: "always"
```

如果`spring-integration-rsocket`可用，开发者可以使用`"spring.rsocket.server.*"`属性配置RSocket服务器，让它使用`IntegrationRSocketEndpoint`或`RSocketOutboundGateway`组件来处理传入的RSocket消息。该基础设施可以处理Spring Integration RSocket通道适配器和`@MessageMapping`处理程序（鉴于`"spring.integration.rsocket.server.message-mapping-enabled"`已被配置）。

Spring Boot还可以使用配置属性自动配置`ClientRSocketConnector`。

```yaml
# Connecting to a RSocket server over TCP
spring:
  integration:
    rsocket:
      client:
        host: "example.org"
        port: 9898
```

```yaml
# Connecting to a RSocket Server over WebSocket
spring:
  integration:
    rsocket:
      client:
        uri: "ws://example.org"
```

更多细节请参见[`IntegrationAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java)和[`IntegrationProperties`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationProperties.java) 类。

默认情况下，如果Micrometer的`meterRegistry` Bean存在，Spring Integration的metrics将由Micrometer管理。如果你想使用传统的Spring Integration度量，请在应用上下文中添加一个`DefaultMetricsFactory` Bean。

## 24. Spring Session

Spring Boot为各种数据存储提供了[Spring Session](https://spring.io/projects/spring-session)的自动配置功能。在构建Servlet Web应用程序时，可以自动配置以下存储。

* JDBC
* Redis
* Hazelcast
* MongoDB

Servlet的自动配置取代了使用`@Enable*HttpSession`的需要。

当构建一个响应式网络应用程序时，以下存储可以被自动配置。

* Redis
* MongoDB

反应式自动配置取代了使用`@Enable*WebSession`的需要。

如果classpath上有一个Spring Session模块，Spring Boot会自动使用该存储实现。如果你有一个以上的实现，你必须选择你希望用来存储会话的[`StoreType`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/session/StoreType.java)。例如，要使用JDBC作为后端存储，你可以对你的应用程序进行如下配置。

```yaml
spring:
  session:
    store-type: "jdbc"
```

> 你可以通过设置`store-type`为`none`来禁用Spring Session。

每个store都有特定的附加设置。例如，可以定制JDBC存储的表的名称，如下例所示。

```yaml
spring:
  session:
    jdbc:
      table-name: "SESSIONS"
```

为了设置session的超时，你可以使用`spring.session.timeout`属性。如果Servlet Web应用程序没有设置该属性，自动配置就会退回到`server.servlet.session.timeout`的值。

你可以使用`@Enable*HttpSession`（Servlet）或`@Enable*WebSession`（Reactive）来控制Spring Session的配置。这将导致自动配置的后退。然后，Spring Session可以使用注解的属性进行配置，而不是之前描述的配置属性。

## 25. 通过JMX进行监控和管理

Java管理扩展（JMX）提供了一个标准的机制来监控和管理应用程序。Spring Boot将最合适的`MBeanServer`作为ID为`mbeanServer`的bean公开。你的任何带有Spring JMX注解的Bean（`@ManagedResource`、`@ManagedAttribute`或`@ManagedOperation`）都会暴露给它。

如果你的平台提供了一个标准的`MBeanServer`，Spring Boot将使用它，并在必要时默认为VM`MBeanServer`。如果所有这些都失败了，将创建一个新的`MBeanServer`。

更多细节请参见[`JmxAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java)类。

## 26. 测试

Spring Boot提供了许多实用程序和注解，以便在测试你的应用程序时提供帮助。测试支持由两个模块提供。`spring-boot-test`包含核心项目，`spring-boot-test-autoconfigure`支持测试的自动配置。

大多数开发者使用`spring-boot-starter-test` "Starter"，它同时导入Spring Boot测试模块以及JUnit Jupiter、AssertJ、Hamcrest和其他一些有用的库。

如果你有使用JUnit 4的测试，可以使用JUnit 5的vintage引擎来运行它们。要使用vintage引擎，请添加对`junit-vint-engine`的依赖，如以下例子所示。

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

`hamcrest-core`被排除在外，而支持`org.hamcrest:hamcrest`，它是`spring-boot-starter-test`的一部分。

### 26.1. Test Scope Dependencies

`spring-boot-starter-test`（在`test`scope`中）包含以下提供的库。

* [JUnit 5](https://junit.org/junit5/)。Java应用程序单元测试的事实标准。
* [Spring Test](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#integration-testing) & Spring Boot Test。对Spring Boot应用程序的实用程序和集成测试支持。
* [AssertJ](https://assertj.github.io/doc/): 一个流畅的断言库。
* [Hamcrest](https://github.com/hamcrest/JavaHamcrest): 一个匹配器对象（也被称为约束或谓语）库。
* [Mockito](https://site.mockito.org/): 一个Java嘲弄框架。
* [JSONassert](https://github.com/skyscreamer/JSONassert)。一个用于JSON的断言库。
* [JsonPath](https://github.com/jayway/JsonPath)。用于JSON的XPath。

我们通常认为这些常用的库在编写测试时很有用。如果这些库不适合你的需要，你可以添加你自己的额外测试依赖。

### 26.2. 测试Spring应用程序

依赖性注入的一个主要优点是，它应该使你的代码更容易进行单元测试。你可以通过使用`new`操作符来实例化对象，甚至不需要涉及Spring。你还可以使用*模拟对象*来代替真实的依赖关系。

通常，你需要超越单元测试，开始进行集成测试（使用Spring `ApplicationContext`）。能够执行集成测试而不需要部署你的应用程序或需要连接到其他基础设施是非常有用的。

Spring框架包括一个专门的测试模块，用于此类集成测试。你可以直接向`org.springframework:spring-test`声明依赖关系，或者使用`spring-boot-starter-test`的 "启动器 "将其转入。

如果你以前没有使用过`spring-test`模块，你应该先阅读Spring框架参考文档的[相关章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#testing)。

### 26.3. 测试Spring Boot应用程序

Spring Boot应用程序是一个Spring `ApplicationContext`，所以除了通常对普通Spring上下文的测试外，不需要做什么特别的测试。

> 只有当你使用`SpringApplication`来创建时，Spring Boot的外部属性、日志和其他功能才会默认安装在上下文中。

Spring Boot提供了一个`@SpringBootTest`注解，当你需要Spring Boot功能时，它可以作为标准`spring-test``@ContextConfiguration`注解的替代品。该注解通过[通过`SpringApplication`创建你的测试中使用的`ApplicationContext`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.detecting-configuration)发挥作用。除了`@SpringBootTest`之外，还提供了一些其他注解，用于[测试更多的具体片断](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-tests)。

> 如果你使用的是JUnit 4，别忘了在测试中也添加`@RunWith(SpringRunner.class)`，否则注释会被忽略。如果你使用的是JUnit 5，就不需要添加等同的`@ExtendWith(SpringExtension.class)`，因为`@SpringBootTest`和其他`@...Test`注解已经被注解了。

默认情况下，`@SpringBootTest`不会启动一个服务器。你可以使用`@SpringBootTest`的`webEnvironment`属性来进一步完善你的测试运行方式。

* `MOCK`（默认）：加载一个网络`ApplicationContext`并提供一个模拟的网络环境。当使用这个注解时，嵌入式服务器不会被启动。如果你的classpath上没有web环境，这种模式会透明地退回到创建一个普通的非web `ApplicationContext`。它可以与[`@AutoConfigureMockMvc`或`@AutoConfigureWebTestClient`](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.with-mock-environment)一起使用，对你的Web应用进行基于模拟的测试。
* `RANDOM_PORT`：加载一个`WebServerApplicationContext`并提供一个真实的网络环境。嵌入式服务器被启动并监听一个随机端口。
* `DEFINED_PORT`：加载一个`WebServerApplicationContext`并提供一个真实的网络环境。嵌入式服务器被启动并监听一个定义的端口（来自你的`application.properties`）或默认端口`8080`。
* `NONE` : 通过使用`SpringApplication`加载一个`ApplicationContext`，但不提供*任何*网络环境（模拟或其他）。

如果你的测试是`@Transactional`，它默认在每个测试方法结束时回滚事务。然而，由于使用这种安排与`RANDOM_PORT`或`DEFINED_PORT`隐含地提供了一个真正的servlet环境，HTTP客户端和服务器在不同的线程中运行，因此，在不同的事务中。在这种情况下，在服务器上发起的任何事务都不会回滚。

`@SpringBootTest`与`webEnvironment = WebEnvironment.RANDOM_PORT`也将在一个单独的随机端口上启动管理服务器，如果你的应用程序为管理服务器使用一个不同的端口。

#### 26.3.1. 检测网络应用程序类型

如果有Spring MVC，就会配置一个基于MVC的常规应用上下文。如果你只有Spring WebFlux，我们会检测到它并配置一个基于WebFlux的应用上下文。

如果两者都有，则以Spring MVC为准。如果你想在这种情况下测试一个反应式Web应用，你必须设置`spring.main.web-application-type`属性。

```java
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(properties = "spring.main.web-application-type=reactive")
class MyWebFluxTests {

    // ...

}
```

#### 26.3.2. 检测测试配置

如果你熟悉Spring测试框架，你可能习惯于使用`@ContextConfiguration(classes=...)`来指定加载哪个Spring`@Configuration`。另外，你可能经常在测试中使用嵌套的`@Configuration`类。

在测试Spring Boot应用程序时，这通常是不需要的。只要你没有明确定义配置，Spring Boot的`@*Test`注释就会自动搜索你的主要配置。

搜索算法从包含测试的包开始，直到找到一个用`@SpringBootApplication`或`@SpringBootConfiguration`注释的类。只要你以合理的方式[结构化你的代码](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code)，你的主配置通常会被找到。

如果你使用[测试注解来测试你的应用程序的一个更具体的slice](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-tests)，你应该避免在[主方法的应用类](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.user-configuration-and-slicing)上添加针对特定区域的配置设置。

`@SpringBootApplication`的底层组件扫描配置定义了排除过滤器，用于确保slicing工作符合预期。如果你在你的`@SpringBootApplication`-注释的类上使用明确的`@ComponentScan`指令，请注意这些过滤器将被禁用。如果你正在使用slicing，你应该重新定义它们。

如果你想定制主配置，你可以使用一个嵌套的`@TestConfiguration`类。与嵌套的`@Configuration`类不同的是，嵌套的`@TestConfiguration`类是在你的应用程序的主要配置之外使用的，它将代替你的应用程序的主要配置。

Spring的测试框架在测试之间缓存了应用程序上下文。因此，只要你的测试共享相同的配置（无论它是如何被发现的），加载上下文的潜在耗时过程只发生一次。

#### 26.3.3. 排除测试配置

如果你的应用程序使用组件扫描（例如，如果你使用`@SpringBootApplication`或`@ComponentScan`），你可能会发现你只为特定测试创建的顶级配置类意外地被到处捡到。

正如我们[之前看到的](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.detecting-configuration)，`@TestConfiguration`可以用在一个测试的内部类上，以定制主要的配置。当放在顶层类上时，`@TestConfiguration`表明`src/test/java`中的类不应该被扫描到。然后你可以在需要的地方明确地导入该类，如下面的例子中所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;

@SpringBootTest
@Import(MyTestsConfiguration.class)
class MyTests {

    @Test
    void exampleTest() {
        // ...
    }

}
```

如果你直接使用`@ComponentScan`（也就是不通过`@SpringBootApplication`），你需要向它注册`TypeExcludeFilter`。详情见[Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/context/TypeExcludeFilter.html)。

#### 26.3.4. 使用Application Arguments

如果你的应用程序期待[arguments](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-arguments)，你可以让`@SpringBootTest`使用`args`属性注入它们。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.test.context.SpringBootTest;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(args = "--app.test=one")
class MyApplicationArgumentTests {

    @Test
    void applicationArgumentsPopulated(@Autowired ApplicationArguments args) {
        assertThat(args.getOptionNames()).containsOnly("app.test");
        assertThat(args.getOptionValues("app.test")).containsOnly("one");
    }

}
```

#### 26.3.5. 用模拟环境进行测试

默认情况下，`@SpringBootTest`并不启动服务器。如果你有想在这个模拟环境中测试的Web端点，你可以额外配置[`MockMvc`](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#spring-mvc-test-framework)，如下例所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class MyMockMvcTests {

    @Test
    void exampleTest(@Autowired MockMvc mvc) throws Exception {
        mvc.perform(get("/")).andExpect(status().isOk()).andExpect(content().string("Hello World"));
    }

}
```

> 如果你想只关注Web层而不启动一个完整的`ApplicationContext`，可以考虑[用`@WebMvcTest`代替](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.spring-mvc-tests).|

另外，你可以配置一个[`WebTestClient`](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#webtestclient-tests)，如下例所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.AutoConfigureWebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest
@AutoConfigureWebTestClient
class MyMockWebTestClientTests {

    @Test
    void exampleTest(@Autowired WebTestClient webClient) {
        webClient
            .get().uri("/")
            .exchange()
            .expectStatus().isOk()
            .expectBody(String.class).isEqualTo("Hello World");
    }

}
```

在模拟的环境中进行测试，通常比使用完整的Servlet容器运行要快。然而，由于嘲讽发生在Spring MVC层，所以依赖低级Servlet容器行为的代码不能直接用MockMvc测试。

> 例如，Spring Boot的错误处理是基于Servlet容器所提供的 "error page" 支持。这意味着，虽然你可以测试你的MVC层抛出并按预期处理异常，但你不能直接测试特定的[自定义错误页面](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.error-handling.error-pages)是否被呈现。如果你需要测试这些低层次的问题，你可以启动一个完全运行的服务器，如下一节所述。

#### 26.3.6. 用一个正在运行的服务器进行测试

如果你需要启动一个完整运行的服务器，我们建议你使用随机端口。如果你使用`@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`，每次测试运行时都会随机挑选一个可用的端口。

`@LocalServerPort`注解可以用来[注入实际使用的端口](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.webserver.discover-port)到你的测试。为了方便，需要对启动的服务器进行REST调用的测试可以另外`@Autowire`一个[`WebTestClient`](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#webtestclient-tests)，它可以解析到运行的服务器的相对链接，并带有专门的API来验证响应，如下面的例子所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class MyRandomPortWebTestClientTests {

    @Test
    void exampleTest(@Autowired WebTestClient webClient) {
        webClient
            .get().uri("/")
            .exchange()
            .expectStatus().isOk()
            .expectBody(String.class).isEqualTo("Hello World");
    }

}

```

这种设置要求classpath上有`spring-webflux`。如果你不能或不愿添加webflux，Spring Boot也提供了一个`TestRestTemplate`设施。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.web.client.TestRestTemplate;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class MyRandomPortTestRestTemplateTests {

    @Test
    void exampleTest(@Autowired TestRestTemplate restTemplate) {
        String body = restTemplate.getForObject("/", String.class);
        assertThat(body).isEqualTo("Hello World");
    }

}
```

#### 26.3.7. 定制WebTestClient

为了定制`WebTestClient`bean，配置一个`WebTestClientBuilderCustomizer` Bean。任何这样的Bean都是与用来创建`WebTestClient`的`WebTestClient.Builder`一起调用的。

#### 26.3.8. 使用JMX

由于测试上下文框架缓存了上下文，JMX默认是禁用的，以防止相同的组件在同一领域注册。如果这样的测试需要访问 `MBeanServer`，可以考虑把它也标记为dirty。

```java
import javax.management.MBeanServer;
import javax.management.MalformedObjectNameException;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import static org.assertj.core.api.Assertions.assertThat;

@ExtendWith(SpringExtension.class)
@SpringBootTest(properties = "spring.jmx.enabled=true")
@DirtiesContext
class MyJmxTests {

    @Autowired
    private MBeanServer mBeanServer;

    @Test
    void exampleTest() throws MalformedObjectNameException {
        assertThat(this.mBeanServer.getDomains()).contains("java.lang");
        // ...
    }

}
```

#### 26.3.9. 使用 Metrics

无论你的classpath如何，在使用`@SpringBootTest`时，除了内存中的备份外，meter registries不会被自动配置。

如果你需要将metrics导出到不同的后端作为集成测试的一部分，请用`@AutoConfigureMetrics`来注释。

#### 26.3.10. Mocking 和 Spying Beans

当运行测试时，有时需要在你的应用环境中模拟某些组件。例如，你可能有一个在开发期间不可用的远程服务的界面。当你想模拟在真实环境中很难触发的故障时，嘲讽也很有用。

Spring Boot包括一个`@MockBean`注解，可以用来为`ApplicationContext`中的Bean定义一个Mockito模拟。你可以使用该注解来添加新的Bean或替换现有的单一Bean定义。该注解可以直接用于测试类，测试中的字段，或用于`@Configuration`类和字段。当在一个字段上使用时，创建的模拟实例也被注入。模拟Bean在每个测试方法后都会自动重置。

如果你的测试使用了Spring Boot的一个测试注解（如`@SpringBootTest`），这个功能就会自动启用。要用不同的排列来使用这个功能，必须明确地添加监听器，如下面的例子所示。

```java
import org.springframework.boot.test.mock.mockito.MockitoTestExecutionListener;
import org.springframework.boot.test.mock.mockito.ResetMocksTestExecutionListener;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.TestExecutionListeners;

@ContextConfiguration(classes = MyConfig.class)
@TestExecutionListeners({ MockitoTestExecutionListener.class, ResetMocksTestExecutionListener.class })
class MyTests {

    // ...

}
```

下面的例子用一个模拟的实现替换了现有的`RemoteService` Bean。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;

@SpringBootTest
class MyTests {

    @Autowired
    private Reverser reverser;

    @MockBean
    private RemoteService remoteService;

    @Test
    void exampleTest() {
        given(this.remoteService.getValue()).willReturn("spring");
        String reverse = this.reverser.getReverseValue(); // Calls injected RemoteService
        assertThat(reverse).isEqualTo("gnirps");
    }

}
```

> `@MockBean`不能用来模拟在application context刷新期间行使的Bean的行为。当测试被执行时，application context的刷新已经完成，配置模拟的行为已经太晚了。我们建议在这种情况下使用`@Bean`方法来创建和配置模拟行为。

此外，你可以使用`@SpyBean`来用Mockito的`spy`包装任何现有的bean。详细内容请参见[Javadoc](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/test/mock/mockito/SpyBean.html)。

> CGLib代理，比如那些为作用域Bean创建的代理，将代理的方法声明为`final`。这使得Mockito无法正常工作，因为在其默认配置中，它不能对`final`方法进行模拟或监视。如果你想对这样的Bean进行模拟或监视，请将`org.mockito:mockito-inline`添加到你的应用程序的测试依赖项中，以配置Mockito使用其内联模拟器。这允许Mockito对`final`方法进行模拟和监视。

虽然Spring的测试框架在测试之间缓存application context，并为共享相同配置的测试重用一个context，但使用`@MockBean`或`@SpyBean`会影响缓存的关键，这很可能会增加context的数量。

如果你使用`@SpyBean`来监视一个有`@Cacheable`方法的Bean，这些方法通过名称来引用参数，你的应用程序必须用`-parameters`来编译。这可以确保一旦Bean被监视到，参数名称就可以被缓存基础设施使用。

当你使用`@SpyBean`来监视一个被Spring代理的Bean时，你可能需要在某些情况下移除Spring的代理，例如在使用`given`或`when`设置期望时。使用`AopTestUtils.getTargetObject(yourProxiedSpy)`来做到这一点。

#### 26.3.11. 自动配置的测试

Spring Boot的自动配置系统对应用程序来说效果很好，但对测试来说有时会有点过头。通常情况下，只加载测试应用程序 "slice" 所需的配置部分是有帮助的。例如，你可能想测试Spring MVC控制器是否正确映射了URL，而你不想在这些测试中涉及数据库调用，或者你可能想测试JPA实体，而你对这些测试运行时的Web层不感兴趣。

`spring-boot-test-autoconfigure`模块包括一些注释，可以用来自动配置这种 "slice"。它们中的每一个都以类似的方式工作，提供一个`@...Test`注解来加载`ApplicationContext`和一个或多个`@AutoConfigure...`注解，可以用来定制自动配置设置。

每个slice将组件扫描限制在适当的组件上，并加载一组非常有限的自动配置类。如果你需要排除其中一个，大多数`@...Test`注解提供了一个`excludeAutoConfiguration`属性。另外，你可以使用`@ImportAutoConfiguration#exclude` 。

不支持通过在一个测试中使用几个`@...Test`注解来包括多个 "slice"。如果你需要多个 "slice"，选择其中一个`@...Test` 注释，然后手动包括其他 "slice"的`@AutoConfigure...` 注释。

也可以将`@AutoConfigure...`注解与标准的`@SpringBootTest`注解一起使用。如果你对 "slice" 你的应用程序不感兴趣，但你想要一些自动配置的测试Bean，你可以使用这种组合。

#### 26.3.12. 自动配置的JSON测试

为了测试对象JSON序列化和反序列化是否按预期工作，你可以使用`@JsonTest`注解。`@JsonTest`自动配置可用的支持JSON的映射器，它可以是下列库之一。

* `Jackson ObjectMapper`，任何`@JsonComponent` Bean和任何Jackson模块
* `Gson`
* `Jsonb`

> `@JsonTest`启用的自动配置的列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

如果你需要配置自动配置的元素，你可以使用`@AutoConfigureJsonTesters`注解。

Spring Boot包括基于AssertJ的帮助器，与JSONAssert和JsonPath库一起工作，以检查JSON是否出现在预期中。`JacksonTester`、`GsonTester`、`JsonbTester`和`BasicJsonTester`类可以分别用于Jackson、Gson、Jsonb和Strings。当使用`@JsonTest`时，测试类上的任何帮助字段都可以是`@Autowired`。下面的例子显示了一个用于Jackson的测试类。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.json.JsonTest;
import org.springframework.boot.test.json.JacksonTester;

import static org.assertj.core.api.Assertions.assertThat;

@JsonTest
class MyJsonTests {

    @Autowired
    private JacksonTester<VehicleDetails> json;

    @Test
    void serialize() throws Exception {
        VehicleDetails details = new VehicleDetails("Honda", "Civic");
        // Assert against a `.json` file in the same package as the test
        assertThat(this.json.write(details)).isEqualToJson("expected.json");
        // Or use JSON path based assertions
        assertThat(this.json.write(details)).hasJsonPathStringValue("@.make");
        assertThat(this.json.write(details)).extractingJsonPathStringValue("@.make").isEqualTo("Honda");
    }

    @Test
    void deserialize() throws Exception {
        String content = "{\"make\":\"Ford\",\"model\":\"Focus\"}";
        assertThat(this.json.parse(content)).isEqualTo(new VehicleDetails("Ford", "Focus"));
        assertThat(this.json.parseObject(content).getMake()).isEqualTo("Ford");
    }

}
```

JSON helper 类也可以直接用于标准单元测试。要做到这一点，如果你不使用`@JsonTest`，请在你的`@Before`方法中调用该helper的`initFields`方法。

如果你使用Spring Boot的基于AssertJ的助手来断言给定JSON路径的数字值，你可能无法使用`isEqualTo`，这取决于类型。相反，你可以使用AssertJ的`satisfies`来断言该值符合给定条件。例如，下面的例子断言实际数字是一个接近`0.15`的浮点数，偏移量为`0.01`。

```java
@Test
void someTest() throws Exception {
    SomeObject value = new SomeObject(0.152f);
    assertThat(this.json.write(value)).extractingJsonPathNumberValue("@.test.numberValue")
            .satisfies((number) -> assertThat(number.floatValue()).isCloseTo(0.15f, within(0.01f)));
}
```

#### 26.3.13. 自动配置的Spring MVC测试

要测试Spring MVC控制器是否按预期工作，请使用`@WebMvcTest`注解。`@WebMvcTest`自动配置Spring MVC基础设施，并将扫描的Bean限制在`@Controller`、`@ControllerAdvice`、`@JsonComponent`、`Converter`、`GenericConverter`、`Filter`、`HandlerInterceptor`、`WebMvcConfigurer`和`HandlerMethodArgumentResolver`。当使用`@WebMvcTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

`@WebMvcTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

如果你需要注册额外的组件，比如Jackson `Module`，你可以通过在测试中使用`@Import`导入额外的配置类。

通常情况下，`@WebMvcTest`仅限于一个controller，并与`@MockBean`结合使用，为需要的合作者提供模拟实现。

`@WebMvcTest`也自动配置`MockMvc`。Mock MVC提供了一个强大的方式来快速测试MVC控制器，而不需要启动一个完整的HTTP服务器。

你也可以在一个非`@WebMvcTest`（如`@SpringBootTest`）中自动配置`MockMvc`，方法是用`@AutoConfigureMockMvc`注释它。下面的例子使用了`MockMvc`。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(UserVehicleController.class)
class MyControllerTests {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot"))
            .willReturn(new VehicleDetails("Honda", "Civic"));
        this.mvc.perform(get("/sboot/vehicle").accept(MediaType.TEXT_PLAIN))
            .andExpect(status().isOk())
            .andExpect(content().string("Honda Civic"));
    }

}
```

如果你需要配置自动配置的元素（例如，何时应该应用servlet过滤器），你可以使用`@AutoConfigureMockMvc`注解中的属性。

如果你使用HtmlUnit或Selenium，自动配置也提供了一个HtmlUnit `WebClient` Bean和/或一个Selenium `WebDriver` Bean。下面的例子使用HtmlUnit。

```java
import com.gargoylesoftware.htmlunit.WebClient;
import com.gargoylesoftware.htmlunit.html.HtmlPage;
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;

@WebMvcTest(UserVehicleController.class)
class MyHtmlUnitTests {

    @Autowired
    private WebClient webClient;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot")).willReturn(new VehicleDetails("Honda", "Civic"));
        HtmlPage page = this.webClient.getPage("/sboot/vehicle.html");
        assertThat(page.getBody().getTextContent()).isEqualTo("Honda Civic");
    }

}
```

默认情况下，Spring Boot将`WebDriver`Bean放在一个特殊的 "scope" 中，以确保每次测试后驱动程序都会退出，并注入一个新实例。如果你不想要这种行为，你可以在你的`WebDriver``@Bean`定义中添加`@Scope("singleton")`。

Spring Boot创建的`webDriver`作用域将取代任何用户定义的同名作用域。如果你定义了自己的`webDriver`作用域，你可能会发现当你使用`@WebMvcTest`时它会停止工作。

如果你在classpath上有Spring Security，`@WebMvcTest`也将扫描`WebSecurityConfigurer` Bean。你可以使用Spring Security的测试支持，而不是为这类测试完全禁用安全。关于如何使用Spring Security的`MockMvc`支持的更多细节，可以在这个[howto.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.testing.with-spring-security)的how-to部分找到。

> 有时仅仅编写Spring MVC测试是不够的；Spring Boot可以帮助你运行[具有实际服务器的完整端到端测试](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.with-running-server)。

#### 26.3.14. 自动配置的Spring WebFlux测试

为了测试[Spring WebFlux](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web-reactive.html)控制器是否按预期工作，你可以使用`@WebFluxTest`注解。`@WebFluxTest`自动配置Spring WebFlux基础设施，并将扫描的bean限制在`@Controller`、`@ControllerAdvice`、`@JsonComponent`、`Converter`、`GenericConverter`、`WebFilter`和`WebFluxConfigurer`。当使用`@WebFluxTest` 注解时，常规的`@Component`和`@ConfigurationProperties` Bean 不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

`@WebFluxTest`启用的自动配置的列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

如果你需要注册额外的组件，如Jackson `Module`，你可以在测试中使用`@Import`导入额外的配置类。

通常，`@WebFluxTest`仅限于一个控制器，并与`@MockBean`注解结合使用，为所需的合作者提供模拟实现。

`@WebFluxTest`还自动配置了[`WebTestClient`](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#webtestclient)，这为快速测试WebFlux控制器提供了一种强大的方式，而不需要启动一个完整的HTTP服务器。

你也可以在非`@WebFluxTest` 中自动配置 `WebTestClient`（如`@SpringBootTest`），方法是用`@AutoConfigureWebTestClient`注释它。下面的例子显示了一个同时使用`@WebFluxTest`和`WebTestClient`的类。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.mockito.BDDMockito.given;

@WebFluxTest(UserVehicleController.class)
class MyControllerTests {

    @Autowired
    private WebTestClient webClient;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot"))
            .willReturn(new VehicleDetails("Honda", "Civic"));
        this.webClient.get().uri("/sboot/vehicle").accept(MediaType.TEXT_PLAIN).exchange()
            .expectStatus().isOk()
            .expectBody(String.class).isEqualTo("Honda Civic");
    }

}
```

这种设置只被WebFlux应用程序支持，因为在模拟的Web应用程序中使用`WebTestClient`目前只适用于WebFlux。

`@WebFluxTest`无法检测到通过功能性网络框架注册的路由。对于测试上下文中的`RouterFunction` Bean，可以考虑通过`@Import`自己导入你的`RouterFunction`或使用`@SpringBootTest`。

`@WebFluxTest`不能检测通过`@Bean`类型的`SecurityWebFilterChain`注册的自定义安全配置。要在你的测试中包括这一点，你将需要通过`@Import`导入注册Bean的配置，或者使用`@SpringBootTest`。

有时仅仅编写Spring WebFlux测试是不够的；Spring Boot可以帮助你运行[具有实际服务器的完整端到端测试](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.with-running-server)。

#### 26.3.15. 自动配置的Data Cassandra测试

你可以使用`@DataCassandraTest`来测试Cassandra应用程序。默认情况下，它配置一个`CassandraTemplate`，扫描`@Table`类，并配置Spring Data Cassandra仓库。当使用`@DataCassandraTest` 注解时，常规的`@Component`和`@ConfigurationProperties` Bean 不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。(关于在Spring Boot中使用Cassandra的更多信息，请参阅本章前面的 [Cassandra](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.cassandra))。

> `@DataCassandraTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

下面的例子显示了在Spring Boot中使用Cassandra测试的一个典型设置。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.cassandra.DataCassandraTest;

@DataCassandraTest
class MyDataCassandraTests {

    @Autowired
    private SomeRepository repository;

}
```

#### 26.3.16. 自动配置的数据JPA测试

你可以使用`@DataJpaTest`注解来测试JPA应用程序。默认情况下，它会扫描`@Entity`类并配置Spring Data JPA存储库。如果classpath上有一个嵌入式数据库，它也会配置一个。默认情况下，通过将`spring.jpa.show-sql`属性设置为`true`来记录SQL查询。这可以通过注解的`showSql()`属性来禁用。

当使用`@DataJpaTest` 注解时，常规的`@Component`和`@ConfigurationProperties` Bean 不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

> `@DataJpaTest`启用的自动配置设置的列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

默认情况下，数据JPA测试是事务性的，并在每次测试结束后回滚。请参阅Spring框架参考文档中的[相关章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#testcontext-tx-enabling-transactions)以了解更多细节。如果这不是你想要的，你可以为一个测试或整个类停用事务管理，如下所示。

```java
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@DataJpaTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class MyNonTransactionalTests {

    // ...

}
```

数据JPA测试也可以注入一个[`TestEntityManager`](https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure/orm/jpa/TestEntityManager.java)bean，它提供了一个标准JPA`EntityManager`的替代品，是专门为测试设计的。如果你想在`@DataJpaTest`实例之外使用`TestEntityManager`，你也可以使用`@AutoConfigureTestEntityManager`注释。如果你需要，也可以使用`JdbcTemplate`。下面的例子显示了`@DataJpaTest`注解的使用情况。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class MyRepositoryTests {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository repository;

    @Test
    void testExample() throws Exception {
        this.entityManager.persist(new User("sboot", "1234"));
        User user = this.repository.findByUsername("sboot");
        assertThat(user.getUsername()).isEqualTo("sboot");
        assertThat(user.getEmployeeNumber()).isEqualTo("1234");
    }

}
```

内存中的嵌入式数据库通常对测试很有效，因为它们速度快，而且不需要任何安装。然而，如果你喜欢针对真实的数据库运行测试，你可以使用`@AutoConfigureTestDatabase`注解，如下面的例子所示。

```java
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase.Replace;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
class MyRepositoryTests {

    // ...

}
```

#### 26.3.17. 自动配置的JDBC测试

`@JdbcTest`与`@DataJpaTest`类似，但用于只需要`DataSource`而不使用Spring Data JDBC的测试。默认情况下，它配置了一个内存嵌入式数据库和一个`JdbcTemplate`。当使用`@JdbcTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

> `@JdbcTest`启用的自动配置的列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

默认情况下，JDBC测试是事务性的，并在每次测试结束后回滚。请参阅Spring框架参考文档中的[相关章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#testcontext-tx-enabling-transactions)以了解更多细节。如果这不是你想要的，你可以为一个测试或整个类停用事务管理，如下所示。

```java
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@JdbcTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class MyTransactionalTests {

}
```

如果你希望你的测试能针对真实的数据库运行，你可以使用`@AutoConfigureTestDatabase`注解，方法与`DataJpaTest`相同。（参见[自动配置的数据JPA测试](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-spring-data-jpa)。

#### 26.3.18. 自动配置的数据JDBC测试

`@DataJdbcTest`与`@JdbcTest`类似，但用于使用Spring Data JDBC存储库的测试。默认情况下，它配置了一个内存中的嵌入式数据库，一个`JdbcTemplate`，和Spring Data JDBC资源库。当使用`@DataJdbcTest` 注解时，常规的`@Component` 和 `@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

> `@DataJdbcTest`启用的自动配置的列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

默认情况下，Data JDBC测试是事务性的，并在每次测试结束后回滚。参见Spring框架参考文档中的[相关章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#testcontext-tx-enabling-transactions)以了解更多细节。如果这不是你想要的，你可以为一个测试或整个测试类禁用事务管理，如[JDBC例子中所示](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-jdbc)。

如果你希望你的测试针对真实的数据库运行，你可以使用`@AutoConfigureTestDatabase`注解，方法与`DataJpaTest`相同。（参见[自动配置的数据JPA测试](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-spring-data-jpa)。

#### 26.3.19. 自动配置的jOOQ测试

你可以以类似于`@JdbcTest`的方式使用`@JooqTest`，但用于与jOOQ相关的测试。由于jOOQ在很大程度上依赖于与数据库模式相对应的基于Java的模式，因此使用了现有的`DataSource`。如果你想用内存数据库取代它，你可以使用`@AutoConfigureTestDatabase`来覆盖这些设置。(关于在Spring Boot中使用jOOQ的更多信息，见本章前面的 [使用jOOQ](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.sql.jooq)) 当使用`@JooqTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

> `@JooqTest`启用的自动配置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

`@JooqTest`配置了一个`DSLContext`。下面的例子显示了`@JooqTest`注解的使用情况。

```java
import org.jooq.DSLContext;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jooq.JooqTest;

@JooqTest
class MyJooqTests {

    @Autowired
    private DSLContext dslContext;

    // ...

}
```

JOOQ测试是事务性的，默认在每个测试结束时回滚。如果这不是你想要的，你可以为一个测试或整个测试类禁用事务管理，如[在JDBC例子中所示](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.autoconfigured-jdbc)。

#### 26.3.20. 自动配置的数据MongoDB测试

你可以使用`@DataMongoTest`来测试MongoDB应用程序。默认情况下，它配置内存中的嵌入式MongoDB（如果可用），配置`MongoTemplate`，扫描`@Document`类，并配置Spring Data MongoDB存储库。当使用`@DataMongoTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean 不会被扫描。`@EnableConfigurationProperties`可用于包括`@ConfigurationProperties` Bean。(关于在Spring Boot中使用MongoDB的更多信息，请参阅本章前面的[MongoDB](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.mongodb))

> `@DataMongoTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)

下面的类显示了`@DataMongoTest`注解的使用情况。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;
import org.springframework.data.mongodb.core.MongoTemplate;

@DataMongoTest
class MyDataMongoDbTests {

    @Autowired
    private MongoTemplate mongoTemplate;

    // ...

}
```

内存中的嵌入式MongoDB通常对测试很有效，因为它速度快，而且不需要任何开发人员安装。然而，如果你更喜欢针对真正的MongoDB服务器运行测试，你应该排除嵌入式MongoDB的自动配置，如下例所示。

```java
import org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;

@DataMongoTest(excludeAutoConfiguration = EmbeddedMongoAutoConfiguration.class)
class MyDataMongoDbTests {

    // ...

}
```

#### 26.3.21. 自动配置的数据Neo4j测试

你可以使用`@DataNeo4jTest`来测试Neo4j应用程序。默认情况下，它扫描`@Node`类，并配置Spring Data Neo4j存储库。当使用`@DataNeo4jTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。(关于在Spring Boot中使用Neo4J的更多信息，请参阅本章前面的[Neo4j](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.neo4j))。

> `@DataNeo4jTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

下面的例子显示了在Spring Boot中使用Neo4J测试的一个典型设置。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;

@DataNeo4jTest
class MyDataNeo4jTests {

    @Autowired
    private SomeRepository repository;

    // ...

}
```

默认情况下，Data Neo4j测试是事务性的，并在每次测试结束后回滚。更多细节请参见Spring框架参考文档中的[相关章节](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/testing.html#testcontext-tx-enabling-transactions)。如果这不是你想要的，你可以为一个测试或整个类停用事务管理，如下所示。

```java
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@DataNeo4jTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class MyDataNeo4jTests {

}
```

```java
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@DataNeo4jTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class MyDataNeo4jTests {

}
```

> 事务测试不支持响应式访问。如果你使用这种风格，你必须如上所述配置`@DataNeo4jTest`测试。

#### 26.3.22. 自动配置的数据Redis测试

你可以使用`@DataRedisTest`来测试Redis应用程序。默认情况下，它扫描`@RedisHash`类并配置Spring Data Redis repositories。当使用`@DataRedisTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。(关于在Spring Boot中使用Redis的更多信息，请参阅本章前面的[Redis](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.redis))

> `@DataRedisTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

下面的例子显示了`@DataRedisTest`注解的使用情况。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.redis.DataRedisTest;

@DataRedisTest
class MyDataRedisTests {

    @Autowired
    private SomeRepository repository;

    // ...

}
```

#### 26.3.23. 自动配置的数据LDAP测试

你可以使用`@DataLdapTest`来测试LDAP应用程序。默认情况下，它配置内存中的嵌入式LDAP（如果可用），配置`LdapTemplate`，扫描`@Entry`类，并配置Spring Data LDAP库。当使用`@DataLdapTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。(关于在Spring Boot中使用LDAP的更多信息，请参阅本章前面的[LDAP](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.nosql.ldap))。

> `@DataLdapTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

下面的例子显示了`@DataLdapTest`注解的使用情况。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.ldap.DataLdapTest;
import org.springframework.ldap.core.LdapTemplate;

@DataLdapTest
class MyDataLdapTests {

    @Autowired
    private LdapTemplate ldapTemplate;

    // ...

}
```

内存中的嵌入式 LDAP 通常对测试很有效，因为它速度快，而且不需要任何开发人员安装。然而，如果你喜欢针对真实的LDAP服务器运行测试，你应该排除嵌入式LDAP的自动配置，如下面的例子所示。

```java
import org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.ldap.DataLdapTest;

@DataLdapTest(excludeAutoConfiguration = EmbeddedLdapAutoConfiguration.class)
class MyDataLdapTests {

    // ...

}
```

#### 26.3.24. 自动配置的REST客户端

你可以使用`@RestClientTest`注解来测试REST客户端。默认情况下，它自动配置Jackson、GSON和Jsonb支持，配置`RestTemplateBuilder`，并增加对`MockRestServiceServer`的支持。当使用`@RestClientTest`注解时，常规的`@Component`和`@ConfigurationProperties` Bean不会被扫描。`@EnableConfigurationProperties`可以用来包括`@ConfigurationProperties` Bean。

> `@RestClientTest`启用的自动配置设置列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

你想测试的特定Bean应该通过使用`@RestClientTest`的`value`或`components`属性来指定，如以下例子所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.client.RestClientTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.client.MockRestServiceServer;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.client.match.MockRestRequestMatchers.requestTo;
import static org.springframework.test.web.client.response.MockRestResponseCreators.withSuccess;

@RestClientTest(RemoteVehicleDetailsService.class)
class MyRestClientTests {

    @Autowired
    private RemoteVehicleDetailsService service;

    @Autowired
    private MockRestServiceServer server;

    @Test
    void getVehicleDetailsWhenResultIsSuccessShouldReturnDetails() throws Exception {
        this.server.expect(requestTo("/greet/details")).andRespond(withSuccess("hello", MediaType.TEXT_PLAIN));
        String greeting = this.service.callRestService();
        assertThat(greeting).isEqualTo("hello");
    }

}
```

#### 26.3.25. 自动配置的Spring REST文档测试

你可以使用`@AutoConfigureRestDocs`注解来在你的测试中使用[Spring REST Docs](https://spring.io/projects/spring-restdocs)与Mock MVC、REST Assured或WebTestClient。它消除了对Spring REST Docs中JUnit扩展的需求。

`@AutoConfigureRestDocs`可用于覆盖默认输出目录（如果你使用Maven，则为`target/generated-snippets`，如果你使用Gradle，则为`build/generated-snippets`）。它还可以用来配置出现在任何文档化URI中的主机、方案和端口。

##### 用Mock MVC自动配置的Spring REST文档测试

`@AutoConfigureRestDocs`定制了`MockMvc` Bean，以便在测试基于Servlet的Web应用程序时使用Spring REST Docs。你可以通过使用`@Autowired`来注入它，并在你的测试中使用它，就像你在使用Mock MVC和Spring REST Docs时通常所做的那样，如以下例子所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(UserController.class)
@AutoConfigureRestDocs
class MyUserDocumentationTests {

    @Autowired
    private MockMvc mvc;

    @Test
    void listUsers() throws Exception {
        this.mvc.perform(get("/users").accept(MediaType.TEXT_PLAIN))
            .andExpect(status().isOk())
            .andDo(document("list-users"));
    }

}
```

如果你需要对Spring REST Docs的配置进行更多的控制，而不是由`@AutoConfigureRestDocs`的属性提供，你可以使用`RestDocsMockMvcConfigurationCustomizer` Bean，如以下例子所示。

```java
import org.springframework.boot.test.autoconfigure.restdocs.RestDocsMockMvcConfigurationCustomizer;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.restdocs.mockmvc.MockMvcRestDocumentationConfigurer;
import org.springframework.restdocs.templates.TemplateFormats;

@TestConfiguration(proxyBeanMethods = false)
public class MyRestDocsConfiguration implements RestDocsMockMvcConfigurationCustomizer {

    @Override
    public void customize(MockMvcRestDocumentationConfigurer configurer) {
        configurer.snippets().withTemplateFormat(TemplateFormats.markdown());
    }

}
```

如果你想利用Spring REST Docs对参数化输出目录的支持，你可以创建一个`RestDocumentationResultHandler` Bean。自动配置用这个结果处理程序调用`alwaysDo`，从而使每个`MockMvc`调用自动生成默认的片段。下面的例子显示了一个`RestDocumentationResultHandler`被定义。

```java
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.restdocs.mockmvc.MockMvcRestDocumentation;
import org.springframework.restdocs.mockmvc.RestDocumentationResultHandler;

@TestConfiguration(proxyBeanMethods = false)
public class MyResultHandlerConfiguration {

    @Bean
    public RestDocumentationResultHandler restDocumentation() {
        return MockMvcRestDocumentation.document("{method-name}");
    }

}
```

##### 用WebTestClient自动配置的Spring REST文档测试

`@AutoConfigureRestDocs`也可以在测试响应式Web应用时与`WebTestClient`一起使用。你可以通过使用`@Autowired`来注入它，并在你的测试中使用它，就像你在使用`@WebFluxTest`和Spring REST Docs时通常使用的那样，如以下例子所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.springframework.restdocs.webtestclient.WebTestClientRestDocumentation.document;

@WebFluxTest
@AutoConfigureRestDocs
class MyUsersDocumentationTests {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void listUsers() {
        this.webTestClient
            .get().uri("/")
        .exchange()
        .expectStatus()
            .isOk()
        .expectBody()
            .consumeWith(document("list-users"));
    }

}
```

如果你需要对Spring REST Docs配置进行更多的控制，而不是由`@AutoConfigureRestDocs`的属性提供，你可以使用`RestDocsWebTestClientConfigurationCustomizer` Bean，如下面的例子所示。

```java
import org.springframework.boot.test.autoconfigure.restdocs.RestDocsWebTestClientConfigurationCustomizer;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.restdocs.webtestclient.WebTestClientRestDocumentationConfigurer;

@TestConfiguration(proxyBeanMethods = false)
public class MyRestDocsConfiguration implements RestDocsWebTestClientConfigurationCustomizer {

    @Override
    public void customize(WebTestClientRestDocumentationConfigurer configurer) {
        configurer.snippets().withEncoding("UTF-8");
    }

}
```

##### 用REST保证的自动配置的Spring REST文档测试

`@AutoConfigureRestDocs`使一个`RequestSpecification` Bean，预先配置为使用Spring REST Docs，可用于你的测试。你可以通过使用`@Autowired`来注入它，并在你的测试中使用它，就像你在使用REST Assured和Spring REST Docs时一样，如以下例子所示。

```java
import io.restassured.specification.RequestSpecification;
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.web.server.LocalServerPort;

import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.is;
import static org.springframework.restdocs.restassured3.RestAssuredRestDocumentation.document;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureRestDocs
class MyUserDocumentationTests {

    @Test
    void listUsers(@Autowired RequestSpecification documentationSpec, @LocalServerPort int port) {
        given(documentationSpec)
            .filter(document("list-users"))
        .when()
            .port(port)
            .get("/")
        .then().assertThat()
            .statusCode(is(200));
    }

}
```

如果你需要对Spring REST Docs的配置进行更多的控制，而不是由`@AutoConfigureRestDocs`的属性提供，可以使用`RestDocsRestAssuredConfigurationCustomizer` Bean，如下所示。

```java
import org.springframework.boot.test.autoconfigure.restdocs.RestDocsRestAssuredConfigurationCustomizer;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.restdocs.restassured3.RestAssuredRestDocumentationConfigurer;
import org.springframework.restdocs.templates.TemplateFormats;

@TestConfiguration(proxyBeanMethods = false)
public class MyRestDocsConfiguration implements RestDocsRestAssuredConfigurationCustomizer {

    @Override
    public void customize(RestAssuredRestDocumentationConfigurer configurer) {
        configurer.snippets().withTemplateFormat(TemplateFormats.markdown());
    }

}
```

#### 26.3.26. 自动配置的 Spring Web 服务测试

你可以使用`@WebServiceClientTest`来测试使用Spring Web服务项目调用Web服务的应用程序。默认情况下，它配置了一个模拟的`WebServiceServer` Bean，并自动定制了你的`WebServiceTemplateBuilder`。 (关于用Spring Boot使用Web服务的更多信息，请参阅本章前面的 [Web服务](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.webservices))。

> `@WebServiceClientTest`启用的自动配置设置的列表可以[在附录中找到](https://docs.spring.io/spring-boot/docs/current/reference/html/test-auto-configuration.html#test-auto-configuration)。

下面的例子显示了`@WebServiceClientTest`注解的使用。

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.webservices.client.WebServiceClientTest;
import org.springframework.ws.test.client.MockWebServiceServer;
import org.springframework.xml.transform.StringSource;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.ws.test.client.RequestMatchers.payload;
import static org.springframework.ws.test.client.ResponseCreators.withPayload;

@WebServiceClientTest(SomeWebService.class)
class MyWebServiceClientTests {

    @Autowired
    private MockWebServiceServer server;

    @Autowired
    private SomeWebService someWebService;

    @Test
    void mockServerCall() {
        this.server
            .expect(payload(new StringSource("<request/>")))
            .andRespond(withPayload(new StringSource("<response><status>200</status></response>")));
        assertThat(this.someWebService.test())
            .extracting(Response::getStatus)
            .isEqualTo(200);
    }

}
```

#### 26.3.27. 额外的自动配置和片段

每个片段提供一个或多个`@AutoConfigure...`注解，即定义自动配置，应该作为片段的一部分。通过创建一个自定义的`@AutoConfigure...`注解或在测试中添加`@ImportAutoConfiguration`，可以在每个测试的基础上添加额外的自动配置，如下例所示。

```java
import org.springframework.boot.autoconfigure.ImportAutoConfiguration;
import org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration;
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;

@JdbcTest
@ImportAutoConfiguration(IntegrationAutoConfiguration.class)
class MyJdbcTests {

}
```

> 请确保不要使用常规的`@Import`注解来导入自动配置，因为Spring Boot会以特定的方式处理这些配置。

另外，通过在`META-INF/spring.plants`中注册，可以为slice注解的任何使用添加额外的自动配置，如下例所示。

```properties
org.springframework.boot.test.autoconfigure.jdbc.JdbcTest=com.example.IntegrationAutoConfiguration
```

> 只要用`@ImportAutoConfiguration` 进行元注释，切片或`@AutoConfigure...` 注解就可以用这种方式定制。

#### 26.3.28. 用户配置和片段

如果你以合理的方式[构建你的代码](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code)，你的`@SpringBootApplication`类就会[默认使用](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.detecting-configuration)作为你的测试配置。

因此，不要在应用程序的主类中加入针对其功能的特定区域的配置设置，这一点很重要。

假设你正在使用Spring Batch，并且你依赖它的自动配置。你可以这样定义你的`@SpringBootApplication`。

```java
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableBatchProcessing
public class MyApplication {

    // ...

}
```

因为这个类是测试的源配置，任何切片测试实际上都试图启动Spring Batch，这绝对不是你想做的。推荐的方法是将特定区域的配置移到与你的应用程序相同级别的单独的`@Configuration`类中，如下面的例子所示。

```java
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
@EnableBatchProcessing
public class MyBatchConfiguration {

    // ...

}
```

> 根据你的应用程序的复杂性，你可以有一个单一的`@Configuration`类用于你的定制，或者每个域区有一个类。后一种方法可以让你在你的一个测试中启用它，如果有必要的话，用`@Import`注解。

测试片从扫描中排除了`@Configuration`类。例如，对于一个`@WebMvcTest`，下面的配置将不包括测试片加载的应用程序上下文中的给定`WebMvcConfigurer` Bean。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration(proxyBeanMethods = false)
public class MyWebConfiguration {

    @Bean
    public WebMvcConfigurer testConfigurer() {
        return new WebMvcConfigurer() {
            // ...
        };
    }

}
```

然而，下面的配置将导致自定义的`WebMvcConfigurer`被测试片段加载。

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Component
public class MyWebMvcConfigurer implements WebMvcConfigurer {

    // ...

}
```

混乱的另一个来源是classpath扫描。假设在你以合理的方式结构化你的代码时，你需要扫描一个额外的包。你的应用程序可能类似于下面的代码。

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan({ "com.example.app", "com.example.another" })
public class MyApplication {

    // ...

}
```

这样做有效地覆盖了默认的组件扫描指令，其副作用是扫描这两个包，而不管你选择的是哪一个片段。例如，一个`@DataJpaTest`似乎突然扫描了你的应用程序的组件和用户配置。同样，将自定义指令移到一个单独的类中是解决这个问题的好方法。

> 如果这对你来说不是一个选择，你可以在你的测试层次结构的某个地方创建一个`@SpringBootConfiguration`，这样它就会被使用。或者，你可以为你的测试指定一个源，这将禁用寻找默认源的行为。

#### 26.3.29. 使用Spock来测试Spring Boot应用程序

Spock 2.x可以用来测试Spring Boot应用程序。要做到这一点，需要在你的应用程序的构建中添加对Spock的`spock-spring`模块的依赖。`spock-spring`将Spring的测试框架集成到Spock中。更多细节请参见[Spock的Spring模块文档](https://spockframework.org/spock/docs/2.0/modules.html#_spring_module)。

### 26.4. 测试工具

一些在测试你的应用程序时通常有用的测试工具类被打包成`spring-boot`的一部分。

#### 26.4.1. ConfigDataApplicationContextInitializer

`ConfigDataApplicationContextInitializer`是一个`ApplicationContextInitializer`，你可以应用于你的测试来加载Spring Boot`application.properties`文件。当你不需要`@SpringBootTest` 提供的全部功能时，你可以使用它，如下面的例子所示。

```java
import org.springframework.boot.test.context.ConfigDataApplicationContextInitializer;
import org.springframework.test.context.ContextConfiguration;

@ContextConfiguration(classes = Config.class, initializers = ConfigDataApplicationContextInitializer.class)
class MyConfigFileTests {

    // ...

}
```

> 单独使用`ConfigDataApplicationContextInitializer`并不提供对`@Value("${..}")`注入的支持。它唯一的工作是确保`application.properties`文件被加载到Spring的`Environment`。对于`@Value` 支持，你需要额外配置一个 `PropertySourcesPlaceholderConfigurer`，或者使用`@SpringBootTest`，它可以为你自动配置一个。

#### 26.4.2. 测试属性值（TestPropertyValues）

`TestPropertyValues`让你快速添加属性到一个`ConfigurableEnvironment`或`ConfigurableApplicationContext`。你可以用`key=value`字符串来调用它，如下所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.boot.test.util.TestPropertyValues;
import org.springframework.mock.env.MockEnvironment;

import static org.assertj.core.api.Assertions.assertThat;

class MyEnvironmentTests {

    @Test
    void testPropertySources() {
        MockEnvironment environment = new MockEnvironment();
        TestPropertyValues.of("org=Spring", "name=Boot").applyTo(environment);
        assertThat(environment.getProperty("name")).isEqualTo("Boot");
    }

}
```

#### 26.4.3. 输出捕获（OutputCapture）

`OutputCapture`是一个JUnit的`扩展`，你可以用来捕获`System.out`和`System.err`输出。使用时，添加`@ExtendWith(OutputCaptureExtension.class)`，并将`CapturedOutput`作为参数注入你的测试类构造函数或测试方法，如下所示。

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

import org.springframework.boot.test.system.CapturedOutput;
import org.springframework.boot.test.system.OutputCaptureExtension;

import static org.assertj.core.api.Assertions.assertThat;

@ExtendWith(OutputCaptureExtension.class)
class MyOutputCaptureTests {

    @Test
    void testName(CapturedOutput output) {
        System.out.println("Hello World!");
        assertThat(output).contains("World");
    }

}
```

#### 26.4.4. TestRestTemplate

`TestRestTemplate`是Spring的`RestTemplate`的便利替代品，在集成测试中很有用。你可以得到一个虚无缥缈的模板，或者一个发送Basic HTTP认证（有用户名和密码）的模板。在这两种情况下，模板都是容错的。这意味着它的行为对测试有利，不会在4xx和5xx错误时抛出异常。相反，这种错误可以通过返回的 `ResponseEntity` 和它的状态代码来检测。

> Spring Framework 5.0提供了一个新的`WebTestClient`，可用于[WebFlux集成测试](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.spring-webflux-tests)和[WebFlux和MVC端到端测试](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.with-running-server)。它为断言提供了一个流畅的API，与`TestRestTemplate`不同。

建议使用Apache HTTP客户端（版本4.3.2或更高），但不是必须的。如果你的classpath上有这个客户端，`TestRestTemplate`会通过适当配置客户端来响应。如果你使用Apache的HTTP客户端，一些额外的测试友好功能将被启用。

* 重定向不被跟踪（所以你可以断定响应位置）。
* 忽略Cookies（所以template是无状态的）。

`TestRestTemplate`可以在你的集成测试中直接实例化，如以下例子所示。

```java
import org.junit.jupiter.api.Test;

import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

class MyTests {

    private TestRestTemplate template = new TestRestTemplate();

    @Test
    void testRequest() throws Exception {
        ResponseEntity<String> headers = this.template.getForEntity("https://myhost.example.com/example", String.class);
        assertThat(headers.getHeaders().getLocation()).hasHost("other.example.com");
    }

}
```

另外，如果你使用`@SpringBootTest`注解与`WebEnvironment.RANDOM_PORT`或`WebEnvironment.DEFINED_PORT`，你可以注入一个完全配置的`TestRestTemplate`并开始使用它。如果有必要，可以通过`RestTemplateBuilder` Bean应用额外的定制。任何没有指定主机和端口的URL都会自动连接到嵌入式服务器，如以下例子所示。

```java
import java.time.Duration;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.http.HttpHeaders;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class MySpringBootTests {

    @Autowired
    private TestRestTemplate template;

    @Test
    void testRequest() {
        HttpHeaders headers = this.template.getForEntity("/example", String.class).getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }

    @TestConfiguration(proxyBeanMethods = false)
    static class RestTemplateBuilderConfiguration {

        @Bean
        RestTemplateBuilder restTemplateBuilder() {
            return new RestTemplateBuilder().setConnectTimeout(Duration.ofSeconds(1))
                    .setReadTimeout(Duration.ofSeconds(1));
        }

    }

}
```

## 27. WebSockets

TODO

{{#include ../license.md}}
