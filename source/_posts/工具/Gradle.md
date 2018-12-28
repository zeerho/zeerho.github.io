---
title: Gradle
date: 2018-08-16 09:00:00
tags: [工具]
---

# 命令

`gradle {taskName} [options]`

- `-i`: 改变日志级别为 INFO
- `-s`: 如果运行时错误发生打印堆栈信息
- `-q`: 只打印错误信息
- `-?`,`-h`,`--help`: 打印所有的命令行选项
- `-b`,`--build-file`: 执行其他脚本（默认是 build.gradle）
- `--offline`: 在离线模式运行，只检查本地缓存中的依赖
- `-D`,`--system-prop`: Gradle 作为 JVM 进程运行，可以提供一个系统属性，比如：-Dmyprop=myValue
- `-P`,`--project-prop`: 传递一个属性值给 build 脚本，比如：-Pmyprop=myValue
- `-x`: 排除指定的任务

自带的命令：

- `tasks`: 显示所有可运行的任务。
- `properties`: 显示项目中所有属性。


# 常用配置

配置脚本中的方法一般都在 gradle 项目各子模块的 `org.gradle.api` 包下。相应的实现一般在对应的 `org.gradle.api.internal` 及其子包下。

```groovy
allprojects {
  repositories {
    maven {
      url 'theUrl'
    }
  }
}

// java 项目用到的插件
apply plugin: 'java'

// 上传至 maven 仓库
apply plugin: 'maven'

uploadArchives {
  repositories.mavenDeployer {
    repository(url: "http://localhost:8088/nexus/content/repositories/snapshots/") {
      authentication(userName: "admin", password: "admin123")
        pom.groupId = "com.example"
        pom.artifactId = "my-example"
    }
  }
}

// 默认文件结构跟 maven 一样，也可以自定义
sourceSets {
  main {
    java {
      srcDir 'src/java'
    }
    resources {
      srcDir 'src/resources'
    }
  }
}
```

# 基本元素

## Script

`Script` 脚本用来定义一个 `Project`。

**Init Script**

每次构建之前都会执行初始化脚本，用来设置一些全局变量。存放初始化脚本的位置包括：

- $USER/.gradle/
- $USER/.gradle/init.d/
- $GRADLE_HOME/init.d/

**Settings Script**

`settings.gradle` 用来配置多模块的项目。位于项目根目录下。

**Build Script**

每个 `build.gradle` 中包含两种元素：

- statement：方法调用、属性赋值、局部变量等；
- script blocks：脚本块。即一个接受闭包作为参数的方法，该闭包用来配置它的委托对象。


脚本在运行期会被编译成一个实现了 `Script` 接口的类，每个该类的实例都有各自的委托对象：

- Build Script -> Project
- Init Script -> Gradle
- Settings Script -> Settings

## Project

每个 `build.gradle` 与一个 `Project` 对应。构建时 gradle 通过以下方式创建 `Project` 对象：

1. 根据 `settings.gradle` 创建一个 `Settings` 对象；
2. 根据 `Settings` 中定义的工程父子关系创建 `Project` 对象；
3. 执行每个工程的 `build.gradle` 对相应的 `Project` 对象进行配置。

`build.gradle` 文件中定义的属性和方法会委托给 `Project` 执行。

属性范围：

1. `Project`；
2. `Project` 的 `extra` 属性？
3. 通过 `plugin` 添加的 `extension`，每个 `extension` 通过一个和 `extension` 同名的只读属性访问；
4. 通过 `plugin` 添加的属性。一个 `plugin` 可以通过 `Project` 的 `Convention` 对象为 `Project` 添加属性和方法；
5. `Project` 中的 `task`，一个 `task` 对象可以通过 `Project` 中的同名属性访问，但是它是只读的；
6. 当前 `Project` 的父工程的 `extra` 属性和 `convention` 属性；

方法范围：

1. `Project` 对象本身；
2. `build.gradle` 文件中定义的方法；
3. 通过 `plugin` 添加的 `extension`，每个 `extension` 都作为一个接受一个闭包/Action 作为参数的方法被访问；
4. 通过 `plugin` 添加的方法。一个 `plugin` 可以通过 `Project` 的 `Convention` 对象为 `project` 添加属性和方法。
5. `Project` 中的 `task`。每一个 `task` 都会在当前 `Project` 中存在一个接受一个闭包或者/Action 作为参数的方法，这个闭包会在 `task` 的 `configure(closure)` 方法中调用。
6. 当前工程的父工程中的方法；
7. 当前工程的属性可见范围中所有的闭包属性都可以作为方法访问

## Task

`Project` 中的原子性工作，比如编译、单元测试等。

# 构建的生命周期

1. 初始化：gradle 决定构建哪些模块，并创建相应的 `Project` 对象。对于多模块的项目，即使指定了要构建的子模块，也会执行所有在 Settings Script 中的模块的 Build Script。
2. 配置：执行此次构建所包含的所有模块的 Build Script 来配置各个 `Project` 对象。默认会配置所有 `Project`。1.4 引入了新特性：`--configuration-on-demmand`，使用该选项，即使模块被包含在 `settings.gradle` 中，只要不参与本次构建，那么 `Project` 会被创建但不会被配置。
3. 执行：gradle 决定执行哪些任务，生成任务依赖图（DAG）。在 Build Script 中出现的任务都会被创建，但如果无需执行的话不会出现在 DAG 里。
