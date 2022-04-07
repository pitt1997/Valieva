# IntelliJ IDEA常用手册

## 简介

IDEA 全称 IntelliJ IDEA，是Java语言开发的集成环境，IntelliJ在业界被公认为最好的Java开发工具之一。

## 说明

搜索手册中的关键词寻找问题解决。手册包含IDEA快捷docker、k8s等。具体链接如下：https://www.w3cschool.cn/intellij_idea_doc/

## IntelliJ IDEA 介绍

- IntelliJ IDEA 主张一个工作空间 （在 IDEA 中叫 project）就写一个项目，这样我们的工作空间就跟着项目走，不像 eclipse 中把项目和工作空间分开，难于管理。
- 使用 IntelliJ IDEA 能够让我们开发者看清楚集成开发环境到底为我们开发者做了一些什么事情， eclipse 看起来足够强大，但它却对我们开发者施展了障眼法，这样会导致两个结果：（1）eclipse 会执行很多我们不想让它执行的事情；（2）有时 eclipse 不会帮我们做有用的事情，让我们无法轻松驾驭它。
- IntelliJ IDEA 集成的 tomcat 的功能能很方便地帮助我们实现热部署，我们还可以有选择地重新编译和加载部分字节码文件，再加上 IDEA 强大的 DEBUG 功能，可以很方便地帮助我们进行代码的调试工作。 
- IntelliJ IDEA的智能补全功能强大、快捷键功能强大，极大方便了我们的开发。

## IntelliJ IDEA 小技巧

教你简单学习IntelliJ IDEA 的快捷键，办法如下：

1. Alt菜单与鼠标右键，执行的时候，自然会提示快捷键。
2. **Help | Default Keymap Reference**，这个是一个大的常用快捷键表，建议有空的时候，花点时间过一遍。
3. **Help | Find Action (Ctrl+Shift+A)**，这个快捷键非常有用，是一个命令查找，在任何时间，都可以执行此命令，输入你需要的操作，例如”extract method”，下面会出现命令以及对应的快捷键
4. 这就要求你熟悉操作的英文名， 在Eclipse里面可能习惯了肌肉记忆，不太记得快捷健的英文名了。大不了去Eclipse里面找找，或者翻IDEA的Keymap表，有分类目录。

## IntelliJ IDEA 插件

IDEA Kubernates插件

https://www.w3cschool.cn/intellij_idea_doc/kubernetes.html

IntelliJ IDEA使用Docker

https://www.w3cschool.cn/intellij_idea_doc/intellij_idea_doc-jeug2r97.html

Alibaba Cloud AI Coding Assistant



## IntelliJ IDEA 的快捷键

<img src="images\image-20220404224829508.png" alt="images\image-20220404224829508.png" style="zoom:120%;" />

## IntelliJ IDEA 修改中文字体

设置有两种方式，也可以两次shift快捷键然后直接搜索 "Color Scheme Font"或者 "font" 然后进行设置。

方式一：Editor > font 里面设置默认字体 

方式二：Editor > Color Scheme > Color Scheme Font 里面设置自己的字体，例如SimSun - 宋体。



## IntelliJ IDEA Debug调试技巧

[学会这些 IDEA Debug调试技巧！提升开发效率10倍！ (qq.com)](https://mp.weixin.qq.com/s/9_dVKgnSsbNJBFikZifeyw)

[实战篇：9 个让你快速提效的 IDEA DEBUG 技巧 (qq.com)](https://mp.weixin.qq.com/s/v9_X5vKmvbo0_50uG2k_UA)

[最好的IDEA debug长文？看完我佛了 (qq.com)](https://mp.weixin.qq.com/s/cMMEQRZmLXfeqGvoii9gqA)

[起飞，会了这4个 Intellij IDEA 调试魔法，阅读源码都简单了！ (qq.com)](https://mp.weixin.qq.com/s/KG0yzb_9XhhTSzjHr4DkIQ)



## IntelliJ IDEA 方法和类注释模板

### 创建类的注释模板

1、创建类的注释模板

```
File > Settings > Editor > File and Code Templates > Class
```

2、自定义设置注释模板

```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
/**
 * @author author
 * @date ${YEAR}-${MONTH}-${DAY}
 * @description 
 */
public class ${NAME} {
}
```

3、勾选Enable Live Templates

<img src="images\image-20220407135312863.png" alt="image-20220407135312863" style="zoom:120%;" />

### 快捷键创建类注释模板

1、类注释

```
File > Settings > Editor > Live Templates > Class
```

可以通过直接快捷键检索

2、Live Templates 下新建 Template Group 自定义命名userDefine

<img src="images\image-20220407135559630.png" alt="image-20220407135559630" style="zoom:120%;" />

3、然后选中userDefine之后在右侧添加Live Template

<img src="images\image-20220407140141651.png" alt="image-20220407140141651" style="zoom:120%;" />

4、设置模板

Abbreviation

```
/*
```

Template text

```
/**
 * @author ljs
 * @date $DATE$
 * @description 
 */
```

<img src="images\image-20220407140507500.png" alt="image-20220407140507500" style="zoom:120%;" />

5、Edit variables 设置Date变量 date("yyyy-MM-dd")

<img src="images\image-20220407140636549.png" alt="image-20220407140636549" style="zoom:120%;" />

6、测试

类定义开头输入/*然后直接Tab

```
/*
public class TestMain {
...
```

执行之后效果自动补齐该类注释模板

```
/**
 * @author ljs
 * @date 2022-04-07
 * @description 
 */
public class TestMain {
...
```

### 快捷创建方法注释模板

https://www.jianshu.com/p/7786edabb2b6



## IntelliJ IDEA 卡掉后Tomat启动失败

1.查看该端口被哪些进程占用

netstat -ano|findstr [端口号]  

2.查看进程信息

tasklist | findstr [pid]  

3.根据进程ID或进程名称杀进程

taskkill /f /pid [pid]

taskkill /f /im [进程名]



## IntelliJ IDEA "java: 无效的目标发行版: 8"

IntelliJ IDEA编译器报错 java: 无效的目标发行版: 8

Idea ->  File | Settings | Build, Execution, Deployment  -> java compiler



## IntelliJ IDEA 解决占用C盘问题

使用idea无法启动项目，错误如下：

```
Low disk space on a IntelliJ IDEA system directory partition
```

解决办法：

```
找到安装路径下有个属性文件idea.properties（比如：D:\IntelliJ IDEA\bin）
 
修改成你要设置的路径
 
#---------------------------------------------------------------------
# Uncomment this option if you want to customize path to IDE config folder. Make sure you're using forward slashes
#---------------------------------------------------------------------
idea.config.path=D:/IntelliJ IDEA/.IntelliJIdea12/config
 
#---------------------------------------------------------------------
# Uncomment this option if you want to customize path to IDE system folder. Make sure you're using forward slashes
#---------------------------------------------------------------------
idea.system.path=D:/IntelliJ IDEA/.IntelliJIdea12/system
 
#---------------------------------------------------------------------
# Uncomment this option if you want to customize path to user installed plugins folder. Make sure you're using forward slashes
#---------------------------------------------------------------------
idea.plugins.path=D:/IntelliJ IDEA/.IntelliJIdea12/config/plugins
 
#---------------------------------------------------------------------
# Uncomment this option if you want to customize path to IDE logs folder. Make sure you're using forward slashes.
#---------------------------------------------------------------------
idea.log.path=D:/IntelliJ IDEA/.IntelliJIdea12/system/log
```

