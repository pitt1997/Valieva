# CAS单点登录JDBC认证

之前介绍过在cas.properties文件中，修改默认的静态用户名和密码。这节分析下，取消静态登陆方式，该用从数据库链接获取用户进行登录认证。CAS支持各种数据库，这里选用MySQL进行实践。

## 加密方案

cas支持jdbc校验方案：

- 根据sql给予用户名进行查询，根据密码字段进行鉴定（`select * from table_users where username=?`）可判断有效等
- 通过盐等手段进行编码加密再进行匹配（推荐）
- 根据sql给予用户名以及密码进行查询（`select count(x) from tables_users where username = ? and password=?`），不可判断有效期，若数量大于0则成功
- 根据用户名密码连接数据库，原理是通过jdbc，若连接成功则成功

常用单向加密算法：MD5、SHA、HMAC

一般的加密策略的三种：

- 单项加密算法(密码)
- 单向加密算法(密码+动态盐+私有盐)*加密次数（推荐）
- 不加密（不推荐）

## 一、数据库脚本

> 创建表

```sql
create table sys_user (
 `id` int(11) not null auto_increment,
 `username` varchar(30) not null,
 `password` varchar(64) not null,
 `email`    varchar(50),
 `address`  varchar(100),
 `age`      int,
 `expired` int,     // 是否过期
 `disabled` int,    // 是否禁用
 `locked` int,      // 是否锁定
  primary key (`id`)
) engine=innodb auto_increment=1 default charset=utf8;
```

未加密数据

```sql
insert into sys_user values ('6', 'tingfeng', 'tingfeng', 'admin@foxmail.com', '郑州东区', 24, 0, 0, 0);
```

明文MD5数据

```sql
/*123*/
insert into sys_user values ('1', 'admin', '202cb962ac59075b964b07152d234b70', 'admin@foxmail.com', '广州天河', 24, 0, 0, 0);
/*12345678*/
insert into sys_user values ('2', 'zhangsan', '25d55ad283aa400af464c76d713c07ad', 'zhangsan@foxmail.com', '广州越秀', 26, 0, 0, 0);
/*1234*/
/*禁用账户*/
insert into sys_user values('3', 'zhaosi','81dc9bdb52d04dc20036dbd8313ed055', 'zhaosi@foxmail.com', '广州海珠', 25, 0 , 1, 0);
/*12345*/
/*过期账户*/
insert into sys_user values('4', 'wangwu','827ccb0eea8a706c4c34a16891f84e7b', 'wangwu@foxmail.com', '广州番禺', 27, 1 , 0, 0);
/*123*/
/*锁定账户*/
insert into sys_user values('5', 'boss','202cb962ac59075b964b07152d234b70', 'boss@foxmail.com', '深圳', 30, 0 , 0, 1);
```