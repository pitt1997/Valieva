# IntelliJ IDEA中gradle配置和使用

## 前言

### gradle的优点

1. 按约定声明构建和建设； 
2. 强大的支持多工程的构建； 
3. 强大的依赖管理（基于Apache Ivy），提供最大的便利去构建工程； 
4. 全力支持已有的 Maven 或者Ivy仓库基础建设； 
5. 支持传递性依赖管理，在不需要远程仓库和pom.xml和ivy配置文件的前提下； 
6. 基于groovy脚本构建，其build脚本使用groovy语言编写； 
7. 具有广泛的领域模型支持构建； 
8. 深度 API； 
9. 易迁移； 
10. 自由和开放源码，Gradle是一个开源项目，基于 ASL 许可。

### gradle、maven、ant

ant可以自动化打包逻辑。maven也可以自动化打包，相比于ant，它多做的事是帮你下载jar包。但是maven的打包逻辑太死板，定制起来太麻烦，不如ant好用。gradle就是又能自动下jar包，又能自己写脚本，并且脚本写起来还比ant好用的这么个东西。



## IDEA配置gradle

为了使用gradle命令，idea里使用不了命令。比如编译命令gradle build。

### 下载

[gradle安装包下载地址](https://gradle.org/releases/)

下载[v7.0.1](https://gradle.org/next-steps/?version=7.0.1&format=bin)后解压到本地磁盘：E:\gradle-7.0.1

### 配置环境变量步骤

Windows打开【控制面板主页】->【高级系统设置】->【环境变量】->【系统变量】->【新建系统变量】->【变量名：**"GRADLE_HOME"**，变量值：**"E:\gradle-7.0.1"**】->【找到path变量编辑】->【新建添加变量："**%GRADLE_HOME%\bin**"】

### 测试

cmd命令里输入gradle -v如果能打出版本号，说明环境配置完毕。

### IDEA配置步骤

首先在E盘新建文件.gradle（使用dos命令mkdir .gradle ），做为gradle下载的jar包仓库主目录，默认在C:\Users\Administrator\.gradle
然后再在IDEA中打开【File】->【Settings】->【Build,Execution,Deployment】->【Build Tools】->【Gradle】

Gradle user home 修改为：**E:\gradleRepository\.gradle**（gradle的仓库位置）

Use Gradle from（gradle工具的路径）

<img src="images\image-20210801234024206.png" alt="image-20210801234024206" style="zoom:100%;" />



### gradle仓库设置

**build.gradle文件配置**

```
repositories {
    mavenLocal()
    maven { url "http://maven.aliyun.com/nexus/content/groups/public/"}
    mavenCentral()
    jcenter()
    maven { url "https://repo.spring.io/snapshot" }
    maven { url "https://repo.spring.io/milestone" }
    maven { url 'http://oss.jfrog.org/artifactory/oss-snapshot-local/' }  //转换pdf使用
}
```

存储库只是文件的集合，按分组，名称和版本来组织构造。 默认情况下，Gradle不定义任何存储库。 这里使用repositories 指定存储库。
**mavenLocal()**：指定使用maven本地仓库，而本地仓库在配置maven时setting文件指定的仓库位置。如<localRepository>D:/repository</localRepository>，同时将setting文件拷贝到C:\Users\Administrator\.m2目录下，一般该目录下是没有setting文件的，gradle查找jar包顺序如下：gradle默认会按以下顺序去查找本地的仓库：USER_HOME/.m2/settings.xml >> M2_HOME/conf/settings.xml >> USER_HOME/.m2/repository。 
maven { url "http://maven.aliyun.com/nexus/content/groups/public/"}：指定阿里云镜像加速地址 
mavenCentral()：这是Maven的中央仓库，无需配置，直接声明就可以使用 
jcenter():JCenter中央仓库，实际也是是用的maven搭建的，但相比Maven仓库更友好，通过CDN分发，并且支持https访问。 
后面的maven { url 地址}，指定maven仓库，一般用私有仓库地址或其它的第三方库 
gradle按配置顺序寻找jar文件。如果本地存在就不会再去下载。不存在的再去maven仓库下载，这里注意下载下来的jar文件不在maven仓库里，而是在gradle的主工作目录下，如上面的E:\gradleRepository\.gradle目录 

#### 构建命令

清理命令

```
gradle clean
```

构建打包命令

```
gradle clean build
```

编译时跳过测试，使用`-x`,`-x`参数用来排除不需要执行的任务

```
gradle clean build -x test
```

#### 创建缓存依赖

执行命令`gradle clean build --refresh-dependencies`或删除.gradle/caches目录，构建的时候它会下载所有依赖并加入到缓存中。

#### 阿里云镜像

```json
buildscript {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven{ url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
    }
```

#### gradle.build构建脚本

build.gradle是Gradle默认的构建脚本文件，执行Gradle命令的时候，会默认加载当前目录下的build.gradle脚本文件。 
gradle.build脚本如下：

```
buildScript {
    repositories {
         mavenCentral()
    }
}

repositories {
     mavenCentral()
}
```

buildScript里的repositories是这个脚本需要的依赖库，与项目无关，在执行脚本时，会从这个库里download对应的jar和插件。第二个repositories是项目里需要依赖的jar的库。

#### 查看项目已定义的所有task以及含义

命令

```
gradle tasks
```

比如结果如下

- assemble: 编译
- build：编译并执行测试
- clean：删除build目录
- jar： 生成jar包
- test：执行单元测试

#### maven项目转换为gradle项目

根目录执行

```
gradle init --type pom
```

上面的命令会根据pom文件自动生成gradle项目所需的文件和配置，然后以gradle项目重新导入即可。

#### settings.gradle配置

是模块Module配置文件，大多数setting.gradle的作用是为了配置子工程，根目录下的settings.gradle脚本文件是针对module的全局配置，它的作用域所包含的所有module是通过settings.gradle来配置。 
settings.gradle用于创建多Project的Gradle项目。Project在IDEA里对应Module模块。 
例如配置module名rootProject.name = 'DyoonPLM'


参考文章：https://blog.csdn.net/achenyuan/article/details/80682288