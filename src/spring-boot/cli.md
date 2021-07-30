# Spring Boot CLI

Spring Boot CLI是一个命令行工具，如果你想快速开发一个Spring应用，可以使用它。它可以让你运行Groovy脚本，这意味着你有一个熟悉的类似Java的语法，而没有那么多模板代码。你也可以引导一个新的项目或为其编写自己的命令。

# 1.安装 CLI

Spring Boot CLI（命令行界面）可以通过使用SDKMAN! (SDK管理器)或使用Homebrew或MacPorts（如果您是OSX用户）来手动安装。请参阅 "入门 "部分的[getting-started.html](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.installing.cli) ，了解全面的安装说明。

# 2.使用 CLI

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

## 2.1.运行 程序使用 CLI

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

### 2.1.1.推断出的 "grab "依赖性

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


### 2.1.2. 推断出 "依赖拉取 "坐标

`Spring Boot`扩展了Groovy的标准`@Grab`支持，允许你指定一个没有组或版本的依赖关系（例如，`@Grab('freemarker')）`。这样做会查阅`Spring Boot`的默认依赖元数据来推断工件的组和版本。

**笔记**
>默认元数据与你使用的CLI版本相联系。它只在你转移到CLI的新版本时才会改变，使你能够控制你的依赖关系的版本何时改变。在[附录](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#dependency-versions )中可以找到一个显示默认元数据中包含的依赖关系及其版本的表格。


### 2.1.3. 默认的导入语句

为了帮助减少你的Groovy代码的大小，几个导入语句被自动包括在内。注意前面的例子是如何引用`@Component`、`@RestController`和`@RequestMapping`的，而不需要使用完全限定的名称或导入语句。

**提示**
>许多Spring注解在不使用导入语句的情况下工作。在添加导入语句之前，试着运行你的应用程序，看看有什么失败的地方。

### 2.1.4. 自动为您创建main方法

与同等的Java应用程序不同，你不需要在你的Groovy脚本中包含一个公共静态`void main(String[] args)`方法。一个SpringApplication会被自动创建，你的编译后的代码会作为源码。

### 2.1.5. 自定义依赖性管理

默认情况下，CLI在解析@Grab依赖关系时使用`spring-boot-dependencies`中声明的依赖关系管理。可以通过使用`@DependencyManagementBom`注解来配置额外的依赖性管理，它覆盖了默认的依赖性管理。该注解的值应指定一个或多个Maven BOM的坐标`（groupId:artifactId:version）`。

例如，考虑下面的声明。
```java
@DependencyManagementBom("com.example.custom-bom:1.0.0")

```

前面的声明在`com/example/custom-versions/1.0.0/`下的Maven仓库中获取了`custom-bom-1.0.0.pom`。

当你指定多个BOM时，它们会按照你声明的顺序应用，如下例所示。

```java
@DependencyManagementBom([
    "com.example.custom-bom:1.0.0",
    "com.example.another-bom:1.0.0"])

```

前面的例子表明，Another-bom中的依赖性管理覆盖了custom-bom中的依赖性管理。

你可以在任何可以使用 @Grab 的地方使用` @DependencyManagementBom`。然而，为了确保依赖管理的顺序一致，你在你的应用程序中最多只能使用一次`@DependencyManagementBom`。



## 2.2. 具有多个源文件的应用程序

你可以在所有接受文件输入的命令中使用 `"shell globbing"`。这样做可以让你从一个目录中使用多个文件，如下面的例子所示。

```java
$ spring run *.groovy

```
## 2.3. 打包你的程序

你可以使用jar命令将你的应用程序打包成一个独立的可执行的jar文件，如下面的例子中所示。

```java
$ spring jar my-app.jar *.groovy

```

由此产生的jar文件包含了通过编译应用程序产生的类和应用程序的所有依赖项，这样就可以通过使用java -jar来运行它。该jar文件还包含了应用程序`classpath`的条目。你可以通过使用`-include`和`-exclude`来添加和删除jar的明确路径。两者都是以逗号分隔的，并且都接受前缀，以 "+"和"-"的形式，表示应该从默认值中删除它们。默认的包括如下。

```java
public/**, resources/**, static/**, templates/**, META-INF/**, *

```

默认的排除法如下。

```java
.*, repository/**, build/**, target/**, **/*.jar, **/*.groovy

```

在命令行上输入`spring help jar`以获得更多信息。

## 2.4. 新初始化一个项目

init命令让你在不离开shell的情况下通过start.spring.io创建一个新项目，如下例所示。


```java
$ spring init --dependencies=web,data-jpa my-project
Using service at https://start.spring.io
Project extracted to '/Users/developer/example/my-project'

```

前面的例子创建了一个`my-project`目录，其中有一个基于Maven的项目，使用`spring-boot-starter-web`和`spring-boot-starter-data-jpa`。你可以通过使用-list标志列出服务的能力，如下例所示。

```java
$ spring init --list
=======================================
Capabilities of https://start.spring.io
=======================================

Available dependencies:
-----------------------
actuator - Actuator: Production ready features to help you monitor and manage your application
...
web - Web: Support for full-stack web development, including Tomcat and spring-webmvc
websocket - Websocket: Support for WebSocket development
ws - WS: Support for Spring Web Services

Available project types:
------------------------
gradle-build -  Gradle Config [format:build, build:gradle]
gradle-project -  Gradle Project [format:project, build:gradle]
maven-build -  Maven POM [format:build, build:maven]
maven-project -  Maven Project [format:project, build:maven] (default)

...

```

init命令支持许多选项。更多细节请参见帮助输出。例如，下面的命令创建了一个使用Java 8和war打包的Gradle项目。

```java
$ spring init --build=gradle --java-version=1.8 --dependencies=websocket --packaging=war sample-app.zip
Using service at https://start.spring.io
Content saved to 'sample-app.zip'

```

## 2.5. 使用嵌入式shell

Spring Boot包括`BASH`和`zsh shells`的命令行完成脚本。如果你不使用这两个shell（也许你是Windows用户），你可以使用shell命令来启动一个集成shell，如下例所示。

```java
$ spring shell
Spring Boot (v2.5.3)
Hit TAB to complete. Type \'help' and hit RETURN for help, and \'exit' to quit.

```

在嵌入式shell内，你可以直接运行其他命令。

```java
$ version
Spring CLI v2.5.3

```
内嵌的shell支持ANSI颜色输出以及tab完成。如果你需要运行一个本地命令，你可以使用！前缀。要退出嵌入式外壳，按`ctrl-c`。

## 2.6. 向CLI添加扩展程序

你可以通过使用安装命令向CLI添加扩展。该命令接收一组或多组工件坐标，格式为`group:artifact:version`，如下面的例子所示。

```java
$ spring install com.example:spring-boot-cli-extension:1.0.0.RELEASE

```

除了安装由你提供的坐标确定的工件外，所有工件的依赖关系也被安装。

要卸载一个依赖关系，使用卸载命令。与安装命令一样，它需要一组或多组工件坐标，格式为`group:artifact:version`，如下面的例子所示。

```java
$ spring uninstall com.example:spring-boot-cli-extension:1.0.0.RELEASE

```
它卸载由你提供的坐标识别的工件和它们的依赖关系。

要卸载所有额外的依赖，你可以使用`--all`选项，如下面的例子所示。

```java
$ spring uninstall --all

```


# 3. 使用Groovy Beans DSL开发应用程序
`Spring Framework 4.0`原生支持`beans{}"DSL"`（从Grails借来的），你可以通过使用相同的格式在Groovy应用脚本中嵌入bean定义。有时这也是包含中间件声明等外部特性的好方法，如下例所示。

```java
@Configuration(proxyBeanMethods = false)
class Application implements CommandLineRunner {

    @Autowired
    SharedService service

    @Override
    void run(String... args) {
        println service.message
    }

}

import my.company.SharedService

beans {
    service(SharedService) {
        message = "Hello World"
    }
}

```
你可以将类的声明与Bean{}混合在同一个文件中，只要它们保持在最高级别，或者，如果你愿意，你可以将Bean DSL放在一个单独的文件中。

# 4. 用settings.xml配置CLI

Spring Boot CLI使用Maven的依赖性解析引擎Aether来解析依赖性。CLI利用~/.m2/settings.xml中的Maven配置来配置Aether。CLI会遵守以下配置设置。

- Offline
- Mirrors
- Servers
- Proxies
- Profiles
    - Activation
    - Repositories
- Active profiles

更多信息请参见[Maven的设置文档]( https://maven.apache.org/settings.html )。

# 5. 接下来要读什么


在GitHub仓库里有一些[groovy脚本样本]( https://github.com/spring-projects/spring-boot/tree/v2.5.3/spring-boot-project/spring-boot-cli/samples )，你可以用它来试用Spring Boot CLI。源代码中还有大量的Javadoc。

如果你发现你已经达到了CLI工具的极限，你可能想把你的应用程序转换为一个完整的Gradle或Maven构建的 "Groovy项目"。下一节将介绍Spring Boot的 ["构建工具插件"]( https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins.html#build-tool-plugins )，你可以在Gradle或Maven中使用这些插件。




{{#include ../license.md}}