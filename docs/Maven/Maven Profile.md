## 概述

本文主要是对`Maven`的构建配置文件进行简单的介绍。

**索引**

[TOC]

本文的主要是笔者对`Maven`官方手册的翻译和理解：[http://maven.apache.org/guides/introduction/introduction-to-profiles.html](http://maven.apache.org/guides/introduction/introduction-to-profiles.html)

参考了W3Cschool的`Maven`教程：[https://www.w3cschool.cn/maven/tchv1ht8.html](https://www.w3cschool.cn/maven/tchv1ht8.html)

## 构建配置文件（Profiles）简介

### 为什么要有构建配置文件

`Maven`的宗旨是尽可能确保构建的可移植性，但在某些情况下可移植并不是完全可能的。例如，可能需要使用本地文件系统环境变量来配置插件，或者根据构建环境的不同来选择使用不同的插件等。为了解决这些问题，`Maven`引入了构建配置文件的概念，通过在`<profiles>`元素中额外或者再次配置`POM`中某些元素，并通过各种条件来触发`<profile>`的配置。*`<profile>`会根据触发条件在构建时修改`POM`，这意味着`<profiles>`作为补充集合在构建时会根据不同的目标环境提供等价但不同的参数*。这样，项目仍然能够保持可移植性，根据不同的构建需求生成不同的构建结果。

### Profile的类型以及可以定义的位置

|         类型          |                          定义的位置                          |
| :-------------------: | :----------------------------------------------------------: |
| 项目（`Per Project`） |                      在`pom.xml`中定义                       |
|  用户（`Per User`）   |    在`Maven`设置中定义（`%USER_HOME%/.m2/settings.xml`）     |
|   全局（`Global`）    | 在`Maven`全局设置中定义（`${maven.home}/conf/settings.xml`） |

### Profile的触发方式

`Profile` 可以通过以下几种方式触发/激活：

- 直接通过命令行触发
- 通过`Maven`配置来设置
- 取决于环境参数
- 操作系统设置
- 存在/缺少文件

#### Profile的触发细节

**1.命令行**

`Profiles`可以通过直接指定命令行选项`-P`来激活。这个命令行选项接受一个被逗号（`,`）分割的`profile id`列表（注：每个profile都可以设置`id`这个标签），在`-P`的参数中指定的`profile`将会在构建时被激活，此外任何符合其`<activation>`条件（`profile`一般在它的子元素`<activation>`中定义触发条件）或者在`settings.xml`文件中的`<activeProfiles>`部分定义的`profile`也会被激活。

```shell
mvn groupId:artifactId:goal -P profile-1,profile-2
```

**2.`Maven`配置**

`Profiles`可以通过`Maven`配置来设置激活，即通过上文提到的`<activeProfiles>`部分进行配置，该部分包含一个或多个以`profile id`作为参数的`<activeProfile>`元素。如果在项目中使用了`<activeProfiles>`，那么在其中指定的`profile`列表在每次都会被激活。

```xml
<settings>
  ...
  <activeProfiles>
    <activeProfile>profile-1</activeProfile>
  </activeProfiles>
  ...
</settings>
```

**3.其他**

可以基于自动检测构建环境状态来触发`Profiles`，这些触发器通过`Profile`自身的子元素`<activation>`来指定。目前触发器只能检测`JDK`的大版本、系统属性存在与否以及系统属性的值（这里的系统属性指的是命令行参数中的属性或者是`settings.xml`文件的属性，详见下面的几个例子）。

下面的例子中，以`JDK`大版本为`1.4`(例如，`1.4.0_08`,`1.4.2_07`)的情况下会触发该`Profile`

```xml
<profiles>
  <profile>
    <activation>
      <jdk>1.4</jdk>
    </activation>
    ...
  </profile>
</profiles>
```

你也可以以范围的形式来设置`JDK`版本，下面的配置将会在`1.3, 1.4, 1.5`版本起作用

```xml
<profiles>
  <profile>
    <activation>
      <jdk>[1.3,1.6)</jdk>
    </activation>
    ...
  </profile>
</profiles>
```

`Profile`也可以通过操作系统参数触发，如下

```xml
<profiles>
  <profile>
    <activation>
      <os>
        <name>Windows XP</name>
        <family>Windows</family>
        <arch>x86</arch>
        <version>5.1.2600</version>
      </os>
    </activation>
    ...
  </profile>
</profiles>
```

在下面的例子中，`Profile`会在属性`debug`未定义或者定义为非`true`值时被激活

```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>debug</name>
        <value>!true</value>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```

激活上面的例子可以通过执行下面的命令中的一个

```shell
mvn groupId:artifactId:goal
mvn groupId:artifactId:goal -Ddebug=false
```

下面的例子会在属性`environment`的值被指定为`test`时触发

```xml
<profiles>
  <profile>
    <activation>
      <property>
        <name>environment</name>
        <value>test</value>
      </property>
    </activation>
    ...
  </profile>
</profiles>
```

要触发上面的例子，你可以执行下面的命令

```shell
mvn groupId:artifactId:goal -Denvironment=test
```

下面的例子会在文件`target/generated-sources/axistools/wsdl2java/org/apache/maven`缺失的情况下触发`Profile`

```xml
<profiles>
  <profile>
    <activation>
      <file>
        <missing>target/generated-sources/axistools/wsdl2java/org/apache/maven</missing>
      </file>
    </activation>
    ...
  </profile>
</profiles>
```

`<exists>`和`<missing>`标签支持以插值的形式赋值，但仅支持系统属性(如`${user.home}`)和环境变量（如`${env.HOME}`），并不支持用户在`pom.xml`自定义的属性（`<properties>`中定义的属性）。

`Profile`也可以设置为默认激活，如下：

```xml
<profiles>
  <profile>
    <id>profile-1</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    ...
  </profile>
</profiles>
```

你也可以在命令行执行在`Profile id`前添加`！`或者`-`字符来取消激活`Profile`，如下：

```shell
mvn groupId:artifactId:goal -P !profile-1,!profile-2
```

### 在Profile中能够指定的内容

你能够在`Profile`中指定的内容取决于你选择在什么位置配置你的`Profile`，当配置位置不同时你能够指定的内容也不一样。

**在外部文件中配置`Profile`**

在外部文件中配置`Profile`（即`settings.xml`或`profiles.xml`）在严格意义上是不可移植的，所以能够大概率改变构建结构的内容只能在`POM`中进行配置。因此，你只能在外部文件中修改`<repositories>、<pluginRepositories>`以及`<properties>`部分。

**在`POM`中配置`Profile`**

如果你可以在`POM`中合理地配置`Profile`，那你将会有更多的配置选项。这样你可以保留更多的可移植性，即可以添加更多的信息而不会有其他用户获得不到信息的风险。在`POM`中`Profile`可以修改以下元素：

- `<repositories>`
- `<pluginRepositories>`
- `<dependencies>`
- `<plugins>`
- `<properties>`
- `<modules>`
- `<reporting>`
- `<dependencyManagement>`
- `<distributionManagement>`
- `<build>`元素的子集，由以下元素组成：
	- `<defaultGoal>`
	- `<resources>`
	- `<testResources>`
	- `<finalName>`