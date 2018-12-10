---
title: Maven
date: 2017-01-01 09:00:00
tags: [工具]
---

# 简介

Maven 是一个项目管理工具，包含：

- 一个项目对象模型（Project Object Model）
- 一组标准集合
- 一个项目生命周期（Project Lifecycle）
- 一个依赖管理系统（Dependency Management System）
- 用来运行定义在生命周期阶段（phase）中插件目标（goal）的逻辑

# 基本步骤

- `mvn install:install-file  -Dfile=D:/xxx.jar  -DgroupId=xxx.xxx  -DartifactId=xxx -Dversion=x.x -Dpackaging=jar` 将指定 jar 包安装至本地库

若要使打包得到的 .jar 中包含 main 方法（即为可执行 .jar），需要借助 `maven-shade-plugin`，配置该插件如下：

<!-- more -->

```XML
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>1.2.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation =  “org.apache.maven.plugins.shade.resource.ManifestResourceTransformer”>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

其中，`<plugin>` 元素在 `<project><build><plugins>` 下面。
maven 中下载的插件都在 `~\.m2\repository\org\apache\maven` 文件夹内。

在使用 mvn package 进行编译、打包时，Maven 会执行 src/test/java 中的 JUnit 测试用例，有时为了跳过测试，会使用参数 `-DskipTests` 和 `-Dmaven.test.skip=true`，这两个参数的主要区别是：
`-DskipTests` :不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下。
`-Dmaven.test.skip=true` :不执行测试用例，也不编译测试用例类。


# 依赖的配置

*“[...]”表示可选*

```XML
<project>
    <modelVersion>...</modelVersion>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <name>...</name>
    <version>...</version>

    [
    <properties>
        <...>...</...>
        ...
    </properties>
    ]

    <dependencies>
        <denpendency>
            <gourpId>...</groupId>
            <artifactId>...</artifactId>
            <version>...</version>
            [<type>...</type>]通常不写
            [<scope>...</scope>]
            [<optional>...</optional>]
            [<exclusions>
                <exclusion>
                    <groupId>...</groupId>
                    <artifactId>...</artifactId>
                    ...
                </exclusion>
                ...
            </exclusions>]
        </dependency>
        ...
    </dependencies>
    ...
</project>
```

- groupId、artifactId 和 version：依赖的基本坐标。
- type：依赖的类型，对应于项目坐标定义的 packaging。通常不声明，默认为 jar。
- scope：依赖的范围。用来控制依赖与不同 classpath 的关系。详见“依赖范围与 classpath 的关系”。
    - compile：编译依赖范围。默认值。对编译、测试、运行有效。
    - test：测试依赖范围。对测试有效。
    - provided：已提供依赖范围。对编译、测试有效。
    - runtime：运行时依赖范围。对测试、运行有效。
    - import(Maven2.0.9及以上)：从另一个 `<dependencyManagement>` 导入依赖范围。
    - system：系统依赖范围。同 provided，但是必须通过 `systemPath` 元素显式指定依赖文件路径。由于不是通过 Maven 仓库解析，且往往与本机系统绑定，可能造成构建的不可移植。`systemPath` 元素可引用环境变量，如：`<systemPath>${java.home}/lib/rt.jar</systemPath>`。
- optional：标记依赖是否可选，若为true则该依赖不会被传递。
- exclusions：排除传递性依赖。

**依赖范围与 classpath 的关系**

|依赖范围（scope）|编译|测试|运行|
|:---------------:|:--:|:--:|:--:|
|compile	        |1   |1	  |1   |
|test	            |-   |1	  |-   |
|provided	        |1   |1	  |-   |
|runtime	        |-   |1   |1   |
|system	          |1   |1	  |-   |

**依赖范围影响传递性依赖**

|第一依赖\第一依赖|compile |test|provided|runtime |
|:---------------:|:-----: |:--:|:------:|:------:|
|compile          |compile |-   |-       |runtime |
|test             |test    |-   |-       |test    |
|provided         |provided|-   |provided|provided|
|runtime	        |runtime |-   |-       |runtime |

`mvn dependency:list`：查看当前项目的已解析依赖。
`mvn dependency:tree`：查看当前项目的依赖树。
`mvn dependency:analyze`：分析当前项目的依赖树。其中“Used undeclared dependencies”指项目中使用到的，但未显式声明的依赖；“Unused declared dependencies”指项目中未使用的，但显式声明的依赖。该命令只会分析编译阶段（包括主代码和测试代码）用到的依赖，一些测试和运行时需要的依赖它发现不了，因此需要经过仔细分析再决定是否删除未使用的依赖。
`mvn dependency:list --> C:/list.txt`：将结果输出到指定文件。其它两条命令同理。


# 远程仓库、镜像仓库

**远程仓库的配置**

在 `pom.xml` 中配置 repository：

```XML
<project>
    ...
    <repositories>
        <repository>
            <id>...</id>
            <name>...</name>
            <url>...</url>
            <releases>
                <enabled>true/false</enabled>
                [<updatePolicy>daily(默认)/never/always/interval:X</updatePolicy>]
                [<checksumPolicy>warn(默认)/fail/ignore</checksumPolicy>]
            </releases>
            <snapshots>
                <enabled>true/false</enabled>
                [<updatePolicy>daily(默认)/never/always/interval:X</updatePolicy>]
                [<checksumPolicy>warn(默认)/fail/ignore</checksumPolicy>]
            </snapshots>
            <layout>default</layout>
        </repository>
    </repositories>
    ...
</project>
```

在 `settings.xml` 中配置仓库认证信息

```
<settings>
    ...
    <servers>
        <server>
            <id>必须与POM文件中认证的repository元素的id一致</id>
            <username>...</username>
            <password>...</password>
        </server>
    </servers>
    ...
</settings>
```

部署到远程仓库，在 `pom.xml` 中配置构件部署地址：

```
<project>
    ...
    <distributionManagement>
        <repository>
            <id>...</id>
            <name>...</name>
            <url>...</url>
        </repository>
        <snapshotRepository>
            <id>...</id>
            <name>...</name>
            <url>...</url>
        </snapshotRepository>
    </distributionManagement>
    ...
</project>
```

配置正确后，运行 `mvn clean deploy` 进行部署。

**配置镜像**

```
<settings>
    ...
    <mirrors>
        <mirror>
            <id>...</id>
            <name>...</name>
            <url>...</url>
            <mirrorOf>...</mirrorOf>
        </mirror>
    </mirrors>
    ...
</settings>
```
- `<mirrorOf>*</mirrorOf>`：匹配所有远程仓库
- `<mirrorOf>external:*</mirrorOf>`：匹配所有不在本机上的远程仓库，即排除localhost、file://
- `<mirrorOf>repo1,repo2</mirrorOf>`：逗号分隔多个远程仓库
- `<mirrorOf>*,!repo1</mirrorOf>`：使用感叹号将仓库排除

---
# 生命周期、阶段、插件
Maven 有三套生命周期，互相独立，目的不同。

- `clean` 清理项目
    - pre-clean 执行一些清理前需要完成的工作
    - clean 清理上一次构建生成的文件
    - post-clean 执行一些清理后需要完成的工作
- `default` 构建项目
    - validate
    - initialize
    - generate-sources
    - process-sources 处理项目主资源文件。一般来说，是对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中。
    - generate-resources
    - process-resources
    - **compile** 编译项目的主源码。一般来说，是编译 src/main/java 目录下的 Java 文件至项目输出的主 classpath 目录中。
    - process-classes
    - generate-test-sources
    - process-test-sources 处理项目测试资源文件。一般来说，是对 src/test/resources 目录的内容进行变量替换等工作后，复制到项目输出的测试 classpath 目录中。
    - generate-test-resources
    - process-test-resources
    - test-compile 编译项目的测试代码。一般来说，是编译 src/test/java 目录下的 Java 文件至项目输出的测试 classpath 目录中。
    - process-test-classes
    - test 使用单元测试框架运行测试，测试代码不会被打包或部署。
    - prepare-package
    - **package** 接受编译好的代码，打包成可发布的格式，如 JAR。
    - pre-integration-test
    - integration-test
    - post-integration-test
    - verify
    - **install** 将包安装到 Maven 本地仓库。
    - **deploy** 将最终的包复制到远程仓库。
- `site` 建立项目站点
    - pre-site 执行一些在生成站点之前需要完成的工作。
    - site 生成项目站点文档。
    - post-site 执行一些在生成项目站点之后需要完成的工作。
    - site-deploy 将生成的项目站点发布到服务器上。

每个生命周期包含一些**阶段**（phase），这些阶段是有顺序的，后面阶段依赖于前面阶段；不同生命周期的阶段不会互相触发。

**命令行与生命周期**

- `mvn clean`：调用 clean 生命周期的 clean 阶段。实际执行的阶段为 pre-clean 和 clean。
- `mvn clean install`：可以同时调用多个不同生命周期中的阶段。调用 clean 生命周期的 clean 阶段和 default 生命周期的 install 阶段。

**插件绑定**

Maven 的生命周期与插件相互绑定，用以完成实际的构建任务。

**表3 clean 生命周期阶段与插件目标的绑定关系**

|生命周期阶段|插件目标                 |
|:----------:|:-----------------------:|
|pre-clean   |	                       |
|clean       |maven-clean-plugin:clean |
|post-clean  |                         |

**表4 site 生命周期阶段与插件目标的绑定关系**

|生命周期阶段|插件目标                 |
|:----------:|:-----------------------:|
|pre-site    |                         |	
|site	       |maven-site-plugin:site   |
|post-site   |        	               |
|site-deploy |maven-site-plugin:deploy |

**表5 default 生命周期的内置插件绑定关系及具体任务（打包类型：jar）**

|生命周期阶段	         |插件目标	                          |执行任务                      |
|:--------------------:|:----------------------------------:|:----------------------------:|
|process-resources     |maven-resources-plugin:resources    |复制主资源文件至主输出目录    |
|compile               |maven-compiler-plugin:compile	      |编译主代码至主输出目录        |
|process-test-resources|maven-resources-plugin:testResources|复制测试资源文件至测试输出目录|
|test-compile	         |maven-compiler-plugin:testCompile   |编译测试代码至测试输出目录    |
|test	                 |maven-surefire-plugin:test	        |执行测试用例                  |
|package	             |maven-jar-plugin:jar	              |创建项目 jar 包               |
|install	             |maven-install-plugin:install	      |将项目输出构件安装到本地仓库  |
|deploy	               |maven-deploy-plugin:deploy	        |将项目输出构件部署到远程仓库  |

Maven 核心内置绑定了主要的生命周期阶段与插件的目标。用户也可自定义绑定。

**自定义绑定插件目标：**
*复制使用时可选择删除注释*

```
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <!-- Maven官方插件的groupId -->
                <groupId>org.apache.maven.plugins</groupId>
                <!-- Maven官方插件的artifactId -->
                <artifactId>maven-source-plugin</artifactId>
                <!-- 应选择非快照版本，避免不稳定性 -->
                <version>2.1.1</version>
                <executions>
                    <!-- 每个<execution>元素可用来配置执行一个任务 -->
                    <execution>
                        <!-- 任务id -->
                        <id>attach-sources</id>
                        <!-- 绑定到verify阶段上 -->
                        <phase>verify</phase>
                        <!-- 要执行的插件目标 -->
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```

---
# 聚合、继承

**聚合**

目的是能够使用一条命令就构建多个模块（或称项目）。需要建立一个聚合项目，该项目有自己的POM，如下，其中关键元素是 `<modules>`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0:"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <!-- 注意packaging的值 -->
    <packaging>pom</packaging>
    <name>...</name>
    <modules>
        <module>模块name元素（父子目录结构）</module>
        <module>../模块name元素（平行目录结构）</module>
        ...
    </modules>
</project>
```

**继承**

父模块

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <!-- 注意packaging的值，同聚合中packaging的值 -->
    <packaging>pom</packaging>
    <name>...</name>
</project>
```

子模块

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <parent>
        <groupId>...</groupId>
        <artifactId>...</artifactId>
        <version>...</version>
        <relativePath>
            指明父模块的pom.xml的相对路径，无此元素则会自动到本地仓库查找
            ../ParentModuleName/pom.xml平行结构
            ../pom.xml父子结构
        </relativePath>
    </parent>

    <artifactId>...</artifactId>
    <name>...</name>
    ...
</project>
```

可继承的POM元素

- `groupId`：项目组ID，项目坐标的核心元素。
- `version`：项目版本，项目坐标的核心元素。
- `description`：项目的描述信息。
- `organization`：项目的组织信息。
- `inceptionYear`：项目的创始年份。
- `url`：项目的URL地址。
- `developers`：项目的开发者信息。
- `contributors`：项目的贡献者信息。
- `distributionManagement`：项目的部署配置。
- `issueManagement`：项目的缺陷跟踪系统信息。
- `ciManagement`：项目的持续集成系统信息。
- `scm`：项目的版本控制系统信息。
- `mailingLists`：项目的邮件列表信息。
- `properties`：自定义的Maven属性。
- `dependencies`：项目的依赖配置。
- `dependencyManagement`：项目的依赖管理配置。
- `repositories`：项目的仓库配置。
- `build`：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等。
- `reporting`：包括项目的报告输出目录配置、报告插件配置等。

依赖管理
在父模块中加入`<dependencyManagement>`元素

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>...</groupId>
                <artifactId>...</artifactId>
                <version>...</version>
                [<scope>...</scope>]
            </dependency>
            ...
        </dependencies>
    </dependencyManagement>
    ...
</project>
```

子模块中，对于继承的依赖，不需配置 `<version>` 和 `<scope>`，且只会继承在子模块中声明的依赖，自动忽略了父模块中声明了而子模块中未声明的依赖。这种方式能有效避免依赖的版本冲突。

<span id="importScope"></span>
`<scope>import</scope>` 只在 `<dependencyManagement>` 元素中起作用，用来拷贝另一个父模块的 `<dependencyManagement>` 中的内容。
[返回](#returnFromImportScope)

**反应堆**
反应堆（Reactor）指所有模块组成的构建结构。
通过命令行剪裁反应堆：

- `-am`：also-make同时构建所列模块的依赖模块。
- `-amd`：also-make-dependents同时构建依赖于所列模块的模块。
- `-pl`：projects[arg]构建指定的模块，模块间用逗号分隔（不含方括号，下同）。
- `rf`：resume-from[arg]从指定的模块回复反应堆。

几个命令可组合使用。

# 问题

## mvn 编译报错 -source 1.5 不支持...

maven compiler 默认 jdk 版本是 1.5，要更高版本需要手动指定。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```








