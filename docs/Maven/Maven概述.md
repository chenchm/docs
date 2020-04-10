## 概述

本文将以举例的方式对`Maven`常用部分进行说明。

本文的主要是笔者对`Maven`官方手册的翻译和理解：[http://maven.apache.org/guides/getting-started/index.html](http://maven.apache.org/guides/getting-started/index.html)

参考了W3Cschool的`Maven`教程：[https://www.w3cschool.cn/maven/u7oe1ht0.html](https://www.w3cschool.cn/maven/u7oe1ht0.html)

## 如何创建一个Maven项目

现在一般都是使用`IDE`以更人性化的方式创建`Maven`项目，这里我们使用`Maven`的**原型（archetype）**插件来创建项目，在命令行以下命令：

```shell
mvn archetype:generate
-DgroupId=com.mycompany.app \ 
-DartifactId=my-app \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DinteractiveMode=false
```

执行该命令后，在执行该命令的文件夹下会生成一个名为`my-app`的项目，并且`my-app`文件夹下包含了一个`pom.xml`文件，该文件的内容类似于：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

`pom.xml`包含了此项目的项目对象模型（`Project Object Model`），`POM`是`Maven`的基本工作单元，简而言之就是`pom.xml`中包含了你项目所有重要的信息，能够查找项目的一切相关内容。关于`POM`的介绍，可以查看[`POM`简介](../Maven POM)。

下面我们将逐一介绍这个简单的`POM`几个关键元素：

- **project**  这是所有`Maven pom.xml`文件中的根元素
- **modelVersion** 此元素表示`POM`使用的对象模型版本，对象模型版本很少发生变化。
- **groupId** 此元素表示创建项目的组织的唯一标识符，`groupId`是项目的关键标识符之一，通常基于组织的完全限定域名
- **artifactId** 此元素表示该项目生成的工件唯一基本名称，项目生成的工件通常是`JAR`文件。由`Maven`生成的典型工件具有`<artifactId>-<version>.<extension>`形式（例如，`myapp-1.0.jar`）。
- **packaging**  此元素表示工件的打包类型（例如`JAR`,`WAR`,`EAR`等）。这不仅意味着生成的工时`JAR`,`WAR`或是`EAR`，而且还将表明特定的生命周期将会被绑定构建过程中。（详见[Maven的构建生命周期](../Maven构建生命周期)）
- **version** 此元素表示项目生成的工件的版本。
- **name** 此元素表明项目的显示名称，常用于`Maven`文档的生成。
- **url** 此元素表明项目的站点网址，常用于`Maven`文档的生成。
- **description** 此元素提供项目的基本描述，常用于`Maven`文档的生成。

有关`POM`中能够使用的元素更加完整的参考，请参阅官方的 [POM Reference](http://maven.apache.org/ref/current/maven-model/maven.html)。

使用`archetype `插件生成项目后，你可以看到如下的目录结构被创建：

```shell
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

如你所见，除了`pom.xml`文件外，还生成了应用程序资源和测试资源两颗资源树。这是`Maven`项目的标准布局，应用程序资源位于`${basedir}/src/main/java`而测试资源位于`${basedir}/src/test/java`（`${basedir}`代表包含`pom.xml`文件的目录），这也是`Maven`推荐的布局格式或者说是`Maven`项目的约定，更多关于`Maven`的文件布局请参阅 [Introduction to the Standard Directory Layout](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)。

## 如何编译源码

在`pom.xml`文件所在的路径下执行以下命令（执行`Maven`命令需要从`pom.xml`中获取配置信息）：

```shell
mvn compile
```

通过执行上面的命令，你可以在控制台看到类似的输出：

```shell
[INFO] -------------------------------------------------------------------
[INFO] Building Maven Quick Start Archetype
[INFO]    task-segment: [compile]
[INFO] -------------------------------------------------------------------
[INFO] artifact org.apache.maven.plugins:maven-resources-plugin: \
  checking for updates from central
...
[INFO] artifact org.apache.maven.plugins:maven-compiler-plugin: \
  checking for updates from central
...
[INFO] [resources:resources]
...
[INFO] [compiler:compile]
Compiling 1 source file to <dir>/my-app/target/classes
[INFO] -------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] -------------------------------------------------------------------
[INFO] Total time: 3 minutes 54 seconds
[INFO] Finished at: Fri Sep 23 15:48:34 GMT-05:00 2005
[INFO] Final Memory: 2M/6M
[INFO] -------------------------------------------------------------------
```

当第一次执行命令时（任何`Maven`命令），`Maven`需要从仓库下载完成该命令所需要的插件和依赖。这可能会花费一些时间，但之后如果再执行相同的命令就很快了。

从输出中可以看到，编译生成的文件被放到了`${basedir}/target/classes`目录下，这也是`Maven`的一个标准约定。

## 如何编译测试源码并且运行单元测试

只要执行以下一条命令即可完成：

```shell
mvn test
```

通过执行上面的命令，你可以在控制台看到类似的输出：

```shell
[INFO] -------------------------------------------------------------------
[INFO] Building Maven Quick Start Archetype
[INFO]    task-segment: [test]
[INFO] -------------------------------------------------------------------
[INFO] artifact org.apache.maven.plugins:maven-surefire-plugin: \
  checking for updates from central
...
[INFO] [resources:resources]
[INFO] [compiler:compile]
[INFO] Nothing to compile - all classes are up to date
[INFO] [resources:testResources]
[INFO] [compiler:testCompile]
Compiling 1 source file to C:\Test\Maven2\test\my-app\target\test-classes
...
[INFO] [surefire:test]
[INFO] Setting reports dir: C:\Test\Maven2\test\my-app\target/surefire-reports
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
[surefire] Running com.mycompany.app.AppTest
[surefire] Tests run: 1, Failures: 0, Errors: 0, Time elapsed: 0 sec
Results :
[surefire] Tests run: 1, Failures: 0, Errors: 0
[INFO] -------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] -------------------------------------------------------------------
[INFO] Total time: 15 seconds
[INFO] Finished at: Thu Oct 06 08:12:17 MDT 2005
[INFO] Final Memory: 2M/8M
[INFO] -------------------------------------------------------------------
```

需要额外的注意的是`surefire`插件（它会运行测试用例）会寻找`${basedir}/src/test/java`下具有特定命名规则的测试文件执行测试，例如：

- `**/*Test.java`
- `**/Test*.java`
- `**/*TestCase.java`

但会排除具有以下命名规则的文件：

- `**/Abstract*Test.java`
- `**/Abstract*TestCase.java`

从上面的输出可以看出，构建过程执行了源码编译、测试源码编译并且运行了单元测试，你可能会觉得奇怪为什么`test`命令能执行这么多操作，这是因为`test`会执行`test`阶段之前所有的阶段，详见[`Maven`的构建生命周期](../Maven构建生命周期)。

如果你只想编译测试源码但不运行单元测试，你可以执行以下命令：

```shell
mvn test-compile
```

## 如何生成一个JAR包并安装到本地仓库

要生成`JAR`文件，首先你需要在`POM`中指定`<packaging>`值为`jar`（或者不进行指定，默认值为`jar`），这将让`Maven`知道要生成一个`jar`文件，然后执行以下命令：

```shell
mvn package
```

当命令执行完成后，你可以在`${basedir}/target`目录下看到生成的`JAR`文件。

如果你想安装你的生成的工件（这里是`JAR`文件）到你的本地仓库（`${user.home}/.m2/repository`是默认的仓库位置）。你可以执行下面的命令：

```shell
mvn install
```

通过执行上面的命令，你可以在控制台看到类似的输出：

```shell
[INFO] -------------------------------------------------------------------
[INFO] Building Maven Quick Start Archetype
[INFO]    task-segment: [install]
[INFO] -------------------------------------------------------------------
[INFO] [resources:resources]
[INFO] [compiler:compile]
Compiling 1 source file to <dir>/my-app/target/classes
[INFO] [resources:testResources]
[INFO] [compiler:testCompile]
Compiling 1 source file to <dir>/my-app/target/test-classes
[INFO] [surefire:test]
[INFO] Setting reports dir: <dir>/my-app/target/surefire-reports
 
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
[surefire] Running com.mycompany.app.AppTest
[surefire] Tests run: 1, Failures: 0, Errors: 0, Time elapsed: 0.001 sec
Results :
[surefire] Tests run: 1, Failures: 0, Errors: 0
[INFO] [jar:jar]
[INFO] Building jar: <dir>/my-app/target/my-app-1.0-SNAPSHOT.jar
[INFO] [install:install]
[INFO] Installing <dir>/my-app/target/my-app-1.0-SNAPSHOT.jar to \
   <local-repository>/com/mycompany/app/my-app/1.0-SNAPSHOT/my-app-1.0-SNAPSHOT.jar
[INFO] -------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] -------------------------------------------------------------------
[INFO] Total time: 5 seconds
[INFO] Finished at: Tue Oct 04 13:20:32 GMT-05:00 2005
[INFO] Final Memory: 3M/8M
[INFO] -------------------------------------------------------------------
```

## 如何使用插件

当你想要自定义`Maven`项目的构建时，你可以通过添加或重新配置插件来实现。下面的例子我们通过配置使`Java`编译器允许`JDK 5.0`的源，你可以简单地添加以下内容：

```xml
...
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.3</version>
      <configuration>
        <source>1.5</source>
        <target>1.5</target>
      </configuration>
    </plugin>
  </plugins>
</build>
...
```

`Maven`会自动下载和使用你指定版本的插件（如果你没有指定插件版本，则默认使用最新版本）。

在上面的例子中，`configuration`元素会被应用到`compiler`插件的所有目标（goal）上（注：你也可以在`<plugin>`的子元素`<execution>`中将`configuration`应用的某一个目标上），实际上`compiler`插件已经在构建过程中使用了，上面的例子只是修改了部分配置信息。你也可以为构建过程添加新的插件目标，更多信息可以查看[`Maven`的构建生命周期](../Maven构建生命周期)。

你可以在`Maven`官方文档中查看更多可用的插件，详见[Plugins List](http://maven.apache.org/plugins/) 。官方文档同时也对`Maven`插件的配置和进行了比较详细的介绍，详见 [Guide to Configuring Plugins](http://maven.apache.org/guides/mini/guide-configuring-plugins.html)。

## 如何往JAR文件中添加资源文件

在`JAR`包中添加资源文件是一个常见的需求，对于这个需求`Maven`仍然依赖于的他的标准文件布局（[Standard Directory Layout](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)），这意味着只要遵守`Maven`的标准约定，你就可以通过简单地将这些资源文件放在标准的目录结构下实现在`JAR`包中封装资源。

在下面的例子中，我们添加了`${basedir}/src/main/resources`目录并把想要被打包进`JAR`包的资源都放在该目录下。`Maven`采用的规则是：放在`${basedir}/src/main/resources`目录下的任何文件或者文件夹都会被打包进行`JAR`包中，并且这些资源的文件结构将在`JAR`包内与原来一致。

```shell
my-app
|-- pom.xml
`-- src
    |-- main
    |   |-- java
    |   |   `-- com
    |   |       `-- mycompany
    |   |           `-- app
    |   |               `-- App.java
    |   `-- resources
    |       `-- META-INF
    |           `-- application.properties
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

如你所见，我们在`resource`文件下创建了一个`META-INF`文件夹并新建了一个`application.properties`文件。如果你解压了由上面的项目生成的`JAR`包，你可以看到如下结构：

```shell
|-- META-INF
|   |-- MANIFEST.MF
|   |-- application.properties
|   `-- maven
|       `-- com.mycompany.app
|           `-- my-app
|               |-- pom.properties
|               `-- pom.xml
`-- com
    `-- mycompany
        `-- app
            `-- App.class
```

可以看到在`resources`文件中添加的内容都以相同的结构出现在`JAR`包中了，但你可能注意到了几个额外的文件（`MANIFEST.MF, pom.xml, pom.properties`）和目录，其实这些都是`Maven`生成`JAR`包的标准产出物（注：其实`META-INF`就是标准产出的一部分，只是由于在`resources`中取了相同的名字所以`application.properties`会出现在这个位置），这些文件存在目的是让`Maven`生成的工件能够实现自述（`self-describing`）。

## 如何替换资源文件中的引用值

有时资源文件需要包含一些只能在构建阶段才提供的值，要在`Maven`中完成此操作，可以使用语法`$ {<property name>}`将包含值的属性引用到资源文件中。该属性可以是`pom.xml`文件中定义的一个值，在某用户的`setting.xml`文件中定义的值，一个外部的properties 文件中的属性或者是系统属性。

为了在资源准备阶段（`process-resources phase`）将语法`$ {<property name>}`替换（`filter`）为特定的值，只要简单地将`pom.xml`文件中的`<filtering>`设置为`true`即可：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
 
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
</project>
```

如上所示，我们添加了`<build>、<resources>`以及`<resource>`元素，而且还明确声明了`src/main/resources`为资源文件所在的位置。这些信息在未声明前都是以默认值的形式提供，但由于`<filtering>`的默认值为`false`，所以我们必须显示地声明它并且设置为`true`。

下面让我们在`application.properties`文件中添加两个属性，它们会在构建阶段被替换为特定的值：

```properties
# application.properties
application.name=${project.name}
application.version=${project.version}
```

其中`${project.name}`和`${project.version}`为`Maven`的项目变量，分别代表当前项目的名字和版本，详见[项目变量以及插值](../Maven POM#_4)部分。之后，你可执行下面的命令（`process-resources`是构建生命周期的一个阶段，执行资源文件的复制和替换）：

```shell
mvn process-resources
```

然后，你会在`target/classes`目录下看到替换之后的`application.properties`如下：

```properties
# application.properties
application.name=Maven Quick Start Archetype
application.version=1.0-SNAPSHOT
```

如果你要引用外部`properties`文件（指的应该是非系统属性和`pom.xml`中定义的属性，即使`properties`文件在`resources`目录下）的某一个属性，你需要在`pom.xml`中添加对这个外部文件的引用。例如，我们创建了一个外部`properties`文件，它的完整路径为`src/main/filters/filter.properties`，内容如下：

```properties
# filter.properties
my.filter.value=hello!
```

现在，我们需要在`pom.xml`中添加新的引用：

```xml
...
  <build>
    <filters>
      <filter>src/main/filters/filter.properties</filter>
    </filters>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
...
```

接着，就可以在`application.properties`文件中引用这个属性了：

```properties
# application.properties
application.name=${project.name}
application.version=${project.version}
message=${my.filter.value}
```

在外部文件中定义my.filter.value属性的替代方案是在`pom.xml`的`<properties>`元素中定义它并获得相同的效果（注意，这里没有添加`filters`元素）：

```xml
...
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
 
  <properties>
    <my.filter.value>hello</my.filter.value>
  </properties>
...
```

替换值也可以从系统属性中获取，包括`Java`内置的系统属性（如`java.version, user.home`）或者使用`Java -D`参数在命令行上定义的属性。

例如，`application.properties`的内容如下：

```properties
# application.properties
java.version=${java.version}
command.line.prop=${command.line.prop}
```

我们将在命令行上定义`command.line.prop`属性的值，如下：

```shell
mvn process-resources "-Dcommand.line.prop=hello again"
```

## 如何使用外部依赖

外部依赖即`POM`中的`<dependencies>`部分定义的内容（例如之前例子中的`junit jar`包），接下来我们将了解它是如何工作的。更加全面的介绍，请参考官方文档对依赖机制的介绍（ [Introduction to Dependency Mechanism](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)）。

让我们以之前的项目为例子：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

对于外部依赖，你需要至少定义4个属性：`groupId, artifactId, version`和`scope`。其中`groupId, artifactId, version`就是你需要引用的项目的`pom.xml`文件中定义的，而`scope`表示你要在哪个范围内使用这个依赖，下表为`scope`可用的值：

|   Scope    |                             描述                             |
| :--------: | :----------------------------------------------------------: |
| `compile`  | `compile`为`scope`的默认值，如果未指定`scope`则`scope`的值为`compile`。此范围的依赖项（`dependencie`）在项目的所有类路径（`classpath`）下都可以使用，并且如果其他项目依赖该项目，依赖了该项目的项目也可以使用这个依赖项。 |
| `provided` | 类似于`compile`，但`provided`表示依赖项在运行时由`JDK`或容器提供。例如，在构建`JaveEE WEB`项目时，你应该把`Servlet API`和`Java EE APIs`依赖项的范围设置为`provided,`因为`WEB`容器的`classpaths`会提供这些类。此范围的依赖项只能在编译和测试类路径中使用，且不具有传递性。 |
| `runtime`  | 此范围表明，在运行时需要该依赖项但编译时不需要。它位于运行时和测试类路径下。 |
|   `test`   | 此范围表明，该依赖项只在测试部分的编译和运行阶段使用，并且不具有传递性。 |
|  `system`  |   类似于`provided`，但必须提供明确包含了依赖项的`jar`包。    |

通过这些依赖信息，`Maven`能够在构建项目时通过这些信息找到这些依赖项。`Maven`会在本地仓库（默认位置为`${user.home}/.m2/repository`）查找依赖，在[如何生成一个JAR包并安装到本地仓库](#jar)中我将`my-app-1.0-SNAPSHOT.jar`安装到了本地仓库中，所以我们可以在其他项目的`pom.xml`文件中添加依赖信息来引用这个`jar`包：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-other-app</artifactId>
  ...
  <dependencies>
    ...
    <dependency>
      <groupId>com.mycompany.app</groupId>
      <artifactId>my-app</artifactId>
      <version>1.0-SNAPSHOT</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
</project>
```

如果项目引用了本地仓库中不存在的依赖项，`Maven`会将该依赖从远程仓库下载到本地仓库中。`Maven`默认使用的远程仓库为`http://repo.maven.apache.org/maven2/`，你也可以配置自己的远程仓库（国内可以使用阿里的远程仓库`http://maven.aliyun.com/nexus/content/groups/public`）来替代或做为默认仓库的补充。更多关于仓库的的资料可以参考官方文档[Introduction to Repositories](http://maven.apache.org/guides/introduction/introduction-to-repositories.html)。