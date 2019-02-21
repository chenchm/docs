## 概述

本文主要是对`Maven`的构建生命周期进行简单的介绍。

**索引**

[TOC]

本文的主要是笔者对`Maven`官方手册的翻译和理解：[http://maven.apache.org/guides/introduction/introduction-to-the-pom.html](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)

参考了W3Cschool的`Maven`教程：[https://www.w3cschool.cn/maven/tkhr1ht6.html](https://www.w3cschool.cn/maven/tkhr1ht6.html)

## Maven的构建生命周期

Maven的构建生命周期一共有三个：分别是`default(build)`、`site`和`clean`。

|    生命周期    |              描述              |
| :------------: | :----------------------------: |
| default(build) |        用于处理项目部署        |
|      site      |      用于创建项目站点文档      |
|     clean      | 用于清理其他生命周期生成的文件 |

### Maven构建生命周期是由多个构建阶段组成的

以default构建生命周期为例，它由以下几个构建阶段组成

- `validate` - 检查工程配置是否正确，完成构建过程的所有必要信息是否能够获取到
- `compile` - 编译工程源码
- `test` - 使用适当的单元测试框架（例如JUnit）运行测试
- `package` - 获取编译后的代码，并按照可发布的格式进行打包，例如 `JAR`、`WAR` 或者 `EAR` 文件
- `verify` - 运行检查操作来验证工程包是有效的，并满足质量要求
- `install` - 安装工程包到本地仓库中，该仓库可以作为本地其他工程的依赖
- `deploy` - 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程

这些生命周期阶段（以及此处未显示的其他生命周期阶段）将按顺序执行，以完成默认生命周期。 鉴于上面的生命周期阶段，这意味着当使用默认生命周期时，`Maven`将首先验证项目，然后将尝试编译源代码，针对测试运行，打包二进制文件（例如`jar`），对其运行集成测试包，验证集成测试，将经过验证的软件包安装到本地存储库，然后将已安装的软件包部署到远程存储库。

### 构建阶段由多个插件目标组成

插件目标（goal）表示特定任务（比构建阶段更精细），这有助于构建和管理项目。 它可能绑定到零个或多个构建阶段。没有绑定任何构建阶段的目标可以在构建生命周期之外被直接调用执行。执行顺序取决于调用目标和构建阶段的顺序。例如，下面的命令中`clean`和`package`参数是构建阶段，而`dependency:copy-dependencies`是（插件的）目标。

```shell
mvn clean dependency:copy-dependencies package
```

这里的 `clean` 阶段将会被首先执行，然后 `dependency:copy-dependencies `目标会被执行，最终` package `阶段被执行。

### 生命周期参考

有一些与 Maven 生命周期相关的重要概念需要说明：当一个阶段通过 `Maven` 命令调用时，例如 *mvn package*，*该阶段之前以及包括该阶段在内的所有阶段会被执行*。不同的 maven 目标将根据`package`的类型（`JAR / WAR / EAR/ POM`），被绑定到不同的 Maven 生命周期阶段。

#### Clean生命周期

| 生命周期阶段 |               描述               |
| :----------: | :------------------------------: |
|  pre-clean   | 在实际项目清理之前执行所需的过程 |
|    clean     | 删除之前构建过程中生成的所有文件 |
|  post-clean  |    执行完成项目清理所需的过程    |

#### Default生命周期

|     生命周期阶段      |                             描述                             |
| :-------------------: | :----------------------------------------------------------: |
|       validate        | 检查工程配置是否正确，完成构建过程的所有必要信息是否能够获取到。 |
|      initialize       |                初始化构建状态，例如设置属性。                |
|   generate-sources    |             生成编译阶段需要包含的任何源码文件。             |
|    process-sources    |      处理源代码，例如，过滤任何值（filter any value）。      |
|  generate-resources   |               生成工程包中需要包含的资源文件。               |
|   process-resources   |      拷贝和处理资源文件到目的目录中，为打包阶段做准备。      |
|        compile        |                        编译工程源码。                        |
|    process-classes    |   处理编译生成的文件，例如 Java Class 字节码的加强和优化。   |
| generate-test-sources |            生成编译阶段需要包含的任何测试源代码。            |
| process-test-sources  |    处理测试源代码，例如，过滤任何值（filter any values)。    |
|     test-compile      |                编译测试源代码到测试目的目录。                |
| process-test-classes  |              处理测试代码文件编译后生成的文件。              |
|         test          |        使用适当的单元测试框架（例如JUnit）运行测试。         |
|    prepare-package    |        在真正打包之前，为准备打包执行任何必要的操作。        |
|        package        | 获取编译后的代码，并按照可发布的格式进行打包，例如 JAR、WAR 或者 EAR 文件。 |
| pre-integration-test  | 在集成测试执行之前，执行所需的操作。例如，设置所需的环境变量。 |
|   integration-test    |      处理和部署必须的工程包到集成测试能够运行的环境中。      |
| post-integration-test |      在集成测试被执行后执行必要的操作。例如，清理环境。      |
|        verify         |      运行检查操作来验证工程包是有效的，并满足质量要求。      |
|        install        |  安装工程包到本地仓库中，该仓库可以作为本地其他工程的依赖。  |
|        deploy         |  拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程。  |

#### Site生命周期

| 生命周期阶段 |                    描述                    |
| :----------: | :----------------------------------------: |
|   pre-site   |    在实际项目站点生成之前执行所需的过程    |
|     site     |             生成项目的站点文档             |
|  post-site   | 执行完成站点生成所需的流程，并准备站点部署 |
| site-deploy  |   将生成的站点文档部署到指定的Web服务器    |

### 通过配置项目来使用构建生命周期

#### 1.Packaging

最常见的方法是通过指定`pom`元素`<packaging>`的类型来绑定各构建阶段及其对应的插件目标。可用的打包类型主要有`jar`, `war`, `ear` 和`pom`，如果未在`pom.xml`文件中指定打包类型，则会被设置为默认值`jar`。

每种打包类型包含一系列被绑定到特定构建阶段的插件目标。例如，`jar`类型将绑定如下目标以构建默认生命周期的阶段。

|          Phase           |        plugin:goal        |
| :----------------------: | :-----------------------: |
|   `process-resources`    |   `resources:resources`   |
|        `compile`         |    `compiler:compile`     |
| `process-test-resources` | `resources:testResources` |
|      `test-compile`      |  `compiler:testCompile`   |
|          `test`          |      `surefire:test`      |
|        `package`         |         `jar:jar`         |
|        `install`         |     `install:install`     |
|         `deploy`         |      `deploy:deploy`      |

大部分打包类型的绑定集相差不大，但是有一些打包类型却比较特殊。例如，当打包类型为`pom`的项目仅将目标绑定到install和deploy阶段（有关某些打包类型的目标到构建阶段绑定的完整列表，请参阅[standard set of bindings](http://maven.apache.org/ref/current/maven-core/default-bindings.html)）。

#### 2.Plugins

另一种添加目标到指定阶段的方式是在项目中配置插件。插件具有一个或多个目标，其中每个目标表示该插件的能力。 例如，`Compiler`插件有两个目标：`compile`和`testCompile`。前者编译主代码的源代码，后者编译测试代码的源代码。

插件可以包含将目标绑定到哪个生命周期阶段的信息，需要注意的是仅仅指定插件本身是不够的，必须还要指明在构建过程中要运行的插件目标。

目标的配置会应用到已经被绑定到指定打包类型的目标上。如果有多个目标绑定一个特定构建阶段，则目标的运行顺序是先执行绑定到打包类型上的目标，然后再执行`pom`文件中配置的目标。这些配置可以使用`<executions>`元素来进行详细的控制。

##### 例子

例如，`Modello`插件的目标`modello:java`被默认绑定到了`generate-sources`阶段（注：`maven`中的插件目标一般会有默认绑定的阶段，所以如果没有指定绑定阶段则目标会在默认阶段执行，[详细内容可见maven手册](http://maven.apache.org/developers/mojo-api-specification.html)。因此，要使用`Modello`插件并在构建过程中执行`java`目标，可以通过在`pom`文件中的`<build>`的元素的子元素`<plugins>`中添加以下内容：

```xml
 <plugin>
   <groupId>org.codehaus.modello</groupId>
   <artifactId>modello-maven-plugin</artifactId>
   <version>1.8.1</version>
   <executions>
     <execution>
       <configuration>
         <models>
           <model>src/main/mdo/maven.mdo</model>
         </models>
         <version>4.0.0</version>
       </configuration>
       <goals>
         <goal>java</goal>
       </goals>
     </execution>
   </executions>
 </plugin>
```

请注意，在`<executions>`元素中`<goal>`元素是一定要指定的。当有多个`<execution>`指定到特定的阶段时，它们将按照`pom`文件中指定的顺序执行，并且先执行子类的`<execution>`。

对于上面的例子，`modello:java`只会在它的默认阶段`generate-sources`执行，但你可能会有自己指定阶段的需求。例如，假设目标`display:time`可以输出当前时间到命令行，并且你希望它在`process-test-resources`阶段运行以显示测试开始的时间。你可以进行如下配置：

```xml
 <plugin>
   <groupId>com.mycompany.example</groupId>
   <artifactId>display-maven-plugin</artifactId>
   <version>1.0</version>
   <executions>
     <execution>
       <phase>process-test-resources</phase>
       <goals>
         <goal>time</goal>
       </goals>
     </execution>
   </executions>
 </plugin>
```