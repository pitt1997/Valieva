# pg_upgrade — 升级PostgreSQL服务器实例

## 说明

当前的PostgreSQL版本号由一个主要版本号和一个次要版本号组成。例如，在版本号 10.1 中，10 是主要版本号，1 是次要版本号，这意味着这将是主要版本 10 的第一个次要版本。对于PostgreSQL版本 10.0之前的版本，版本号包括三个数字，例如 9.5.3。在这些情况下，主要版本由版本号的前两位数字组组成，例如 9.5，次要版本是第三个数字，例如 3，这意味着这将是主要版本 9.5 的第三个次要版本。次要版本永远不会改变内部存储格式，并且始终与相同主要版本号的早期和更高版本的次要版本兼容。例如，10.1 版与 10.0 版和 10.6 版兼容。同样，例如，9.5.3 与 9.5.0、9.5.1 和 9.5.6 兼容。要在兼容版本之间进行更新，您只需在服务器关闭时替换可执行文件并重新启动服务器。数据目录保持不变——小升级就是这么简单。对于PostgreSQL 的主要版本，内部数据存储格式可能会发生变化，从而使升级复杂化。

## 描述

pg_upgrade（以前称为pg_migrator）允许将存储在PostgreSQL数据文件中的数据升级到更高的PostgreSQL主要版本，而无需进行主要版本升级通常需要的数据转储/重新加载，例如，从 9.5.8 到 9.6.4 或从 10.7 到 11.2 . 次要版本升级不需要它，例如从 9.6.2 到 9.6.3 或从 10.1 到 10.2。

主要的 PostgreSQL 版本会定期添加新功能，这些功能经常更改系统表的布局，但内部数据存储格式很少更改。pg_upgrade使用这个事实通过创建新的系统表和简单地重用旧的用户数据文件来执行快速升级。如果未来的主要版本以一种使旧数据格式不可读的方式更改数据存储格式，则pg_upgrade将无法用于此类升级。

pg_upgrade尽最大努力确保新旧集群是二进制兼容的，例如，通过检查兼容的编译时设置，包括 32/64 位二进制文件。重要的是，任何外部模块也是二进制兼容的，尽管pg_upgrade无法检查这一点。

pg_upgrade 支持从 8.4.X 及更高版本升级到PostgreSQL的当前主要版本，包括快照和 beta 版本。

## 用法

1、安装新版pgsql数据库并且初始化 已经执行过

```bash
$ #/usr/pgsql-10/bin/postgresql-10-setup initdb
```

2、停止旧版本的数据库

```bash
$ kill -INT `head -1 /var/lib/pgsql/9.3/data/postmaster.pid`
```

3、切换至postgres用户

```bash
$ su - postgres
```

4、pg_upgrade检查数据库版本兼容性

```bash
$ /usr/pgsql-10/bin/pg_upgrade -b /usr/pgsql-9.3/bin/ -B /usr/pgsql-10/bin/ -d /var/lib/pgsql/9.3/data/ -D /var/lib/pgsql/10/data/ -k -c
```

> pg_upgrade参数说明：
>
> `-b` 旧的 PostgreSQL 可执行目录；`-B`  新的 PostgreSQL 可执行目录；默认是pg_upgrade所在的目录；
>
> `-d` 旧的数据库集群配置目录；`-D` 新的数据库集群配置目录；
>
> `-c` 仅检查集群，不要更改任何数据 `-k` 使用硬链接而不是将文件复制到新集群

5、使用pg_upgrade普通模式升级

```bash
$ /usr/pgsql-10/bin/pg_upgrade -b /usr/pgsql-9.3/bin/ -B /usr/pgsql-10/bin/ -d /var/lib/pgsql/9.3/data/ -D /var/lib/pgsql/10/data/
```

6、升级完毕退出postgres用户

```bash
$ exit
```

7、注意pg_hba.conf

如果您修改了`pg_hba.conf`，请恢复其原始设置。可能还需要调整新集群中的其他配置文件以匹配旧集群，例如`postgresql.conf`（及其包含的任何文件）、`postgresql.auto.conf`.

8、启动postgresql10数据库

```bash
$ systemctl enable postgresql-10.service
$ systemctl start postgresql-10.service
# systemctl restart postgresql-10.service
```

9、检查当前数据库版本

```bash
$ psql --version
```

10、升级后处理

如果需要任何升级后处理，pg_upgrade 将在完成时发出警告。它还将生成必须由管理员运行的脚本文件。脚本文件将连接到需要升级后处理的每个数据库。每个脚本都应该使用：

```
psql --username=postgres --file=script.sql postgres
```

脚本可以按任何顺序运行，并且可以在运行后删除。