---
title: spring源码下载编译阅读
tags: 
	- spring
categories: spring
date: 2021-3-30 00:42:13
---


# Spring源码下载编译

## 1. 前言

​	经过了两年多的工作沉淀, 该掌握的技术体系已经能过熟练运用了.但很多时候,都是知道具体的解决方法,至于为什么可以用此方法解决问题却一知半解.

​	之前也有尝试着看过源码,但debug模式,一步步执行下去后,就被绕晕了.执行到哪了都不知道,有时候又感觉绕回了原来的地方.当然,没有添加自己的注释也是一个很大的原因.给自己当时理解了的地方打上注释,等下次运行到时查看注释帮助理解,再去理解下一步执行的代码.稳扎稳打才是更有效的学习方法.

## 2. 源码编译阅读流程

### 2.1. 下载spring源码

​	github源码地址: https://github.com/spring-projects/spring-framework

 可以通过直接download zip或者idea中git拉去代码.我这里拉取的是5.2.13版

###  2.2. 下载安装gradle

github地址: https://github.com/gradle/gradle

或者

官方地址: https://services.gradle.org/distributions/ 

下载zip压缩包,解压到指定目录.并在idea中配置gradle环境.

![gradle配置](/images/spring源码下载编译阅读.assets/gradle配置.png)

通过源码中的 [spring-framework](https://github.com/spring-projects/spring-framework/tree/5.2.x)/[gradle](https://github.com/spring-projects/spring-framework/tree/5.2.x/gradle)/[wrapper](https://github.com/spring-projects/spring-framework/tree/5.2.x/gradle/wrapper)/gradle-wrapper.properties 配置可以查看spring官方使用的是哪个版本的gradle.

![gradle版本.png](/images/spring源码下载编译阅读.assets/gradle版本.png)

我这里随手下了一个6.0的,但能正常构建.

### 2.3. 导入spring源码进行编译

导入Spring源码或打开源码项目.点击右侧的gradle模块里的rebulid.

![gradle_reload.png](/images/spring源码下载编译阅读.assets/gradle_reload.png)

等待将所有的依赖下载完成并构建好后.

## 3. 优化与问题

### 3.1. gradle使用阿里源加速

全局: 在USER_HOME/.gradle/下新增`init.gradle`文件, 

当前项目: 在project/.gradle/下编辑init.gradle文件,

```xml
allprojects {
    repositories {
        maven {
            name "ALIYUN_CENTRAL_URL" // name 可以不需要
            url 'https://maven.aliyun.com/nexus/content/repositories/central'
        }
        maven {
            name "ALIYUN_JCENTER_URL"
            url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'
        }
        maven {
            name "ALIYUN_GOOGLE_URL"
            url 'https://maven.aliyun.com/nexus/content/repositories/google'
        }
    }
}

```

或在项目下的build.gradle文件中添加阿里源:![gradle_加速](/images/spring源码下载编译阅读.assets/gradle_加速.png)

### 3.2. jdk.jfr 不存在

java complier中 project bytecode version 使用jdk11版本及以上

### 3.3. com.ibm.wsspi.uow不存在

build.gradle文件中添加阿里源的时候将默认源注释导致没有下载到uow包.

### 3.4 . 找不到符号 变量 CoroutinesUtils

 Error:(354, 51) java: 找不到符号

符号: 变量 CoroutinesUtils
位置: 类 org.springframework.core.ReactiveAdapterRegistry.CoroutinesRegistrar



构建的时候没有将spring-core中依赖的kotlin-coroutines构建进去.需要手动加包.

添加依赖包 该包项目中有 在源码目录项目spring-core/kotlin-coroutines/build/libs下

![image-20210330022227719](/images/spring源码下载编译阅读.assets/导入kotlin包.png)

### 3.5. Kotlin: warnings found and -Weeror specified

1. 给自己的测试项目添加kotlin依赖.

2. 在core模块下 加载cglibRepackJar 和 objenesisRepackJar