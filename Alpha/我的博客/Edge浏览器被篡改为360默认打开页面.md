# Edge浏览器被篡改为360默认打开页面

Edge浏览器被篡改为360默认打开页面

## 解决办法

1、打开注册表

Win+R  输入 regedit 

2、找到Start Page

具体路径如下：

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\Main
```

3、将Start Page修改为正常网站

```
https://cn.bing.com/
```

