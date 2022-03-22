# Java中的clone方法

在程序开发中，有时可能好会遇到下列情况：已经存在一个对象A，现在需要一个与对象A完全相同的B对象，并对B对象的值进行修改，但是A对象的原有的属性值不能改变。这时，如果使用Java提供的对象赋值语句，当修改B对象值后，A对象的值也会被修改。那么应该如何实现创建一个和对象A完全相同的对象B，而且修改对象B时，对象A的属性值不被改变呢？

可以使用Java的clone方法针对对象进行拷贝。要实现这一功能，可以使用Object类中的clone方法。clone方法可以完成对象的**浅克隆**。所谓浅克隆就是说被克隆的对象的各个属性都是基本类型，而不是引用类型（接口、类、数组），如果存在引用类型的属性，则需要进行深克隆。

## Object clone()  - 浅拷贝

Object clone() 方法用于创建并返回一个对象的拷贝。clone 方法是**浅拷贝**，**对象内属性引用的对象只会拷贝引用地址，而不会将引用的对象重新分配内存**，相对应的**深拷贝**则会连引用的对象也重新创建。

### 语法

```java
object.clone()
```

### 返回值

返回一个对象的拷贝。**由于 Object 本身没有实现 Cloneable 接口，所以不重写 clone 方法并且进行调用的话会发生 CloneNotSupportedException 异常。**

### 实例

```java
public class Person implements Cloneable {
	
	private int age ;
	private String name;
	
	public Person(int age, String name) {
		this.age = age;
		this.name = name;
	}
	
	public Person() {}
 
	public int getAge() {
		return age;
	}
 
	public String getName() {
		return name;
	}
	
	@Override
	protected Object clone() throws CloneNotSupportedException {
		return (Person)super.clone();
	}
}
```

## 覆盖Object中的clone方法 - 深拷贝

继承接口 implements Cloneable
```java
static class Body implements Cloneable {
	public Head head;
	public Body() {}
	public Body(Head head) {this.head = head;}
	@Override
	protected Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}

static class Head {
	public Face face;
	public Head() {}
	public Head(Face face){this.face = face;}
} 

public static void main(String[] args) throws CloneNotSupportedException {
	Body body = new Body(new Head());
	Body body1 = (Body) body.clone();
	System.out.println("body == body1 : " + (body == body1) );
	System.out.println("body.head == body1.head : " +  (body.head == body1.head));
}
```

以上代码中， 有两个主要的类， 分别为Body和Face， 在Body类中， 组合了一个Face对象。当对Body对象进行clone时， 它组合的Face对象只进行浅拷贝。打印结果可以验证该结论：

```
body == body1 : false
body.head == body1.head : true
```

**如果要使Body对象在clone时进行深拷贝， 那么就要在Body的clone方法中，将源对象引用的Head对象也clone一份。**

```java
    static class Body implements Cloneable {
		public Head head;
		public Body() {}
		public Body(Head head) {this.head = head;}
 
		@Override
		protected Object clone() throws CloneNotSupportedException {
			Body newBody =  (Body) super.clone();
			newBody.head = (Head) head.clone();
			return newBody;
		}
	}

	static class Head implements Cloneable {
		public  Face face;
		
		public Head() {}
		public Head(Face face){this.face = face;}
		@Override
		protected Object clone() throws CloneNotSupportedException {
			return super.clone();
		}
	} 

	public static void main(String[] args) throws CloneNotSupportedException {
		Body body = new Body(new Head());
		Body body1 = (Body) body.clone();
		System.out.println("body == body1 : " + (body == body1) );
		System.out.println("body.head == body1.head : " +  (body.head == body1.head));
        System.out.println("body.head.face == body1.head.face : " +  (body.head.face == body1.head.face));
	}
```

打印结果为：

```
body == body1 : false
body.head == body1.head : false
body.head.face == body1.head.face : true
```

那么，对Body对象来说，算是这算是深拷贝吗？其实应该算是深拷贝，因为对Body对象内所引用的其他对象（目前只有Head）都进行了拷贝，也就是说两个独立的Body对象内的head引用已经指向了独立的两个Head对象。但是，这对于两个Head对象来说，他们指向了同一个Face对象，这就说明，两个Body对象还是有一定的联系，并没有完全的独立。这应该说是一种不彻底的深拷贝。





## 深拷贝和浅拷贝区别

**浅拷贝：被复制对象的所有变量都含有与原来对象相同的值，而所有对其他对象的引用仍然指向原来的对象。换言之，浅拷贝只是复制所考虑的对象，而不是复制它所引用的对象。**

**深拷贝：被复制的对象的所有变量都含有与原来对象相同的值，除去那些引用其他对象的变量。那些引用其他对象的变量将指向被复制的新对象。那些引用其他对象的变量将指向被复制的新对象，而不再是原有的那些被引用的对象。换言之，深拷贝把复制的对象所有引用的对象都复制了一遍。**

<img src="images\image-20220322003309220.png" alt="image-20220322003309220" style="zoom:120%;" />

## 相关参考

https://blog.csdn.net/duyiwuerluozhixiang/article/details/86129217

https://www.runoob.com/java/java-object-clone.html

https://blog.csdn.net/zhangjg_blog/article/details/18369201





