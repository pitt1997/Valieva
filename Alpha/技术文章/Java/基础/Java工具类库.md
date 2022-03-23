# Java工具类库

工作很多年后，才发现有很多工具类库，可以大大简化代码量，提升开发效率。

## Java自带工具方法

### List集合拼接成以逗号分隔的字符串

```java
// 如何把list集合拼接成以逗号分隔的字符串 a,b,c  
List<String> list = Arrays.asList("a", "b", "c");  // 第一种方法，可以用stream流  
String join = list.stream().collect(Collectors.joining(","));  
System.out.println(join); // 输出 a,b,c  
// 第二种方法，其实String也有join方法可以实现这个功能  
String join = String.join(",", list);  System.out.println(join); // 输出 a,b,c  
```

### 忽略大小写比较两个字符串是否相等

```java
if (strA.equalsIgnoreCase(strB)) {   
	System.out.println("相等");  
}  
```

### 比较两个对象是否相等

当我们用equals比较两个对象是否相等的时候，还需要对左边的对象进行判空，不然可能会报空指针异常，我们可以用java.util包下【**Objects.equals**】封装好的比较是否相等的方法

```java
Objects.equals(strA, strB);

public static boolean equals(Object a, Object b) {  
    return (a == b) || (a != null && a.equals(b));  
}  
```

### 两个List集合取交集

```java
List<String> list1 = new ArrayList<>();  
list1.add("a");  
list1.add("b");  
list1.add("c");  
List<String> list2 = new ArrayList<>();  
list2.add("a");  
list2.add("b");  
list2.add("d");  
list1.retainAll(list2);  
System.out.println(list1); // 输出[a, b]  
```

### apache commons工具类库

apache commons是最强大的，也是使用最广泛的工具类库，里面的子库非常多，下面介绍几个最常用的

建议使用**commons-lang3**，优化了一些api，原来的commons-lang已停止更新

```xml
<dependency>      
	<groupId>org.apache.commons</groupId>      
	<artifactId>commons-lang3</artifactId>      
	<version>3.12.0</version>  
</dependency>  
```

#### 字符串判空

传参CharSequence类型是String、StringBuilder、StringBuffer的父类，都可以直接下面方法判空，以下是源码：

```java
public static boolean isEmpty(final CharSequence cs) {      
	return cs == null || cs.length() == 0;  
}    

public static boolean isNotEmpty(final CharSequence cs) {      
	return !isEmpty(cs);  
}    

// 判空的时候，会去除字符串中的空白字符，比如空格、换行、制表符  
public static boolean isBlank(final CharSequence cs) {      
	final int strLen = length(cs);      
	
    if (strLen == 0) {          
		return true;      
	}      
    
	for (int i = 0; i < strLen; i++) {          
		if (!Character.isWhitespace(cs.charAt(i))) {              
			return false;          
		}      
	}     
    
	return true;  
}    

public static boolean isNotBlank(final CharSequence cs) {      
	return !isBlank(cs);  
}  
```

#### 首字母转成大写

```java
String str = "yideng";  
String capitalize = StringUtils.capitalize(str);  
System.out.println(capitalize); // 输出Yideng  
```

#### 重复拼接字符串

```java
String str = StringUtils.repeat("ab", 2);  
System.out.println(str); // 输出abab  
```

#### 格式化日期

再也不用手写SimpleDateFormat格式化了~

**Date类型转String类型** 

```java
String date = DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss"); 
System.out.println(date); // 输出 2021-05-01 01:01:01
```

**String类型转Date类型**  

```java
Date date = DateUtils.parseDate("2021-05-01 01:01:01", "yyyy-MM-dd HH:mm:ss");    

```

**计算一个小时后的日期** 

```java
Date date = DateUtils.addHours(new Date(), 1);  
```

#### 包装临时对象

当一个方法需要返回两个及以上字段时，我们一般会封装成一个临时对象返回，现在有了Pair和Triple就不需要了

```java
// 返回两个字段  
ImmutablePair<Integer, String> pair = ImmutablePair.of(1, "yideng"); System.out.println(pair.getLeft() + "," + pair.getRight()); // 输出 1,yideng  
// 返回三个字段  
ImmutableTriple<Integer, String, Date> triple = ImmutableTriple.of(1, "yideng", new Date());  
System.out.println(triple.getLeft() + "," + triple.getMiddle() + "," + triple.
```

### commons-collections 集合工具类

```xml
<dependency>      
	<groupId>org.apache.commons</groupId>      
	<artifactId>commons-collections4</artifactId>      
	<version>4.4</version>  
</dependency>  
```

#### 集合判空

```java
public static boolean isEmpty(final Collection<?> coll) {      
	return coll == null || coll.isEmpty();  
}    

public static boolean isNotEmpty(final Collection<?> coll) {      
	return !isEmpty(coll);  
}  
```

#### 两个集合交集

```java
Collection<String> collection = CollectionUtils.retainAll(listA, listB);  
```

#### 两个集合取并集  

```
Collection<String> collection = CollectionUtils.union(listA, listB);  
```

#### 两个集合取差集  

```
Collection<String> collection = CollectionUtils.subtract(listA, listB); 
```

### common-beanutils 操作对象

```xml
<dependency>      
	<groupId>commons-beanutils</groupId>      
	<artifactId>commons-beanutils</artifactId>      
	<version>1.9.4</version>  
</dependency> 
```

```java
User user = new User();  
BeanUtils.setProperty(user, "id", 1);  
BeanUtils.setProperty(user, "name", "yideng"); System.out.println(BeanUtils.getProperty(user, "name")); // 输出 yideng  
System.out.println(user); // 输出 {"id":1,"name":"yideng"}  
```

```java
Map<String, String> map = BeanUtils.describe(user);  
System.out.println(map); // 输出 {"id":"1","name":"yideng"}  
// map转对象  
User newUser = new User();  
BeanUtils.populate(newUser, map);  
System.out.println(newUser); // 输出 {"id":1,"name":"yideng"}  
```

### commons-io 文件流处理

```xml
<dependency>      
	<groupId>commons-io</groupId>      
	<artifactId>commons-io</artifactId>      
	<version>2.8.0</version> 
</dependency> 
```

```java
File file = new File("demo1.txt");  
// 读取文件  
List<String> lines = FileUtils.readLines(file, Charset.defaultCharset());  
// 写入文件  
FileUtils.writeLines(new File("demo2.txt"), lines);  
// 复制文件  
FileUtils.copyFile(srcFile, destFile);  
```



### 废弃的方法注解

```
/** @deprecated */
```

### 枚举类型

```java
public enum HttpStatus {
    CONTINUE(100, "Continue"),
    SWITCHING_PROTOCOLS(101, "Switching Protocols"),
    PROCESSING(102, "Processing"),
    CHECKPOINT(103, "Checkpoint"),
    OK(200, "OK"),
    CREATED(201, "Created"),
    ACCEPTED(202, "Accepted"),
    NON_AUTHORITATIVE_INFORMATION(203, "Non-Authoritative Information"),
    NO_CONTENT(204, "No Content"),
    RESET_CONTENT(205, "Reset Content"),
    PARTIAL_CONTENT(206, "Partial Content"),
    MULTI_STATUS(207, "Multi-Status"),
    ALREADY_REPORTED(208, "Already Reported"),
    IM_USED(226, "IM Used"),
    MULTIPLE_CHOICES(300, "Multiple Choices"),
    MOVED_PERMANENTLY(301, "Moved Permanently"),
    FOUND(302, "Found"),
    /** @deprecated */
    @Deprecated
    MOVED_TEMPORARILY(302, "Moved Temporarily")
    private final int value;
    private final String reasonPhrase;

    private HttpStatus(int value, String reasonPhrase) {
        this.value = value;
        this.reasonPhrase = reasonPhrase;
    }

    public int value() {
        return this.value;
    }

    public String getReasonPhrase() {
        return this.reasonPhrase;
    }
}
```

### 策略模式

接下来就看看如何用策略模式进行重构：

```java
public interface ShareStrategy {
    // 定义分享策略执行方法
    void shareAlgorithm(String param);
}

public class OrderItemShare implements ShareStrategy {
    @Override
    public void shareAlgorithm(String param) {
        System.out.println("当前分享图片是" + param);
    }
}

// 省略 MultiItemShare以及SingleItemShare策略

// 分享工厂
public class ShareFactory {
  // 定义策略枚举
    enum ShareType {
        SINGLE("single", "单商品"),
        MULTI("multi", "多商品"),
        ORDER("order", "下单");
        // 场景对应的编码
        private String code;
       
        // 业务场景描述
        private String desc;
        ShareType(String code, String desc) {
            this.code = code;
            this.desc = desc;
        }
        public String getCode() {
            return code;
        }
       // 省略 get set 方法
    }
  	// 定义策略map缓存
    private static final Map<String, ShareStrategy> shareStrategies = new       HashMap<>();
    static {
        shareStrategies.put("order", new OrderItemShare());
        shareStrategies.put("single", new SingleItemShare());
        shareStrategies.put("multi", new MultiItemShare());
    }
    // 获取指定策略
    public static ShareStrategy getShareStrategy(String type) {
        if (type == null || type.isEmpty()) {
            throw new IllegalArgumentException("type should not be empty.");
        }
        return shareStrategies.get(type);
    }
 
    public static void main(String[] args) {
        // 测试demo
        String shareType = "order";
        ShareStrategy shareStrategy = ShareFactory.getShareStrategy(shareType);
        shareStrategy.shareAlgorithm("order");
        // 输出结果：当前分享图片是order
    }
}
```
