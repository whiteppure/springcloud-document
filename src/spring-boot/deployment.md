# 部署SpringBoot应用

Spring Boot灵活的打包选项在部署应用程序时提供了大量的选择。你可以将Spring Boot应用部署到各种云平台、容器镜像（如Docker）或虚拟/真实机器上。

本节涵盖了一些比较常见的部署场景。

## 1. 部署到容器

如果从容器运行应用程序，则可以使用可执行jar，但将其分解并以不同的方式运行通常也是一个优势。某些PaaS实现还可能选择在运行归档文件之前解包。例如，Cloud Foundry就是这样操作的。运行解压后的归档文件的一种方法是启动相应的启动程序，如下所示:

```java
$ jar -xf myapp.jar
$ java org.springframework.boot.loader.JarLauncher
```
这实际上在启动时(取决于jar的大小)比从未展开的归档文件中运行要快一些。在运行时，您不应该期望有任何差异。
一旦你解压了jar文件，你还可以通过使用“自然的”主方法而不是JarLauncher来运行应用程序，从而额外提高启动时间。例如:


```java
$ jar -xf myapp.jar
$ java -cp BOOT-INF/classes:BOOT-INF/lib/* com.example.MyApplication
```

**笔记**
>在应用程序的主方法上使用JarLauncher有一个可预测的类路径顺序的额外好处。jar包含一个类路径。当构造类路径时，JarLauncher会使用这个idx文件。

更有效的容器映像还可以通过为依赖项、应用程序类和资源(通常更改更频繁)创建单独的层来创建。

## 2. 部署到云平台

Spring Boot的可执行jar是为大多数流行的云平台即服务(Platform-as-a-Service)提供商准备的。这些提供商往往要求您“自带您自己的容器”。它们管理应用程序进程(不是专门的Java应用程序)，因此它们需要一个中间层，使您的应用程序适应云的运行进程概念。



两家流行的云提供商Heroku和cloud Foundry采用了“构建包”方法。构建包将部署的代码包装在启动应用程序所需的任何内容中。它可能是一个JDK和对java的调用，一个嵌入式web服务器，或者一个成熟的应用服务器。构建包是可插拔的，但理想情况下，您应该尽可能少地对它进行定制。这减少了不在您控制范围内的功能占用。它最小化了开发环境和生产环境之间的差异。



理想情况下，您的应用程序就像Spring Boot可执行jar一样，将运行所需的所有东西打包在其中。



在本节中，我们将了解如何让我们在“入门”一节中开发的应用程序在云中运行。

### 2.1. [Cloud Foundry](https://baike.baidu.com/item/Cloud%20Foundry/6868029?fr=aladdin)
Cloud Foundry提供默认的构建包，如果没有指定其他的构建包，就会起作用。Cloud Foundry Java buildpack对Spring应用程序(包括Spring Boot)有很好的支持。您可以部署独立的可执行jar应用程序，也可以部署传统的.war打包应用程序。


一旦构建了应用程序(例如，通过使用mvn clean包)并安装了cf命令行工具，就可以使用cf push命令部署应用程序，并替换已编译的.jar的路径。在推送应用程序之前，确保已经使用cf命令行客户端登录。下面一行显示了如何使用cf push命令部署应用程序:

```java
$ cf push acloudyspringtime -p target/demo-0.0.1-SNAPSHOT.jar
```

**笔记**
>在前面的示例中，我们将clouddyspringtime替换为应用程序名称中cf的任何值。

有关更多选项，请参阅[cf push文档](https://docs.cloudfoundry.org/cf-cli/getting-started.html#push )。如果有云铸造清单。Yml文件在同一目录中，它被认为。

此时，cf开始上传您的应用程序，生成类似如下示例的输出:

```java
Uploading acloudyspringtime... OK
Preparing to start acloudyspringtime... OK
-----> Downloaded app package (8.9M)
-----> Java Buildpack Version: v3.12 (offline) | https://github.com/cloudfoundry/java-buildpack.git#6f25b7e
-----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.6s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
       Memory Settings: -Xss349K -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xms681574K -XX:MetaspaceSize=104857K
-----> Downloading Container Certificate Trust Store 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-certificate-trust-store/container-certificate-trust-store-1.0.0_RELEASE.jar (found in cache)
       Adding certificates to .java-buildpack/container_certificate_trust_store/truststore.jks (0.6s)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Checking status of app 'acloudyspringtime'...
  0 of 1 instances running (1 starting)
  ...
  0 of 1 instances running (1 starting)
  ...
  0 of 1 instances running (1 starting)
  ...
  1 of 1 instances running (1 running)

App started
```

祝贺你! 该应用程序现在已经上线了!

一旦你的应用程序上线，你可以通过使用cf apps命令来验证部署的应用程序的状态，如下面的例子所示。

```java
$ cf apps
Getting applications in ...
OK

name                 requested state   instances   memory   disk   urls
...
acloudyspringtime    started           1/1         512M     1G     acloudyspringtime.cfapps.io
...
```
一旦 Cloud Foundry 确认您的应用程序已被部署，您应该能够在给出的 URI 中找到该应用程序。在前面的例子中，您可以在 https://acloudyspringtime.cfapps.io/ 找到它。
#### 2.1.1. 绑定服务

默认情况下，关于正在运行的应用程序的元数据以及服务连接信息会作为环境变量（例如：$VCAP_SERVICES）暴露给应用程序。这一架构决定是由于 Cloud Foundry 的 polyglot（任何语言和平台都可以作为构建包得到支持）性质。进程范围内的环境变量是不分语言的。

环境变量并不总是最简单的 API，因此 Spring Boot 会自动提取它们，并将数据扁平化为可通过 Spring 的环境抽象访问的属性，如以下示例所示。
```java
import org.springframework.context.EnvironmentAware;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class MyBean implements EnvironmentAware {

    private String instanceId;

    @Override
    public void setEnvironment(Environment environment) {
        this.instanceId = environment.getProperty("vcap.application.instance_id");
    }

    // ...

}

```
所有 Cloud Foundry 属性都以 vcap 为前缀。您可以使用 vcap 属性来访问应用程序信息（如应用程序的公共 URL）和服务信息（如数据库凭证）。请参阅 ["CloudFoundryVcapEnvironmentPostProcessor "](https://docs.spring.io/spring-boot/docs/2.5.3/api/org/springframework/boot/cloud/CloudFoundryVcapEnvironmentPostProcessor.html )Javadoc 了解完整的详细信息。

**温馨提示**
>Java CFEnv项目更适合于配置数据源这样的任务。

### 2.2. Kubernetes

Spring Boot通过检查环境中的 "*_SERVICE_HOST "和 "*_SERVICE_PORT "变量，自动检测Kubernetes部署环境。你可以用spring.main.cloud-platform配置属性覆盖这种检测。

Spring Boot帮助你[管理应用程序的状态](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-availability )，并通过使用[Actuator的HTTP Kubernetes Probes](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.kubernetes-probes )导出它。

#### 2.2.1. Kubernetes容器生命周期
当Kubernetes删除一个应用实例时，关闭过程同时涉及几个子系统：关闭钩子、取消注册服务、从负载平衡器中删除实例......由于这种关闭处理是平行发生的（并且由于分布式系统的性质），在这期间有一个窗口，流量可以被路由到一个也已经开始关闭处理的pod。

你可以在preStop处理程序中配置一个睡眠执行，以避免请求被路由到一个已经开始关闭的pod。这个睡眠时间应该足够长，以便新的请求不再被路由到pod，其持续时间将因部署而异。preStop处理程序可以通过pod配置文件中的PodSpec进行配置，如下所示。

```java
spec:
  containers:
  - name: example-container
    image: example-image
    lifecycle:
      preStop:
        exec:
          command: ["sh", "-c", "sleep 10"]
```

一旦预停止钩子完成，SIGTERM将被发送到容器，优雅的关闭将开始，允许任何剩余的正在运行的请求完成。

**笔记**
>当Kubernetes向pod发送一个SIGTERM信号时，它会等待一个指定的时间，称为终止宽限期（默认为30秒）。如果容器在宽限期过后仍在运行，它们会被发送SIGKILL信号并被强行删除。如果pod的关闭时间超过30秒，这可能是因为你增加了spring.lifecycle.timeout-per-shutdown-phase，请确保通过设置Pod YAML中的 terminationGracePeriodSeconds选项来增加终止宽限期。

### 2.3. Heroku

Heroku是另一个流行的PaaS平台。为了定制Heroku的构建，你提供了一个Procfile，它提供了部署一个应用程序所需的咒语。Heroku为Java应用程序分配了一个端口，然后确保路由到外部URI的工作。

你必须配置你的应用程序以监听正确的端口。下面的例子显示了我们的入门REST应用程序的Procfile。

```java
web: java -Dserver.port=$PORT -jar target/demo-0.0.1-SNAPSHOT.jar
```

Spring Boot将-D参数作为可从Spring环境实例中访问的属性。server.port配置属性被反馈给嵌入式Tomcat、Jetty或Undertow实例，然后在启动时使用该端口。$PORT环境变量是由Heroku PaaS分配给我们的。

这应该是你需要的一切。Heroku部署最常见的工作流程是用git推送代码到生产中，如下面的例子所示。
```java
$ git push heroku main
```

这将产生以下结果:

```java
Initializing repository, done.
Counting objects: 95, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (78/78), done.
Writing objects: 100% (95/95), 8.66 MiB | 606.00 KiB/s, done.
Total 95 (delta 31), reused 0 (delta 0)

-----> Java app detected
-----> Installing OpenJDK 1.8... done
-----> Installing Maven 3.3.1... done
-----> Installing settings.xml... done
-----> Executing: mvn -B -DskipTests=true clean install

       [INFO] Scanning for projects...
       Downloading: https://repo.spring.io/...
       Downloaded: https://repo.spring.io/... (818 B at 1.8 KB/sec)
        ....
       Downloaded: https://s3pository.heroku.com/jvm/... (152 KB at 595.3 KB/sec)
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/target/...
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/pom.xml ...
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 59.358s
       [INFO] Finished at: Fri Mar 07 07:28:25 UTC 2014
       [INFO] Final Memory: 20M/493M
       [INFO] ------------------------------------------------------------------------

-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 70.4MB
-----> Launching... done, v6
       https://agile-sierra-1405.herokuapp.com/ deployed to Heroku

To git@heroku.com:agile-sierra-1405.git
 * [new branch]      main -> main
```

你的应用程序现在应该已经在Heroku上运行了。更多细节，请参阅[《将Spring Boot应用部署到Heroku》](https://devcenter.heroku.com/articles/deploying-spring-boot-apps-to-heroku )。

### 2.4. OpenShift
[OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift )有很多资源描述了如何部署Spring Boot应用程序，包括。

- [使用S2I构建器](https://blog.openshift.com/using-openshift-enterprise-grade-spring-boot-deployments/ )
- [架构指南](https://access.redhat.com/documentation/en-us/reference_architectures/2017/html-single/spring_boot_microservices_on_red_hat_openshift_container_platform_3/index )
- [作为一个传统的web应用程序在Wildfly上运行](https://blog.openshift.com/using-spring-boot-on-openshift/ )
- [OpenShift Commons简报](https://blog.openshift.com/openshift-commons-briefing-96-cloud-native-applications-spring-rhoar/ )

## 2.5. Amazon Web Services (AWS)
亚马逊网络服务提供了多种方式来安装基于Spring Boot的应用程序，可以是传统的网络应用程序（war），也可以是嵌入网络服务器的可执行jar文件。这些选项包括。
- AWS Elastic Beanstalk
- AWS代码部署
- AWS运维工作
- AWS云的形成
- AWS容器注册表

每一种都有不同的功能和定价模式。在这份文件中，我们描述了使用AWS Elastic Beanstalk的方法。

#### 2.5.1. AWS Elastic Beanstalk
正如官方Elastic Beanstalk Java[指南](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Java.html )中所描述的，有两个主要的选项来部署Java应用程序。你可以使用 "Tomcat平台 "或 "Java SE平台"。

##### Using the Tomcat Platform
该选项适用于产生war文件的Spring Boot项目。不需要特别的配置。你只需要遵循官方指南。
##### Using the Java SE Platform
该选项适用于产生jar文件并运行嵌入式Web容器的Spring Boot项目。Elastic Beanstalk环境在80端口运行一个nginx实例，以代理在5000端口运行的实际应用程序。要配置它，请在你的application.properties文件中添加以下一行。
```java
server.port=5000
```

**温馨提示**

>上传二进制文件而不是源代码
默认情况下，Elastic Beanstalk会上传源代码，并在AWS中进行编译。然而，最好的办法是上传二进制文件。要做到这一点，请在你的`.elasticbeanstalk/config.yml`文件中添加类似以下的行。
```java
deploy:
    artifact: target/demo-0.0.1-SNAPSHOT.jar
```

**温馨提示**
>通过设置环境类型减少成本
默认情况下，Elastic Beanstalk环境是负载平衡的。负载平衡器有很大的成本。为了避免这种成本，请将环境类型设置为 "单实例"，如[亚马逊文档](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-create-wizard.html#environments-create-wizard-capacity )中所述。你也可以通过使用CLI和以下命令来创建单实例环境。
```java
eb create -s
```

#### 2.5.2. 总结

这是进入AWS的最简单的方法之一，但还有更多的东西需要介绍，比如如何将Elastic Beanstalk集成到任何CI/CD工具中，使用Elastic Beanstalk Maven插件而不是CLI，以及其他。有一篇博文更详细地介绍了这些话题。


### 2.6. Boxfuse和Amazon Web Services

[Boxfuse](https://boxfuse.com/ )的工作原理是将你的Spring Boot可执行的jar或war变成一个最小的虚拟机镜像，可以在VirtualBox或AWS上不变地部署。Boxfuse与Spring Boot深度集成，使用Spring Boot配置文件中的信息来自动配置端口和健康检查URL。Boxfuse利用这些信息来制作镜像以及提供所有资源（实例、安全组、弹性负载均衡器等）。

一旦你创建了[Boxfuse账户](https://console.boxfuse.com/ )，将其连接到AWS账户，安装了最新版本的Boxfuse客户端，并确保应用程序已由Maven或Gradle构建（例如，通过使用mvn clean package），你就可以用类似以下的命令将Spring Boot应用程序部署到AWS。

```java
$ boxfuse run myapp-1.0.jar -env=prod
```
更多选项见[boxfuse运行文档](https://boxfuse.com/docs/commandline/run.html )。如果当前目录下有[boxfuse.conf](https://boxfuse.com/docs/commandline/#configuration )文件，就会考虑它。

**温馨提示**
>默认情况下，Boxfuse会在启动时激活一个名为boxfuse的Spring配置文件。如果你的可执行jar或war文件包含一个[application-boxfuse.properties](https://boxfuse.com/docs/payloads/springboot.html#configuration )文件，Boxfuse会根据它所包含的属性进行配置。

在这一点上，`boxfuse`为你的应用程序创建了一个图像，并将其上传，在AWS上配置和启动必要的资源，从而产生类似于以下例子的输出。

```java
Fusing Image for myapp-1.0.jar ...
Image fused in 00:06.838s (53937 K) -> axelfontaine/myapp:1.0
Creating axelfontaine/myapp ...
Pushing axelfontaine/myapp:1.0 ...
Verifying axelfontaine/myapp:1.0 ...
Creating Elastic IP ...
Mapping myapp-axelfontaine.boxfuse.io to 52.28.233.167 ...
Waiting for AWS to create an AMI for axelfontaine/myapp:1.0 in eu-central-1 (this may take up to 50 seconds) ...
AMI created in 00:23.557s -> ami-d23f38cf
Creating security group boxfuse-sg_axelfontaine/myapp:1.0 ...
Launching t2.micro instance of axelfontaine/myapp:1.0 (ami-d23f38cf) in eu-central-1 ...
Instance launched in 00:30.306s -> i-92ef9f53
Waiting for AWS to boot Instance i-92ef9f53 and Payload to start at https://52.28.235.61/ ...
Payload started in 00:29.266s -> https://52.28.235.61/
Remapping Elastic IP 52.28.233.167 to i-92ef9f53 ...
Waiting 15s for AWS to complete Elastic IP Zero Downtime transition ...
Deployment completed successfully. axelfontaine/myapp:1.0 is up and running at https://myapp-axelfontaine.boxfuse.io/
```
你的应用程序现在应该在AWS上启动和运行。
请参阅关于在[EC2上部署Spring Boot应用程序](https://boxfuse.com/blog/spring-boot-ec2.html )的博文，以及Boxfuse Spring Boot[集成的文档](https://boxfuse.com/docs/payloads/springboot.html )，开始使用Maven构建运行应用程序。

### 2.7. Azure


本[入门指南](https://spring.io/guides/gs/spring-boot-for-azure/ )将指导你将Spring Boot应用部署到[Azure Spring Cloud](https://azure.microsoft.com/en-us/services/spring-cloud/ )或[Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/overview )。

### 2.8. Google Cloud

谷歌云有几个选项，可以用来启动Spring Boot应用程序。最容易上手的可能是App Engine，但你也可以想办法用Container Engine在容器中运行Spring Boot，或者用Compute Engine在虚拟机上运行。

要在App Engine中运行，你可以先在UI中创建一个项目，它为你设置了一个独特的标识符，还设置了HTTP路由。在项目中添加一个Java应用，让它空着，然后使用[Google Cloud SDK](https://cloud.google.com/sdk/install )，从命令行或CI构建中把你的Spring Boot应用推送到该槽中。

App Engine Standard要求你使用WAR打包。按照这些[步骤](https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/appengine-java8/springboot-helloworld/README.md )，将App Engine Standard应用部署到Google Cloud。

另外，App Engine Flex要求你创建一个app.yaml文件来描述你的应用所需要的资源。通常，你把这个文件放在`src/main/appengine`中，它应该类似于以下文件。

```java
service: default

runtime: java
env: flex

runtime_config:
  jdk: openjdk8

handlers:
- url: /.*
  script: this field is required, but ignored

manual_scaling:
  instances: 1

health_check:
  enable_health_check: False

env_variables:
  ENCRYPT_KEY: your_encryption_key_here

```

你可以通过在构建配置中添加项目ID来部署该应用（例如，使用Maven插件），如下例所示。

```java
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>appengine-maven-plugin</artifactId>
    <version>1.3.0</version>
    <configuration>
        <project>myproject</project>
    </configuration>
</plugin>
```

然后用`mvn appengine:deploy`进行部署（如果你需要先进行身份验证，则构建失败）。

## 3. Installing Spring Boot 程序

除了通过使用java -jar运行Spring Boot应用程序外，还可以为Unix系统制作完全可执行的应用程序。完全可执行的jar可以像其他可执行的二进制文件一样执行，也可以用init.d或systemd注册。这有助于在普通生产环境中安装和管理Spring Boot应用程序。

**慎重1** 
>完全可执行的罐子通过在文件的前面嵌入一个额外的脚本来工作。目前，有些工具不接受这种格式，所以你不一定能使用这种技术。例如，jar -xf可能会默默地无法提取一个已经被做成完全可执行的jar或war文件。建议你只有在打算直接执行你的jar或war，而不是用java -jar运行它或将它部署到servlet容器中时，才使它完全可执行。

**慎重2**
>一个zip64格式的jar文件不能被完全执行。试图这样做将导致一个jar文件在直接执行或用java -jar执行时被报告为损坏。一个标准格式的jar文件包含一个或多个zip64格式的嵌套jar，可以完全执行。
>

要用Maven创建一个 "完全可执行 "的jar，请使用以下插件配置
```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

下面的例子显示了相当于Gradle的配置。

```java
bootJar {
    launchScript()
}
```

然后你可以通过输入`./my-application.jar`（其中my-application是你的工件的名称）来运行你的应用程序。包含jar的目录将作为你的应用程序的工作目录。


### 3.1 支持的操作系统

默认脚本支持大多数Linux发行版，并在CentOS和Ubuntu上测试。其他平台，如OS X和FreeBSD，需要使用一个自定义的嵌入式LaunchScript。

### 3.2. Unix/Linux服务

通过使用init.d或systemd，Spring Boot应用程序可以轻松地作为Unix/Linux服务启动。

#### 3.2.1. 作为init.d服务安装(System V)

如果你将Spring Boot的Maven或Gradle插件配置为生成[完全可执行的jar](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.installing )，并且不使用自定义的embeddedLaunchScript，你的应用程序就可以作为init.d服务使用。为此，将jar链接到init.d，以支持标准的启动、停止、重启和状态命令。

该脚本支持以下功能。
- Starts the services as the user that owns the jar file
- Tracks the application’s PID by using /var/run/<appname>/<appname>.pid
- Writes console logs to /var/log/<appname>.log
  
假设你在/var/myapp中安装了一个Spring Boot应用，要把Spring Boot应用安装成init.d服务，请创建一个符号链接，如下所示。

```java
$ sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```

一旦安装完毕，你可以用通常的方式启动和停止该服务。例如，在基于Debian的系统上，你可以用以下命令启动它。

```java
$ service myapp start
```

**温馨提示**
>如果你的应用程序无法启动，请检查写入/var/log/<appname>.log的日志文件是否有错误。

你也可以通过使用你的标准操作系统工具来标记该应用程序自动启动。例如，在Debian上，你可以使用以下命令。

```java
$ update-rc.d myapp defaults <priority>
```

#####  确保init.d服务的安全

**笔记**
>以下是一套关于如何保护作为init.d服务运行的Spring Boot应用程序的指南。它并不打算成为一份详尽的清单，列出为加固应用程序及其运行环境所应做的一切。

当以root身份执行时，如root被用来启动init.d服务时，默认的可执行脚本会以RUN_AS_USER环境变量中指定的用户身份运行应用程序。当环境变量没有设置时，会使用拥有jar文件的用户来代替。你不应该以root身份运行Spring Boot应用程序，所以RUN_AS_USER不应该是root，你的应用程序的jar文件不应该由root拥有。相反，创建一个特定的用户来运行你的应用程序，并设置RUN_AS_USER环境变量或使用chown来使其成为jar文件的所有者，如下例所示。
```java
$ chown bootapp:bootapp your-app.jar
```

在这种情况下，默认的可执行脚本以bootapp用户的身份运行应用程序。

**温馨提示**
>为了减少应用程序的用户账户被入侵的机会，你应该考虑防止它使用登录的外壳。例如，你可以将该账户的shell设置为/usr/sbin/nologin。

你还应该采取措施，防止你的应用程序的jar文件被修改。首先，配置其权限，使其不能被写入，只能由其所有者读取或执行，如下面的例子所示。
```java
$ chmod 500 your-app.jar
```

其次，如果你的应用程序或运行它的账户被入侵，你也应该采取措施限制损失。如果攻击者确实获得了访问权，他们可以使jar文件可写并改变其内容。防止这种情况的一个方法是通过使用chattr使其不可更改，如下面的例子中所示。
```java
$ sudo chattr +i your-app.jar
```

这将防止任何用户，包括root，修改jar。

如果root被用来控制应用程序的服务，并且你使用一个.conf文件来定制它的启动，那么.conf文件就会被root用户读取和评估。它应该被相应地保护起来。使用chmod使该文件只能由所有者读取，并使用chown使root成为所有者，如下例所示。
```java
$ chmod 400 your-app.conf
$ sudo chown root:root your-app.conf
```

#### 3.2.2. 以systemd服务的形式安装
systemd 是 System V init 系统的后继者，现在很多现代 Linux 发行版都在使用。虽然你可以继续使用init.d脚本与systemd，但也可以通过使用systemd的 "服务 "脚本来启动Spring Boot应用程序。

假设你在/var/myapp中安装了一个Spring Boot应用，要把Spring Boot应用安装成systemd服务，需要创建一个名为myapp.service的脚本，并将其放在/etc/systemd/system目录下。下面的脚本提供了一个例子。
```java
[Unit]
Description=myapp
After=syslog.target

[Service]
User=myapp
ExecStart=/var/myapp/myapp.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

**重要**
>记住要为你的应用程序改变描述、用户和ExecStart字段。

**笔记**
> ExecStart字段没有声明脚本动作命令，这意味着默认使用的是运行命令。

注意，与作为 init.d 服务运行时不同，运行应用程序的用户、PID 文件和控制台日志文件由 systemd 自己管理，因此必须在 "服务 "脚本中使用适当的字段进行配置。更多细节请参考[服务单元配置手册](https://www.freedesktop.org/software/systemd/man/systemd.service.html )。

要想标记应用程序在系统启动时自动启动，请使用以下命令。
```java
$ systemctl enable myapp.service
```

更多细节请参考man systemctl。

#### 3.2.3. 定制启动脚本

由Maven或Gradle插件编写的默认嵌入式启动脚本可以通过多种方式定制。对于大多数人来说，使用默认脚本和一些定制的脚本通常就足够了。如果你发现你无法定制你需要的东西，可以使用embeddedLaunchScript选项，完全编写你自己的文件。

##### 编写开始脚本时的自定义

在启动脚本写进jar文件时，定制其元素往往是有意义的。例如，init.d脚本可以提供一个 "描述"。既然你预先知道了描述（而且它不需要改变），你不妨在生成jar时提供它。

要定制书面元素，可使用Spring Boot Maven插件的embeddedLaunchScriptProperties选项或Spring Boot Gradle插件的 launchScript的[属性](https://docs.spring.io/spring-boot/docs/2.5.3/gradle-plugin/reference/htmlsingle/#packaging-executable.configuring.launch-script )。

默认脚本支持以下属性替换。


| Name | 描述 | Gradle 默认 | Maven 默认 |
| --- | --- | --- | --- |
| `mode` | 脚本模式. | `auto` | `auto` |
| `initInfoProvides` | The `Provides` section of “INIT INFO” | `${task.baseName}` | `${project.artifactId}` |
| `initInfoRequiredStart` | `Required-Start` section of “INIT INFO”. | `$remote_fs $syslog $network` | `$remote_fs $syslog $network` |
| `initInfoRequiredStop` | `Required-Stop` section of “INIT INFO”. | `$remote_fs $syslog $network` | `$remote_fs $syslog $network` |
| `initInfoDefaultStart` | `Default-Start` section of “INIT INFO”. | `2 3 4 5` | `2 3 4 5` |
| `initInfoDefaultStop` | `Default-Stop` section of “INIT INFO”. | `0 1 6` | `0 1 6` |
| `initInfoShortDescription` | `Short-Description` section of “INIT INFO”. | `{project.description}` 的单行版本 \(回退到 `{task.baseName}`\) | `{project.name}` |
| `initInfoDescription` | `Description` section of “INIT INFO”. | `${project.description}` \(falling back to `${task.baseName}`\) | `${project.description}` \(falling back to `${project.name}`\) |
| `initInfoChkconfig` | `chkconfig` section of “INIT INFO” | `2345 99 01` | `2345 99 01` |
| `confFolder` | 默认值为 `CONF_FOLDER` | 包含 jar 的文件夹 | 包含 jar 的文件夹 |
| `inlinedConfScript` | 对应内联在默认启动脚本中的文件脚本的引用。这可用于设置环境变量，例如在加载任何外部配置文件之前的`JAVA_OPTS` |  |  |
| `logFolder` | 默认值 `LOG_FOLDER`. 仅对 `init.d` 服务有效 |  |  |
| `logFilename` | 默认值 `LOG_FILENAME`.仅对 `init.d` 服务有效 |  |  |
| `pidFolder` | 默认值 `PID_FOLDER`. 仅对 `init.d` 服务有效 |  |  |
| `pidFilename` | PID 文件名的默认值 `PID_FOLDER`. 仅对 `init.d` 服务有效 |  |  |
| `useStartStopDaemon` |是否`start-stop-daemon`命令，当它可用时，应该用于控制进程 | `true` | `true` |
| `stopWaitTime` |默认值 `STOP_WAIT_TIME` in seconds. 仅对 `init.d` 服务有效 | 60 | 60 |

##### 脚本运行时的自定义
对于脚本中需要在编写jar后进行定制的项目，你可以使用环境变量或配置文件。

默认脚本支持以下环境属性。


| 变量 | 描述 |
| --- | --- |
| `MODE` | 操作的 "模式"。默认情况取决于jar的构建方式，但通常是 "自动"（意思是它试图通过检查是否是一个名为 "init.d "目录下的符号链接来猜测是否是一个启动脚本）。你可以明确地把它设置为`service`，这样`stop|start|status|restart`命令就会起作用，如果你想在前台运行脚本，可以设置为`run`。|
| `RUN_AS_USER` | 将用于运行应用程序的用户。如果不设置，将使用拥有jar文件的用户。 |
| `USE_START_STOP_DAEMON` |当 "start-stop-daemon "命令可用时，是否应使用它来控制进程。默认为 "true"。|
| `PID_FOLDER` | pid文件夹的根名（默认为`/var/run`）。 |
| `LOG_FOLDER` | 放置日志文件的文件夹名称(默认为`/var/log`)。|
| `CONF_FOLDER` |读取.conf文件的文件夹名称（默认与jar-file相同）。 |
| `LOG_FILENAME` | `LOG_FOLDER`中的日志文件名称（默认为`<appname>.log`）。 |
| `APP_NAME` | 应用程序的名称。如果jar是从一个符号链接运行的，脚本会猜测应用程序的名称。如果它不是一个符号链接，或者你想明确地设置应用程序的名称，这可能是有用的。 |
| `RUN_ARGS` | 传递给程序的参数(Spring Boot应用程序)。|
| `JAVA_HOME` | 默认情况下，通过使用`PATH`发现`java`可执行文件的位置，但如果在`$JAVA_HOME/bin/java`有一个可执行文件，你可以明确设置它。 |
| `JAVA_OPTS` |  启动JVM时传递给它的选项。 |
| `JARFILE` |jar文件的明确位置，以防脚本被用来启动一个实际上没有嵌入的jar文件。 |
| `DEBUG` |  如果不是空的，在shell进程中设置`-x`标志，允许你看到脚本中的逻辑。 |
| `STOP_WAIT_TIME` |当停止应用程序时，在强制关机前要等待的时间（秒）（默认为`60`）。|

**笔记**
>PID_FOLDER、LOG_FOLDER 和 LOG_FILENAME 变量只对 init.d 服务有效。对于systemd来说，相应的自定义变量可以通过 "service "脚本来实现。更多细节请参见服务[单元配置手册](https://www.freedesktop.org/software/systemd/man/systemd.service.html )。

除了JARFILE和APP_NAME之外，上一节中列出的设置都可以通过使用.conf文件来配置。该文件应该在jar文件的旁边，具有相同的名称，但后缀为.conf而不是.jar。例如，一个名为/var/myapp/myapp.jar的jar文件使用名为/var/myapp/myapp.conf的配置文件，如以下例子所示。

myapp.conf
```java
JAVA_OPTS=-Xmx1024M
LOG_FOLDER=/custom/log/folder
```

**温馨提示**
>如果你不喜欢把配置文件放在jar文件旁边，你可以设置一个CONF_FOLDER环境变量来定制配置文件的位置。

要了解如何适当地保护这个文件，请参阅[保护init.d服务的指南](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.installing.nix-services.init-d.securing )。

### 3.3 Microsoft Windows服务

Spring Boot应用程序可以通过使用winsw作为Windows服务启动。

一个([单独维护的示例](https://github.com/snicoll/spring-boot-daemon ))详细描述了如何为Spring Boot应用程序创建Windows服务。


## 4. 接下来读什么

查看Cloud Foundry、Heroku、OpenShift和Boxfuse网站，了解更多关于PaaS可以提供的功能类型的信息。这些只是最流行的Java PaaS提供程序中的四个。由于Spring Boot非常适合基于云的部署，您也可以自由地考虑其他提供商。
下一节继续讨论`Spring Boot CLI`，或者您可以跳过这一节阅读构建工具插件。

{{#include ../license.md}}