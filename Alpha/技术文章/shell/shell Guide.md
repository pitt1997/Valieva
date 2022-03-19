# shell Guide



shell 指南。



## shell调用自动执行输入y

Java后台需要开启和关闭防火墙，但是开启防火墙的时候出现询问句，要求填写y或n ，Java不知道怎么后续输入y，所以必须要自动执行y

```
// 询问句加上echo y | 命令行
onlyCallCmd("echo y | bucardo remove all tables");
```

java样例 eg.

```java
    /**
     * shell命令执行要求填写y或n  Java不知道怎么后续输入y，所以必须要自动执行y
     */
    public static String onlyCallCmd(String cmd) throws IOException, InterruptedException {
        // 使用Runtime来执行command，生成Process对象
        String[] command = {"/bin/sh", "-c", cmd};
        Process process = Runtime.getRuntime().exec(command);
        return getCallResult(process);
    }

    private static String getCallResult(Process process) throws IOException, InterruptedException {
        int exitCode = process.waitFor();
        // 取得命令结果的输出流
        InputStream is = process.getInputStream();
        // 用一个读输出流类去读
        InputStreamReader isr = new InputStreamReader(is);
        // 用缓冲器读行
        BufferedReader br = new BufferedReader(isr);
        String line = null;
        StringBuilder sb = new StringBuilder();
        while ((line = br.readLine()) != null) {
            sb.append(line).append("\n");
        }
        is.close();
        isr.close();
        br.close();
        return sb.toString();
    }
```



## shell查看指定文件夹下面文件大小

```
ll -h /data/4a/backup/
```



## shell常用参数变量

 [ -n 参数 ] 可以用来判断该参数是否已被赋值

```bash
!#/bin/bash
#false
#判断的是a这个参数，因为没赋值，所以返回flase
if [ -n "$a" ] 
then
    echo true
else
    echo false
fi
#true
#判断的是“-n $a”这个字符串，此时非空即为true
#可以使用[[ -n $a ]] 来达到上面案例的效果
if [ -n $a ] 
then
    echo true
else
    echo false
fi
#true
#字符串，理由同上
if [ -n a ] 
then
    echo true
else
    echo false
fi
#true
#字符串，理由同上
if [ -n  ] 
then
    echo true
else
    echo false
fi
```



```bash
使用echo $? 查看命令是否执行成功
shell中的特殊变量：
变量名
含义
$0
shell或shell脚本的名字
$*
以一对双引号给出参数列表
$@
将各个参数分别加双引号返回
$#
参数的个数
$_
代表上一个命令的最后一个参数
$$
代表所在命令的PID
$!
代表最后执行的后台命令的PID
$?
代表上一个命令执行后的退出状态

echo $? 
如果返回值是0，就是执行成功；如果是返回值是0以外的值，就是失败。
```

