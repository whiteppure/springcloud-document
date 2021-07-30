# Spring Boot CLI

Spring Boot CLI是一个命令行工具，如果你想快速开发一个Spring应用，可以使用它。它可以让你运行Groovy脚本，这意味着你有一个熟悉的类似Java的语法，而没有那么多模板代码。你也可以引导一个新的项目或为其编写自己的命令。

# 安装 CLI

Spring Boot CLI（命令行界面）可以通过使用SDKMAN! (SDK管理器)或使用Homebrew或MacPorts（如果您是OSX用户）来手动安装。请参阅 "入门 "部分的[getting-started.html](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.installing.cli) ，了解全面的安装说明。

# 使用 CLI

一旦你安装了CLI，你可以在命令行输入spring并按回车键来运行它。如果你在没有任何参数的情况下运行spring，会显示一个帮助屏幕，如下所示。

```java
$ spring
usage: spring [--help] [--version]
       <command> [<args>]

Available commands are:

  run [options] <files> [--] [args]
    Run a spring groovy script

  _... more command help is shown here_

```
你可以键入`spring help`来获得关于任何支持的命令的更多细节，如以下例子所示。

```java
$ spring help run
spring run - Run a spring groovy script

usage: spring run [options] <files> [--] [args]

Option                     Description
------                     -----------
--autoconfigure [Boolean]  Add autoconfigure compiler
                             transformations (default: true)
--classpath, -cp           Additional classpath entries
--no-guess-dependencies    Do not attempt to guess dependencies
--no-guess-imports         Do not attempt to guess imports
-q, --quiet                Quiet logging
-v, --verbose              Verbose logging of dependency
                             resolution
--watch                    Watch the specified file for changes

```

`version`命令提供了一种快速的方法来检查你所使用的`Spring Boot`的版本，如下所示。

```java
$ spring version
Spring CLI v2.5.3
```

# 运行 CLI

您可以通过使用运行命令来编译和运行`Groovy`源代码。`Spring Boot CLI`是完全独立的，所以你不需要安装任何外部Groovy。

下面的例子显示了一个用`Groovy`编写的 `"hello world "`网络应用。

- hello.groovy

```java
@RestController
class WebApplication {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}

```
要编译和运行该应用程序，请输入以下命令。

```java
$ spring run hello.groovy
```

要向应用程序传递命令行参数，使用`--`将命令与 `"spring "`命令参数分开，如下例所示。

```java
$ spring run hello.groovy -- --server.port=9000

```

要设置 JVM 命令行参数，可以使用 JAVA_OPTS 环境变量，如下例所示。

```java
$ JAVA_OPTS=-Xmx1024m spring run hello.groovy

```

**温馨提示**
> 在 `Microsoft Windows` 上设置 `JAVA_OPTS` 时，请确保引用整个指令，例如设置` "JAVA_OPTS=-Xms256m -Xmx2048m"`。这样做可以确保数值被正确地传递给进程。

# 推断出的 "grab "依赖性

标准的Groovy包括一个`@Grab`注解，它可以让你声明对第三方库的依赖性。这项有用的技术让Groovy以与Maven或Gradle相同的方式下载jars，但不要求你使用构建工具。

`Spring Boot`进一步扩展了这种技术，并试图根据你的代码推断出要 `"拉取 "`的库。例如，由于之前显示的WebApplication代码使用了`@RestController`注解，Spring Boot会拉取 `"Tomcat "和 "Spring MVC"`。

以下各项被用来作为 `"拉取 提示"`。

| Items | Grabs |
|-----|-----|
|JdbcTemplate, NamedParameterJdbcTemplate, DataSource| JDBC Application.   |
| @EnableJms | JMS Application.   |
| @EnableCaching | Caching abstraction.   |
| @Test | JUnit.   |
| @EnableRabbit | RabbitMQ.   |
|extends Specification| Spock test.   |
| @EnableBatchProcessing | Spring Batch.   |
| @MessageEndpoint @EnableIntegration |Spring Integration.   |
| @Controller @RestController @EnableWebMvc |Spring MVC + Embedded Tomcat.   |
| @EnableWebSecurity |Spring Security.   |
| @EnableTransactionManagement |Spring Transaction Management.   |

**温馨提示**
>请参阅Spring Boot CLI源代码中的CompilerAutoConfiguration子类，以了解定制的确切应用方式。


# 推断出 "依赖拉取 "坐标

`Spring Boot`扩展了Groovy的标准`@Grab`支持，允许你指定一个没有组或版本的依赖关系（例如，`@Grab('freemarker')）`。这样做会查阅`Spring Boot`的默认依赖元数据来推断工件的组和版本。

**笔记**
>默认元数据与你使用的CLI版本相联系。它只在你转移到CLI的新版本时才会改变，使你能够控制你的依赖关系的版本何时改变。在[附录](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#dependency-versions )中可以找到一个显示默认元数据中包含的依赖关系及其版本的表格。


# 默认的导入语句

为了帮助减少你的Groovy代码的大小，几个导入语句被自动包括在内。注意前面的例子是如何引用`@Component`、`@RestController`和`@RequestMapping`的，而不需要使用完全限定的名称或导入语句。

**提示**
>许多Spring注解在不使用导入语句的情况下工作。在添加导入语句之前，试着运行你的应用程序，看看有什么失败的地方。

# 自动为您创建main方法

与同等的Java应用程序不同，你不需要在你的Groovy脚本中包含一个公共静态`void main(String[] args)`方法。一个SpringApplication会被自动创建，你的编译后的代码会作为源码。



{{#include ../license.md}}