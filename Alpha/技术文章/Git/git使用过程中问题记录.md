



# README

git使用过程中的问题记录。



## Failed to connect to github.com port 443: Timed out

### 问题描述

提交git或者clone时候出现443错误问题，Git - Failed to connect to github.com port 443: Timed out

<img src="images\image-20220316231830669.png" alt="image-20220316231830669" style="zoom: 200%;" />



浏览器访问https://www.ipaddress.com/网址查询下面所需的地址对应的IP，输入hostname或domain查询，比如查询`github.com`的IP

<img src="images\image-20220316232115186.png" alt="image-20220316232115186" style="zoom:120%;" />

查询出github的IP地址之后**修改hosts文件**映射，映射文件位置 C:\Windows\System32\drivers\etc\hosts 

```
140.82.114.4 github.com
```

添加以上配置项后**刷新WindowsDNS缓存**

cmd命令窗口执行以下命令

```
ipconfig /flushdns
```

配置完成之后尝试再次操作git命令观察是否还是有443问题。



如果你的电脑正在使用**VPN或者相关代理**，并且其他解决方案都无法解决您的问题，那么你可以尝试以下方法进行解决，例如我的主机对 HTTP、HTTPS、SOCK 使用 VPN 代理，我的本地代理端点是 127.0.0.1:4780。

<img src="images\image-20220317223518168.png" alt="image-20220317223518168" style="zoom:80%;" />

### 具体步骤

1、请尝试直接编辑 .git目录下面的config 文件。

<img src="images\image-20220317223225026.png" alt="image-20220317223225026" style="zoom:99%;" />

2、.git/config 文件文件下[http]、[https]配置项添加如下内容

```
[http]
	sslVerify = false
	proxy = http://127.0.0.1:11223
[https]
    proxy = http://127.0.0.1:11223	
```

3、设置完毕后正常提交git

### 相关参考

[参考链接](https://stackoverflow.com/questions/48987512/ssl-connect-ssl-error-syscall-in-connection-to-github-com443)

[443Timed out问题处理](https://blog.csdn.net/hzw2017/article/details/115409516?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1.pc_relevant_default&utm_relevant_index=2)

[网上取消代理处理443问题](https://blog.csdn.net/Hodors/article/details/103226958)



## error: failed to push some refs to xxx

### 问题描述

本地初始化 ，并关联新建的github地址，在 pull 的时候发现报错，error: src refspec xxx does not match any error...

<img src="images\image-20220317224818056.png" alt="image-20220317224818056" style="zoom:150%;" />

### 问题分析

估计是由于仓库名称不一样，导致远程和本地的仓库不能关联上

发现现在建的 github 工程默认名为了 main，后面发现由于受到"Black Lives Matter"运动的影响，GitHub 从今年 10 月 1 日起，在该平台上创建的所有新的源代码仓库将默认被命名为 “main”，而不是原先的"master"，所以 pull 和 push 都会报错。

旧版本

<img src="images\image-20220317225050563.png" alt="image-20220317225050563" style="zoom:150%;" />

新版本

<img src="images\image-20220317225115597.png" alt="image-20220317225115597" style="zoom:150%;" />



### 问题解决

所以我们在提交push的时候使用命令 git push origin main ， 而不是使用 git push origin master。

```
git push origin main
```



## remote: Support for password authentication was removed on August 13, 2021.

### 问题现象

push时用户名和密码都输入正确之后仍然无法push成功。

```
$ git push origin main
Username for 'https://github.com': pitt1997
remote: Support for password authentication was removed on August 13, 2021. Please use a personal acces                              s token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/                               for more information.
fatal: Authentication failed for 'https://github.com/pitt1997/Valieva.git/'
```

这是什么情况，大概意思就是你原先的密码凭证从2021年8月13日开始就不能用了，必须使用个人访问令牌（personal access token），就是把你的密码替换成token！

具体步骤

[官方文档手册](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

1、单击您的个人资料照片，然后单击**设置**。

<img src="images\image-20220317231144535.png" alt="image-20220317231144535" style="zoom:150%;" />



2、在左侧边栏中，单击**Developer settings**。

3、选择个人访问令牌Personal access tokens，然后选中生成令牌Generate new token

<img src="images\image-20220317231350429.png" alt="image-20220317231350429" style="zoom:150%;" />

4、设置token的有效期，访问权限等

选择要授予此令牌token的范围或权限。

要使用token从命令行访问仓库，请选择repo。
要使用token从命令行删除仓库，请选择delete_repo
其他根据需要进行勾选

<img src="images\image-20220317231528495.png" alt="image-20220317231528495" style="zoom:150%;" />




5、生成令牌Generate token

记得把你的token保存下来，因为你再次刷新网页的时候，你已经没有办法看到它了，所以我还没有彻底搞清楚这个token的使用，后续还会继续探索！

6、之后用自己生成的token登录，把上面生成的token粘贴到输入密码的位置，然后成功push代码！



### 相关链接

[git密码问题处理](https://blog.csdn.net/weixin_41010198/article/details/119698015)



## github 500问题

<img src="images\image-20220317231442589.png" alt="image-20220317231442589" style="zoom:150%;" />

TODO...