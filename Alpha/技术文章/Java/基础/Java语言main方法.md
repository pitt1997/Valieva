# main方法

## main简介

在Java中，main()方法是Java应用程序的入口方法，也就是说，程序在运行的时候，第一个执行的方法就是main()方法，这个方法和其他的方法有很大的不同，比如方法的名字必须是main，方法必须是public static void 类型的，方法必须接收一个字符串数组的参数。

下面是一个简单的Java应用程序HelloWorld，我将通过这个例子说明Java类中main()方法的奥秘，程序的代码如下：

```java
public class HelloWorld {
    public static void main(String args[]) {
        System.out.println("Hello World!");
    }
}
```



## main方法定义

```java
public static void main(String[] args) {}

public static void main(String args[]) {}

public static final void main(String[] args) {}

static public final void main(String[] args) {}

public static synchronized void main(String[] args) {}
```

 final和synchronized都可以修饰main方法。不管哪种定义，都必须保证main方法的返回值为void，并有static 和 public关键字修饰。





## main方法中可以throw Exception

```java
public class HelloWorld {
    public static void main(String args[]) throws Exception { 
        System.out.println("Hello World!");
        throw new Exception(""); 
    }
}
```

## main方法中字符串参数数组作用

main()方法中字符串参数数组作用是接收命令行输入参数的，命令行的参数之间用空格隔开。

```java
/**
* 打印main方法中的输入参数
*/
public class TestMain {
    public static void main(String args[]){
        System.out.println("打印main方法中的输入参数！");
        for(int i=0;i<args.length;i++){
            System.out.println(args[i]);
        }
    }
}
```

控制台下使用javac TestMain.java指定编译上述程序，然后使用java TestMain 1 2 3 指令运行程序，程序运行结果为：

```
1
2
3
```



