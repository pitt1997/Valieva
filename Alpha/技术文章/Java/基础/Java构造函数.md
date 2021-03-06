# Java构造函数

## 什么是构造函数

构造函数是Java的特殊函数，用来在对象实例化的时候初始化对象的成员变量。在Java中，构造函数具有以下特点：

1、构造函数必须与类的名字相同，并且不能有返回值（返回值也不能为void）

2、每个类文件可以有一个或者多个构造函数。如果开发人员没有指定构造函数时，编译器在把源代码编译成字节码的时候会提供一个无参默认构造函数，但是该构造函数不会执行任何代码，如果开发人与已经提供构造函数，那么编译器就不会再创建默认的构造函数。

3、构造函数总是伴随着new操作一起进行调用的，且不能由程序的编写者直接调用，必须由系统调用。构造函数在对象实例化时会被自动调用，且只运行一次。

4、构造函数的主要作用就是完成对象的初始化工作。

5、构造函数不能被继承，因此，它不能被重写，但是构造函数可以被重载。

6、子类可以通过super关键字类显示调用父类的构造函数。

7、当父类和子类都没有定义构造函数时，编译器会为父类生成一个默认的无参的构造函数，给子类也生成一个无参构造函数。