# Java中的clone方法

在程序开发中，有时可能好会遇到下列情况：已经存在一个对象A，现在需要一个与对象A完全相同的B对象，并对B对象的值进行修改，但是A对象的原有的属性值不能改变。这时，如果使用java提供的对象赋值语句，当修改B对象值后，A对象的值也会被修改。那么应该如何实现创建一个和对象A完全相同的对象B，而且修改对象B时，对象A的属性值不被改变呢？

要实现这一功能，可以使用Object类中的clone方法。clone方法可以完成对象的浅克隆。所谓浅克隆就是说被克隆的对象的各个属性都是基本类型，而不是引用类型（接口、类、数组），如果存在引用类型的属性，则需要进行深克隆。





深克隆

要想实现 Address的深克隆，**首先让Address类实现 Cloneable 接口，重写clone方法**。

public class Address **implements Cloneable**{undefined







https://blog.csdn.net/duyiwuerluozhixiang/article/details/86129217