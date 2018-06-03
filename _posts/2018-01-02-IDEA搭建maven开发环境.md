---
layout: post
title: "IDEA搭建maven开发环境"
date: 2018-01-02
description: "介绍 如何在 IDEA 中配置 maven"
tags: Intellij-IDEA
---

本篇是介绍如何在 IDEA 中配置 maven 开发环境。 IDEA 本身自带了 Maven，版本一般较低，一般我选择自己安装 maven。

### 读完本篇你将：

1. 了解如何下载 maven 和 maven 的基本安装
2.  认识 maven 的几个重要配置
3.  能在 IDEA 中搭建基本的 maven 开发环境

## 一、 maven 的基本安装

### 1. 下载

[maven 官网](http://maven.apache.org/)  http://maven.apache.org/     

如果是 maven3-3.5.3 版本的话可以直接点击 我这里给出的下载链接： [http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.zip](http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.zip)    

maven 的下载有时候可能并不会那么顺利，所以我在这里还是啰嗦一下 maven 的下载。  

官网欢迎页面：   

![1528025971699](/images/posts/IDEA/1528025971699.png)

先确定自己需要下载的版本号，这里以最新版 3.5.3 为例 ：  

![1528027053508](/images/posts/IDEA/1528027053508.png)  

点击 `Download`    

![1528014704930](/images/posts/IDEA/1528014704930.png)  

进入页面，官网是推荐使用镜像下载，那么就会进入这个页面：   

![1528026736562](/images/posts/IDEA/1528026736562.png)  

点击进入镜像下载点后：  

![1528026852527](/images/posts/IDEA/1528026852527.png)  

找到 maven 项目目录：  

![1528026932499](/images/posts/IDEA/1528026932499.png)  

找到正确的版本号：  

![1528028203688](/images/posts/IDEA/1528028203688.png)  



### 2. 存储位置

将下载下来的 zip 包解压后放到一个固定的目录，路径最好不要有中文或者空格，比如和 java 放到一起，这样方便管理。  解压后的 maven 目录结构如图所示：  

![1528028520398](/images/posts/IDEA/1528028520398.png)  

### 3. 环境变量

 1. **配置 MAVEN_HOME 系统环境变量** 

    这一步类似于 JAVA_HOME 的配置，其路径值也只需指向文件夹所在位置即可，比如：  

    ![1528028708343](/images/posts/IDEA/1528028708343.png)  

    需要注意的是指定到目录就行了，不用加其他东西。

 2.  **配置 path 系统环境变量**   
  这个配置也和 Java 中的 path 环境变量类似，基于 MAVEN_HOME 配置，在path变量中添加一条环境变量比如：
```
   path: %JAVA_HOME%\bin; %MAVEN_HOME%\bin;
```   

### 4. 新版本升级

1. **新版本升级的配置**

   如果要升级新版本，只需要修改 MAVEN_HOME 将其指向新版本所在的正确物理路径就可以了。这就是 MAVEN_HOME 存在真正的意义了。

2. **测试**

   重启 cmd ，运行 mvn -v 如果出现对应的版本号提示，则代表配置成功了。

   

## 二、 settings.xml

maven 配置成功之后，为了方便管理，一般还会修改一下 maven 的配置文件，比如设置本地库存储位置等。  

**settings.xml 文件在解压目录后的 conf 文件夹下面**  

### 1. 配置本地库 localRepository

在 maven 的 settings.xml，文件中 localRepository 定义的是**本地存放依赖包的位置**。 该路径可以自由指定

```xml
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository 
  <localRepository>/path/to/local/repo</localRepository>
-->
```

将注释去掉然后填写自己指定的路径就可以了。默认是在用户目录下的 `~/.m2/repository` 。

> 提醒：如果 Intellij Idea 中自定义了 Maven，务必确保“Local repository”与此处配置的 localRepository 相同，以方便统一管理。 

<br>
### 2. 配置 mirror 镜像

还有一个比较重要的配置就是镜像地址了，由于默认是国外的网站，难免导致网络不畅，所以还是非常有必要替换的。如果在国外的话就没必要配置了。   

国内比较流畅的服务器推荐 阿里云镜像：  

```xml
<mirrors>
	<mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>  
</mirrors>   
```

镜像只能配置一个，多配制是没有用的，所以这里如果要配置的话就把其他的注释掉。配置了镜像之后，所有的 依赖包都会通过阿里云的镜像站点下载。亲生体会是阿里云镜像服务不论是速度还是稳定性都是比较满意的。  

### 3. 小脚本分享

由于种种原因我们不能保证每次使用 maven 的时候都是网络顺畅的，如果有时候正在构建项目的时候网络连接出问题的话，那么很有可能下载的依赖包会被一些其他的缓存文件替代掉。

一般缓存文件的名字都是以 `lastUpdated` 结尾的。但是这时候真正的依赖包又没有被下载下来，运行项目的时候就会报错了。不过这种情况多出现在 Eclipse 和 MyEclipse 中。所以这里分享个小脚本，如果遇到环境问题的话，使用这个脚本快速的排除是这种原因引起的问题。  

脚本代码如下：  

```bat
@echo off
rem create by sunhao(sunhao.java@gmail.com)
rem crazy coder
  
rem 这里写你的仓库路径
set REPOSITORY_PATH=C:\develop\Java\Apache\repository
rem 正在搜索...
for /f "delims=" %%i in ('dir /b /s "%REPOSITORY_PATH%\*lastUpdated*"') do (
    del /s /q "%%i"
)
rem 搜索完毕
pause
```

使用方式：  

1. 新建一个文本，将上面的代码复制进去，修改 `REPOSITORY_PATH` 为你自己的本地库路径
2. 保存文本，重命名为 clean.bat 
3. 点击运行

## 三、 在 IDEA 中配置 maven

在 IDEA 中的配置还是和在 eclipse 中的配置有比较大的区别的。接下来详细介绍几种配置方法。

### 1. 基础配置

配置的地方如图所示：   

![1528031013257](/images/posts/IDEA/1528031013257.png)  

图中指出来的地方是需要配置IDE地方：  

- **Maven home directory** ：  你的 maven 安装的文件夹路径
- **User settings  file** ：你的自定义配置文件，就是之前配置的 `settings.xml` 文件
- **Local repository** ：配置本地仓库的存放地址，务必确保该地址与你 `settings.xml` 中配置的 localRepository 地址相同

这里下面两步需要把右边的 `Override` 勾上。

### 2. 运行时配置（Runner）

上面只是配置了 maven 的基本配置，还需要配置一下运行时配置，这里也是非常关键的。如图：  

![1528031916430](/images/posts/IDEA/1528031916430.png)  

点击 maven 配置的 Runner 菜单，图中标明的地方解释如下：  

- **VM Options**： archetypeCatalog=internal ，这样在创建相关组件模板时就只基于本地已有的组件来创建，不再因网络延时而导致异常。archetype 是新建 maven 项目或者模块时的模板选择。

  archetypeCatalog 的值有三种：

  - internal——maven-archetype-plugin内置的
  - local——本地的，位置为~/.m2/archetype-catalog.xml
  - remote——指向Maven中央仓库的Catalog

- **JRE**  选择你自己指定的 JRE

- **Skip tests**  ：这个选项建议勾上，如果不勾选的话，maven 项目运行的时候会先将测试类运行一遍。

### 3. 一劳永逸配置

上面的配置方法都只是在本项目中生效，如果另外新建了一个项目的话还是得从新配置的，下面介绍一下什么是一劳永逸配置，话不多说，看图说话：  

![1528032525874](/images/posts/IDEA/1528032525874.png)  

入口在 File --> Other Settings --> Default Settings   

这样点进去之后和之前的配置基本是一样的，这样每次新建 maven 项目时就都是默认配置啦！



## 总结

IDEA 中 maven 的配置经过上面的步骤，配置的算是非常有条理了。可以愉快的敲代码了 :smile: 。  

这里再强调一下，务必要将本地库 Local Repository 的配置统一起来，避免重复下载依赖。



<br/>

转载请注明: [雷聪的博客](https://allenleic.github.io) » [{{ page.title }}](https://allenleic.github.io/2018/01/{{ page.title }})
