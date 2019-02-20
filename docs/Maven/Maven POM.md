## 概述

本文将对`Maven`的项目对象模型（`POM`）进行简单的介绍。

**索引**

[TOC]

本文的主要是笔者对`Maven`官方手册的翻译和理解：http://maven.apache.org/guides/introduction/introduction-to-the-pom.html

参考了W3Cschool的`Maven`教程：https://www.w3cschool.cn/maven/varq1ht4.html

## POM简介

### 什么是POM？

`POM`即`Project Object Model`（项目对象模型）是`Maven`的基本工作单元。它是一个`xml`文件，文件命名为`pom.xml`，其中包含了关于项目和各种配置细节的信息，`Maven` 使用这些信息构建工程。**当执行一个任务或者目标时，`Maven` 会寻找当前目录下的 `pom.xml`文件，从其中读取所需要的配置信息，然后执行目标。**（即：当你在命令行输入`mvn`命令时，`Maven`会在执行命令的目录下寻找`pom.xml`并且根据其中的配置执行相应的命令）

可以在`pom`文件中指定的信息包括项目依赖（`dependencies`）、插件（`plugins `）或者目标（`goals` ）、构建配置（`profiles`）等。其他信息，例如项目版本（`version`）、描述（`description`）等等，也可以在`pom`文件中指定。

### SuperPOM

在`Maven`中，除非显示指定父`POM`，否则所有的 `POM` 都会从`Super POM`继承，这意味着在 `Super POM`进行的配置都会被未显示指定父`POM`的子`POM`继承。

下面的代码段是`Maven 2.1.x`的`Super POM`配置：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <name>Maven Default Project</name>
 
  <repositories>
    <repository>
      <id>central</id>
      <name>Maven Repository Switchboard</name>
      <layout>default</layout>
      <url>http://repo1.maven.org/maven2</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
 
  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Maven Plugin Repository</name>
      <url>http://repo1.maven.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
 
  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <!-- TODO: MNG-3731 maven-plugin-tools-api < 2.4.4 expect this to be relative... -->
    <scriptSourceDirectory>src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
   <pluginManagement>
       <plugins>
         <plugin>
           <artifactId>maven-antrun-plugin</artifactId>
           <version>1.3</version>
         </plugin>       
         <plugin>
           <artifactId>maven-assembly-plugin</artifactId>
           <version>2.2-beta-2</version>
         </plugin>         
         <plugin>
           <artifactId>maven-clean-plugin</artifactId>
           <version>2.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-compiler-plugin</artifactId>
           <version>2.0.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-dependency-plugin</artifactId>
           <version>2.0</version>
         </plugin>
         <plugin>
           <artifactId>maven-deploy-plugin</artifactId>
           <version>2.4</version>
         </plugin>
         <plugin>
           <artifactId>maven-ear-plugin</artifactId>
           <version>2.3.1</version>
         </plugin>
         <plugin>
           <artifactId>maven-ejb-plugin</artifactId>
           <version>2.1</version>
         </plugin>
         <plugin>
           <artifactId>maven-install-plugin</artifactId>
           <version>2.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-jar-plugin</artifactId>
           <version>2.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-javadoc-plugin</artifactId>
           <version>2.5</version>
         </plugin>
         <plugin>
           <artifactId>maven-plugin-plugin</artifactId>
           <version>2.4.3</version>
         </plugin>
         <plugin>
           <artifactId>maven-rar-plugin</artifactId>
           <version>2.2</version>
         </plugin>        
         <plugin>                
           <artifactId>maven-release-plugin</artifactId>
           <version>2.0-beta-8</version>
         </plugin>
         <plugin>                
           <artifactId>maven-resources-plugin</artifactId>
           <version>2.3</version>
         </plugin>
         <plugin>
           <artifactId>maven-site-plugin</artifactId>
           <version>2.0-beta-7</version>
         </plugin>
         <plugin>
           <artifactId>maven-source-plugin</artifactId>
           <version>2.0.4</version>
         </plugin>         
         <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.4.3</version>
         </plugin>
         <plugin>
           <artifactId>maven-war-plugin</artifactId>
           <version>2.1-alpha-2</version>
         </plugin>
       </plugins>
     </pluginManagement>
  </build>
 
  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>
  <profiles>
    <profile>
      <id>release-profile</id>
 
      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>
 
      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
 
</project>
```

由于`Maven`的继承关系可能比较复杂，如果你想查看一个`pom.xml`文件完整的配置（父`pom`的配置加上自己的配置） ，可以通过在`pom.xml`文件对应的目录下执行 `help:effective-pom`目标来查看。命令如下：

```shell
mvn help:effective-pom
```

### POM文件的最简配置

`POM`文件最精简的配置要求有以下几个：

- project 根元素
- modelVersion - 应该被设置为 4.0.0
- groupId -项目组id
- artifactId - 项目（工件）id
- version - 特定项目组下的工件版本

下面是最简配置的例子：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

`POM`文件必须配置其`groupId`，`artifactId`，`version`这三个字段。 这三个值以`<groupId>：<artifactId>：<version>`的形式构成项目的完整的项目名称。例如，上面的例子完整的名称是`com.mycompany.app:my-app:1`。

另外，如果未在配置文件中指定元素的值，`Maven`将使用其默认值。例如，每个`Maven`项目都有一种打包类型，如果元素`<packaging>`未在`pom.xml`内指定，那么`Maven`将会使用`jar`作为该项目的打包类型。

在上面的`POM`文件中并未指定依赖仓库，因为它将继承了`Super POM`中的存储库配置，它会知道这些依赖关系将从`Super POM`中指定的http://repo.maven.apache.org/maven2仓库中下载。

### 项目继承

子项目继承父项目后，子`POM`会将会把父`POM`中的以下几个元素进行合并：

- dependencies
- developers and contributors
- plugin lists (including reports)
- plugin executions with matching ids
- plugin configuration
- resources

你可以在`pom.xml`文件中配置`<parent>`元素来指定某个`pom.xml`文件的父`POM`。

#### 例子1

现在有另一个名称为 `com.mycompany.app:my-module:1`的子项目，`pom.xml`文件如下：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```

项目结构如下：

```shell
.
 |-- my-module     #注意这里my-module是项目中的一个子模块
 |   `-- pom.xml   #com.mycompany.app:my-module:1 的POM文件
 `-- pom.xml       #com.mycompany.app:my-app:1的POM文件
```

将[最简`pom`文件](#pom_2)(`com.mycompany.app:my-app:1`)作为`com.mycompany.app:my-module:1`项目的父`POM`，可以在`com.mycompany.app:my-module:1`的`POM`添加`<parent>`元素如下：

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```

通过指定父`POM`的`groupId`，`artifactId`，`version`，可以继承父`POM`一些已经设置好的属性。

如果我们希望子`POM`的`groupid`或者`version`与父`POM`保持一致，则可以选择删除子`POM`的`groupid`或者`version`元素，如下：

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```

#### 例子2

[例子1](#1)中子`POM`的配置能生效的前提是父项目已经安装到本地仓库或者子`POM`在特定的目录结构中（父`pom.xml`文在所在的目录是子模块的`pom.xml`所在的目录的上层目录[上层目录举例：`/src/main`是`/src/main/java`的上层目录]）。

所以当目录结构如下时，需要进行额外的配置：

```shell
.
 |-- my-module
 |   `-- pom.xml
 `-- parent
     `-- pom.xml
```

为了处理这种目录结构，我们需要在`<parent>`元素下添加` <relativePath>`元素，如下：

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```

### 项目聚合

不同于项目继承是从子模块中指定父`POM`，项目聚合是在父`POM`中指定子模块。通过这种方式，父项目可以知道它所有的子模块，当`Maven`命令在父项目上调用时，它的所有子模块也会按顺序执行相同的`Maven`命令。要实现项目聚合，需要做以下操作：

- 把父`POM`的打包类型（`<packaging>`元素）修改为`pom`
- 在父`POM`中指定子模块列表（通过`<modules>`元素）

#### 例子3

**com.mycompany.app:my-app:1's POM**

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

**com.mycompany.app:my-module:1's POM**

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>

```

**目录结构**

```shell
.
 |-- my-module
 |   `-- pom.xml
 `-- pom.xml
```

如果我们要把`my-module`项目聚合到`my-app`项目，我们只要修改`my-app`的`pom.xml`文件，如下：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>my-module</module>
  </modules>
</project>
```

我们在`pom.xml`文件中添加了`<packaging>`元素和`<modules>`元素，并指定了相应的值。需要注意的是`<module>`元素的值是两个项目`POM`文件的相对路径，并且必须使用项目的`<artifactId>`作为项目的目录名称（这里好像是以父`POM`所在的路径作为根路径）。

### 项目继承VS项目聚合

如果你的项目是由几个`Maven`项组成，并且他们配置有相同或者类似的部分，这时你可以选择创建一个父项目把相同或者类似的配置在父项目中定义，然后让你的子项目继承这个父项目从而达到减少重复配置的目的。

如果你的需要同时处理或者构建一组项目，则可以创建一个父项目并且在父项目中声明这些子模块，这样你只要执行一次命令就能同时处理多个项目。

当然，你也可以同时使用继承和聚合功能，它们并不是互斥的。

### 项目变量以及插值

`Maven`鼓励的做法是不要重复自己，但某一些情况下你可能需要在几个不同的位置使用相同的值。为了确保你只需要在项目中指定一次值（当你需要修改该值时也只需要修改一次），`Maven`允许在`pom.xml`文件中使用预定义的变量和自己定义的变量。

例如，你可以通过以下方式来获得`project.version`的值：

```xml
 <version>${project.version}</version>
```

需要注意的是，这些变量会在继承之后进行处理，这意味着如果父项目和子项目同时定义了相同的变量，则在子项目中将会以子项目定义的变量为准。

#### 可用变量

**项目模型变量**

作为单个值元素的模型的任何字段都可以作为变量引用。 例如，`$ {project.groupId}`，`$ {project.version}`，`$ {project.build.sourceDirectory}`等等。 这些变量都以`project.`作为前缀， 你也可以看到以`pom.`作为前缀的引用。 请参阅[Maven Model Builder](http://maven.apache.org/ref/3-LATEST/maven-model-builder/)以查看完整的属性列表。

**自定义变量**

你可以在`<propeties>`元素里自定义变量，例如你可以通过如下方式声明一个插件版本：

```xml
<project>
  ...
  <properties>
    <maven.checkstyle.version>7.6</maven.checkstyle.version>
  </properties>
  ...
</project>
```

并且可以通过以下方式在插件声明时使用：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>${maven.checkstyle.version}</version>
</plugin>
```