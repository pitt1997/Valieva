## 操作步骤

**1.选中你想要push的项目文件夹右键，打开Git命令行**

**2.首先使用git init 命令，如果已经存在.git目录可不再执行**

```
git init
```

**3.使用git add . 命令**

```
git add . 
```

**4.如果是远程提交其他远程仓库，复制你github上面的远程仓库地址，再使用命令git remote add origin "你的远程仓库地址"**

```
git remote add origin "你的远程仓库地址"
```

**5.使用git commit . -m "initialize" 命令**

```
git commit . -m "initialize"
```

**6.push到远程仓库**

```
git push "你的远程仓库地址" master
```

**7.git push origin master**

```
git push origin master
```

**8.强制将本地仓库内容push到远程的方法**
简单粗暴且有效！缺点就是强制push后远程文件可能会丢失

```
 git push -f origin master
```

**新版本git工程名称默认是main**

```
git push origin main
```

## 遇到问题

https://blog.csdn.net/rongxiang111/article/details/78825232

## 相关参考

https://blog.csdn.net/Brad_PiTt7/article/details/96432703?spm=1001.2014.3001.5502

