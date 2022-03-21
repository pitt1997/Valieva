

# PostgreSQL数据库 - README

PostgreSQL相关记录。

## 数据库切换用户

```shell
su postgres
bash-4.2$ psql -d basicframe
\q 退出pgsql命令行
exit退出bash
```

## 连接数据库切换数据库

```shell
psql -U user -d dbname
\c dbname 切换数据库 相当于mysql的use dbname
```

## 列举数据库

```shell
\l 列举数据库，相当于mysql的show databases
```

## 列举表

```
\dt 列举表，相当于show tables
```

## 查看表结构

```
\d tblname 查看表结构，相当于desc tblname,show columns from tbname
```

## PostgreSQL递归查询SQL

```xml
<select id="getOrgParentById" resultMap="BaseResultMap" parameterType="java.lang.String">
    WITH RECURSIVE r AS (
    SELECT t.* FROM t_acc_org t WHERE t.id= #{id} and t.status = 'normal'
    union ALL
    SELECT t.* FROM t_acc_org t, r WHERE t.id = r.pid and t.status = 'normal') SELECT * FROM r ORDER BY path
</select>
```

## 新增列删除列

```sql
ALTER TABLE t_auth_rules ADD COLUMN  disk_mapping varchar(10) DEFAULT null;
ALTER TABLE t_auth_rules ADD COLUMN  shear_plate varchar(10) DEFAULT null;
ALTER TABLE t_auth_rules DROP column disk_mapping;
ALTER TABLE t_auth_rules DROP column shear_plate;
```

## 拿到pgsql数据库的序列号

```sql
SELECT (''||''||nextval('Q_BASE')) as id
```

## linux命令行操作pgsql数据库

```shell
psql --host=127.0.0.1 --port=5432 --dbname=basicframe --username=basicframe
```

## linux命令行操作pgsql数据库 - 无需输入密码

```shell
PGPASSWORD=123456 psql -h 127.0.0.1 -p 5432 -d basicframe -U basicframe -c "SELECT * FROM DUAL"
```

## linux将查询数据导出到csv文件

```shell
PGPASSWORD=123456 psql -h 127.0.0.1 -p 5432 -d basicframe -U basicframe -c "COPY (SELECT * FROM DUAL) TO STDOUT CSV" > rolename.csv
```

## 查询索引

指定tablename查询表是否包含索引

```sql
select * from pg_indexes where tablename = 't_user';
```

### PostgreSQL - 查询所有索引

`pg_indexes` 是一个视图，可以通过它获取某个表的[索引](https://so.csdn.net/so/search?q=索引&spm=1001.2101.3001.7020)信息。`pg_indexes`的定义如下：

```
SELECT
	n.nspname AS schemaname,
    c.relname AS tablename,
    i.relname AS indexname,
    t.spcname AS tablespace,
    pg_get_indexdef(i.oid) AS indexdef
FROM pg_index x
    JOIN pg_class c ON c.oid = x.indrelid
    JOIN pg_class i ON i.oid = x.indexrelid
    LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
    LEFT JOIN pg_tablespace t ON t.oid = i.reltablespace
WHERE (c.relkind = ANY (ARRAY['r'::"char", 'm'::"char"])) AND i.relkind = 'i'::"char";
```

例如从 `pg_indexes`中获取pg系统表`pg_index`表的索引信息：

```
select * from pg_indexes where tablename = 'pg_index';
```

查询结果

```
schemaname | tablename |         indexname         | tablespace |                                           indexdef
------------+-----------+---------------------------+------------+-----------------------------------------------------------------------------------------------
 pg_catalog | pg_index  | pg_index_indrelid_index   |            | CREATE INDEX pg_index_indrelid_index ON pg_catalog.pg_index USING btree (indrelid)
 pg_catalog | pg_index  | pg_index_indexrelid_index |            | CREATE UNIQUE INDEX pg_index_indexrelid_index ON pg_catalog.pg_index USING btree (indexrelid)
(2 rows)
```

## 创建索引

```sql
-- 0.6 200w数据量
SELECT * FROM t_auth_r_master_slave WHERE masterid = '1170293';
-- 加上索引之后
CREATE INDEX masterid_index ON t_auth_r_master_slave (masterid);
-- 0.051  将近10倍速
SELECT * FROM t_auth_r_master_slave WHERE masterid = '1170293';
```

## 删除索引

```sql
DROP INDEX status_index;
```

## 查看postgresql进程

```shell
ps -ef|grep postgres
```

## 查看psotgresql端口

```shell
netstat -npl | grep postgres
```

## postgresql服务命令 - 启停/状态

### pg_ctl启动/停止postgres

首先切换至postgres用户，找到数据库bin目录./pg_ctl执行

```shell
[root@localhost bin]# su postgres
bash-4.2$ cd /usr/pgsql-10/bin/
```

查看postgres运行状态 -D pgsql目录

```shell
bash-4.2$ ./pg_ctl status -D /var/lib/pgsql/10/data
pg_ctl: server is running (PID: 61585)
/usr/pgsql-10/bin/postgres "-D" "/var/lib/pgsql/10/data/"
```

pg_ctl启动postgres

```shell
bash-4.2$ ./pg_ctl start -D /var/lib/pgsql/10/data
```

pg_ctl停止postgres

```shell
bash-4.2$ ./pg_ctl stop -D /var/lib/pgsql/10/data
```

pg_ctl重启postgres

```shell
bash-4.2$ ./pg_ctl restart -D /var/lib/pgsql/10/data
```

### systemctl启动/停止postgres

```shell
systemctl status postgresql-9.6
systemctl start postgresql-9.6
systemctl stop postgresql-9.6
# 开机时启用一个服务
systemctl enable postgresql-9.6.service
# 开机时关闭一个服务
systemctl disable postgresql-9.6.service
# 查看服务是否开机启动
systemctl is-enabled postgresql-9.6.service
```

## postgresql修改自增序列号'Q_BASE'

查询当前序列号Q_BASE

```sql
SELECT nextval('Q_BASE');
```

重置数据库Q_BASE

```sql
alter sequence Q_BASE restart with 44900000;
```

## postgresql模糊查询带有下划线"_"

```sql
-- pgSQL中下划线是一个特殊字符。所以在使用通配符包含下划线的时候我们应该对下划线进行转义
select schema_name from information_schema.schemata where schema_name like '%sch\_%'
-- 模糊查询133.40.32.19_SSH_22等格式的数据，需要对下划线进行转义
SELECT r.* FROM t_auth_res r WHERE 1=1 AND r.name like '133.40.32.19\_%_%' and r.status = 'normal'
```

## postgresql导出导入csv

从csv导入

```sql
PGPASSWORD=123456 psql -h 127.0.0.1 -p 5432 -d basicframe -U basicframe -c "copy import_auth_data from  '/data/4a/import_auth_data.csv' delimiter ',' csv; "
```

导出至csv

```sql
PGPASSWORD=123456 psql -h 127.0.0.1 -p 5432 -d basicframe -U basicframe -c "COPY ($role_query) TO STDOUT CSV" > rolename.csv
```

## postgresql恢复备份操作

### 1、数据库全量备份

1、登录到服务器后台

2、控制台直接执行pg_dump命令

```bash
pg_dump -h 127.0.0.1 -p 5432 -U basicframe -d basicframe -W -F c -b -v -f /data/4a/basic20211024.backup
```

备注： -U：用户名 	-d：数据库

主备备份文件目录：/data/backup/4a_db_backup/cfb

```
启停服务
su - postgres   
pg_ctl start    #启动
pg_ctl stop     #停止
pg_ctl restart  #重启
```

### 2、数据库备份恢复

1、停止所有数据库服务（不停止服务，无法删除数据库）

```
service postgresql-9.3 stop
kill -INT `head -1 /var/lib/pgsql/9.3/data/postmaster.pid`
service postgresql-9.3 start
```

```
SELECT pg_terminate_backend(pg_stat_activity.pid)
 FROM pg_stat_activity
 WHERE datname=&apos;basicframe&apos; AND pid<>pg_backend_pid();
```

注：创建用户命令

```
[postgres@localhost data]$ createuser basicframe
[postgres@localhost data]$ createdb basicframe -E UTF8 -e
```

2、进入数据库，删除对应的数据库命（注;对应的备份数据库名）   drop database basicframe （此处只是举例，具体情况具体处理）

```
Starting postgresql-9.3 (via systemctl):                   [  确定  ]
[root@localhost tmp0811]# su postgres
bash-4.2$ psql -l
                                       资料库列表
    名称    |   拥有者   | 字元编码 |  校对规则   |    Ctype    |       存取权限        
------------+------------+----------+-------------+-------------+-----------------------
 basicframe | basicframe | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | 
 postgres   | postgres   | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | 
 template0  | postgres   | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
            |            |          |             |             | postgres=CTc/postgres
 template1  | postgres   | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
            |            |          |             |             | postgres=CTc/postgres
(4 行记录)
bash-4.2$ psql -d basicframe
psql (9.3.6)
输入 "help" 来获取帮助信息.

basicframe=# drop database basicframe
basicframe-# 
basicframe-# \q
basicframe-# \q
bash-4.2$ exit
exit
```

3、创建新数据库

```
[root@localhost tmp0811]# su postgres               
bash-4.2$ psql -d basicframe
psql (9.3.6)
输入 "help" 来获取帮助信息.

basicframe=# create database basicframe
basicframe-# 
basicframe=# create database basicframe
basicframe-# alter ower venustech
basicframe-# 


create database basicframe

（alter ower venustech）

退出：\q

exit
```

4、执行备份恢复操作

```
pg_restore -h 127.0.0.1 -p 5432 -U basicframe -d basicframe -W -j 14 -v /data/basic20211024.backup
```

## postgres 未指定字段类型问题（type "unknown"）

```sql
SELECT (array_to_string(array_agg(t.cmdaudit),'#!')) as cmdaudit 
FROM ( SELECT 'test-new-xx' as cmdaudit from t_acc_master ) t
```

postgres数据库较低版本（9.3.6）存在以下问题，这里的问题是`'' as name`实际上并没有为值指定类型。这是`unknown`类型，而 PostgreSQL 通常从诸如您将其插入到哪个列或您将其传递给哪个函数之类的东西推断出真正的类型。

```
Could not determine polymorphic type because input has type "unknown"
```

PostgreSQL 简写

```
''::text
```

案例

```sql
SELECT (array_to_string(array_agg(t.cmdaudit),'#!')) as cmdaudit 
FROM ( SELECT 'test-new-xx' ::text as cmdaudit from t_acc_master ) t

SELECT (array_to_string(array_agg(t.cmdaudit),'#!')) as cmdaudit FROM ( 
SELECT VARCHAR 'test-new-xx' as cmdaudit from t_acc_master ) t

SELECT (array_to_string(array_agg(t.cmdaudit),'#!')) as cmdaudit FROM ( 
SELECT TEXT 'test-new-xx' as cmdaudit from t_acc_master ) t

SELECT (array_to_string(array_agg(t.cmdaudit),'#!')) as cmdaudit FROM ( 
SELECT CAST('test-new-xx' AS text) as cmdaudit from t_acc_master ) t
```

## Postgres数据库备份异地(不同主机)备份

备份

```
pg_dump 数据库名 -h 要备份的数据库ip -p 你的端口 -U postgres > 备份结果文件名.dmp
pg_dumpall -h 要备份的数据库ip -p 你的端口 -U postgres > 备份结果文件名.dmp
eg：
pg_dump database -h 192.168.101.62 -p 5432 -U postgres > fileName
pg_dumpall -h 192.168.101.62 -p 5432 -U postgres > fileName
```

还原

```
psql -h 要还原的数据库ip -p 端口 -U postgres -d 指定数据库名 < 备份文件地址
psql -h 要还原的数据库ip  -p 端口 -U postgres < 备份文件地址
eg：
psql -h 192.168.101.68 -p 5432 -U postgres -d new_db < fileName
psql -h 192.168.101.68 -p 5432 -U postgres < fileName
```

## postgresql 当前时间，当前月，下个月  

```
SELECT TO_CHAR(NOW(), 'YYYY-MM-DD HH24:MI:SS') THISDATETIME, 
 TO_CHAR(NOW(), 'YYYY-MM') THISMONTH, TO_CHAR(NOW() + INTERVAL '1 MONTH','YYYY-MM') NEXTMONTH 
```

> 查询日期时间偏移（昨天、本周、本月、上月、本年统计数据） 【Postgresql 基础】

参考：[链接](https://blog.csdn.net/qq_43084261/article/details/114262160?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242)

```sql
select  (now() - interval '1 day') 
-- 将查询时间结果偏移一天到昨天的当前时刻
select  (now() - interval '1 month') 
-- 将查询时间结果偏移一月到上个月的当前时刻
select  (now() - interval '1 year') 
-- 将查询时间结果偏移一年到上年的当前时刻
select  (now() - interval '1 week ') 
-- 将查询时间结果偏移一周到上周的当前时刻
偏移可以用到的时间单位：
year :年
week ：该天在所在的年份里是第几周
timezone_minute：时区偏移量的分钟部分
timezone_hour：时区偏移量的小时部分
timezone：与UTC的时区偏移量，以秒记。正数对应 UTC 东边的时区，负数对应 UTC 西边的时区
second ：秒
quarter：日期中年所在季度(1-4)
month：月(0-11)
minute：分钟(0-59)
milliseconds：
isodow：周中的第几天 [1-7] 星期一：1) 星期天：(7)
dow：周中天的索引(0-6 ；星期天是 0)
doy：一年的第几天(1-365/366)
hour：小时(0-23)
day: 天(1-31)
```

## postgreSQL安装

```
yum install readline
yum -y install -y readline-devel
yum install zlib-devel
```

附：

```
yum -y install perl-ExtUtils-Embed openssl openssl-devel pam pam-devel libxml2 libxml2-devel libxslt libxslt-devel openldap openldap-devel python-devel readline-devel
yum install gcc
```



```
tar -xvf postgresql-10.8
cd postgresql-10.8
./configure
make     make已经执行完成
su
make install
adduser postgres
mkdir /usr/local/pgsql/data
chown postgres /usr/local/pgsql/data
su - postgres
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data >logfile 2>&1 &
/usr/local/pgsql/bin/createdb test
/usr/local/pgsql/bin/psql test
```

## postgreSQL卸载

停掉服务

```
service postgres-9.6 stop
```

卸载旧版本的postgres

```
yum remove postgresql*
```

验证是否卸载完毕

```
rpm -qa | grep postgresql
```

删除安装包

```
sudo apt-get --purge remove postgresql\*
```

删除相关配置文件及用户信息

```
sudo rm -r /etc/postgresql/
sudo rm -r /etc/postgresql-common/
sudo rm -r /var/lib/postgresql/
sudo userdel -r postgres
sudo groupdel postgres
```

https://www.py.cn/db/postgresql/15467.html

## 数据库初始化(创建角色用户数据库)

init_database.sql：

```sql
CREATE ROLE basicframe WITH superuser PASSWORD '123456';
ALTER ROLE basicframe WITH LOGIN;
ALTER ROLE basicframe WITH CREATEROLE;
ALTER ROLE basicframe WITH CREATEDB;
ALTER ROLE basicframe WITH REPLICATION;
CREATE TABLESPACE basicframe LOCATION '/data/database/pgsql/data/pg_basicframe';
CREATE DATABASE basicframe OWNER basicframe TABLESPACE basicframe;
```

创建数据库用户

```
createuser basicframe -P
```

创建数据库/密码

```
createdb basicframe  -E UTF8 -e
```

远程进入到psql命令行

```
psql -U pg_user -d pg_db -h pg_host -p 5432
```

```
su - postgres   

pg_ctl start    #启动
pg_ctl stop     #停止
pg_ctl restart  #重启
```

## 关闭服务器

有多种方法可以关闭数据库服务器。您可以通过向主`postgres`进程发送不同的信号来控制关闭的类型。

- SIGTERM

  这是*智能关机*模式。收到SIGTERM 后，服务器不允许新连接，但让现有会话正常结束工作。它仅在所有会话终止后关闭。如果服务器处于在线备份模式，它还会等待直到在线备份模式不再处于活动状态。当备份模式处于活动状态时，仍将允许新连接，但仅限于超级用户（此例外允许超级用户连接以终止在线备份模式）。如果在请求智能关闭时服务器处于恢复状态，则只有在所有常规会话终止后才会停止恢复和流式复制。

- 信号情报

  这是*快速关机*模式。服务器不允许新连接并发送所有现有服务器进程SIGTERM，这将导致它们中止当前事务并立即退出。然后等待所有服务器进程退出并最终关闭。如果服务器处于在线备份模式，则备份模式将终止，使备份无效。

- 非常

  这是*立即关机*模式。服务器将向所有子进程发送SIGQUIT并等待它们终止。如果有任何在 5 秒内没有终止，它们将被发送SIGKILL。一旦所有子进程退出，主服务器进程就会退出，而不进行正常的数据库关闭处理。这将导致在下次启动时恢复（通过重放 WAL 日志）。仅在紧急情况下才建议这样做。

该[pg_ctl](https://www.postgresql.org/docs/10/app-pg-ctl.html)程序提供了一个方便的接口发送这些信号关闭服务器。或者，您可以`kill`在非 Windows 系统上直接使用发送信号。的PID的的`postgres`方法可以使用的发现`ps`程序，或从文件`postmaster.pid`中数据目录中。例如，要进行快速关机：

```
$ kill -INT `head -1 /usr/local/pgsql/data/postmaster.pid`
```

## shell命令行执行SQL

```
#!/bin/bash

#echo "-----------------------> $1"
#echo "-----------------------> $(pwd)"

echo "Running sql to initializating table space for central ...."
psql -f /var/lib/pgsql/installTool/init_database10.sql
echo "done."

#echo "Running sql to initializating tables for central ...."
#psql -f /data/database/pgsql/installTool/centraldb_init.sql
#echo "done."

#echo "Running sql to initializating Menu And Buttons for central ...."
#psql -f /data/database/pgsql/installTool/centraldb_init_data.sql
#echo "done."

#echo "##############################################################"

#echo "Finished installing the postgresql database...."

#add pgsql connet config
if [ `grep -c "# localhost" /var/lib/pgsql/10/data/pg_hba.conf` -eq '0' ]; then
    echo -e "# localhost config
host    all             all             ::1/128                 md5" >> /var/lib/pgsql/10/data/pg_hba.conf
fi

```



```
#!/bin/bash

del_stdin_buf()
{
    read -d '' -t 0.1
}

echo "Copying files to postgre..."

selfPath=$(cd `dirname $0`; pwd)

/usr/pgsql-10/bin/postgresql-10-setup initdb

cp -fr $selfPath/pg_hba.conf /var/lib/pgsql/10/data/
echo "copying files .... $selfPath/pg_hba.conf  ===> /var/lib/pgsql/10/data/"

cp -fr $selfPath/postgresql.conf /var/lib/pgsql/10/data/
echo "copying files .... $selfPath/postgresql.conf  ===> /var/lib/pgsql/10/data/"

systemctl enable postgresql-10.service

systemctl start postgresql-10.service

echo "PostgreSQL has been installed successful."
```

## pg_upgrade 数据库升级

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





## pgsql安装【在线版】

```
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql10-server
sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
sudo systemctl enable postgresql-10
sudo systemctl start postgresql-10
```

## 设置序列

设置序列的值

```
alter sequence Q_BASE restart with 6666666666;
```

将序列的值设置成10000，下一个将是10001

```sql
select setval('Q_BASE', 10000);
```

获取下一个序列值，注意获取后，序列值将会自动增加

```sql
select nextval('Q_BASE');
```

## 交错序列

```sql
DROP SEQUENCE q_base_;

    CREATE SEQUENCE q_base_MASTER
      START WITH 1
      INCREMENT BY 2
      NO MINVALUE
      NO MAXVALUE
      CACHE 1;

CREATE SEQUENCE q_base_STANDBY
  START WITH 2
  INCREMENT BY 2
  NO MINVALUE
  NO MAXVALUE
  CACHE 1;	
  
ALTER SEQUENCE q_base 
START WITH 1
INCREMENT BY 2
NO MINVALUE
NO MAXVALUE
CACHE 1;

```



设置偶数(even):设置当前序列以下一个最近偶数开始，并且每次递增为2。

设置奇数(odd): 设置当前序列以下一个最近奇数开始，并且每次递增为2。



t_hot_backup_sequence



## 查询所有序列

```sql
select relname from pg_class where relkind='S'
```

## 将所有表的序列当前值设置为所需的值

pg的存储过程的创建如下所示：

```sql
create or replace function "public"."update_sequence"("v" int4)
returns void as
$$
   declare
   seq_record record;
   begin
        for seq_record in (select relname from pg_class where relkind='S') loop
             execute 'alter sequence ' || seq_record.relname || ' restart with ' || v || ';';
         end loop;    
    end;        
$$
language plpgsql volatile
cost 100;
```

存储过程中 执行sql语句，要加execute

mybatis的mapper.xml文件中调用pg的存储过程如下：

```xml
<select id="xxx方法名">
      select update_sequence(#{number})
</select>
```

注意：mybatis调用存储过程的入参类型要和pg中创建的存储过程的入参类型一致，不要会找不到对应的存储类型

mybatis 中执行pg的存储过程 用select ; 执行mysql的存储过程 用call

```sql
CREATE OR REPLACE FUNCTION "public"."alter_sequences"("odd_even" varchar)
RETURNS void AS
$$
DECLARE
	seq_record record;
	sequence_sql VARCHAR;
	sequence_now INTEGER;
	sequence_next INTEGER;
BEGIN
	FOR seq_record IN ( SELECT relname FROM pg_class WHERE relkind = 'S' )
	LOOP
		sequence_sql := 'SELECT cast(nextval(' || '''' || seq_record.relname || '''' || ') AS INTEGER)';
		execute sequence_sql into sequence_now;	
		IF ( odd_even = 'odd' ) THEN
			-- 奇数递增
			IF ( mod(sequence_now, 2) = 0 ) THEN
				 sequence_next = sequence_now + 1;
				 EXECUTE 'ALTER SEQUENCE ' || seq_record.relname || ' RESTART WITH ' || sequence_next || ' INCREMENT BY 2 NO MINVALUE NO MAXVALUE CACHE 1;';
			END IF;
			IF ( mod(sequence_now, 2) = 1 ) THEN
				 sequence_next = sequence_now;
				 EXECUTE 'ALTER SEQUENCE ' || seq_record.relname || ' RESTART WITH ' || sequence_next || ' INCREMENT BY 2 NO MINVALUE NO MAXVALUE CACHE 1;';
			END IF;					
		END IF;
		
		IF ( odd_even = 'even' ) THEN
			-- 偶数递增
			IF ( mod(sequence_now, 2) = 0 ) THEN
				 sequence_next = sequence_now;
				 EXECUTE 'ALTER SEQUENCE ' || seq_record.relname || ' RESTART WITH ' || sequence_next || ' INCREMENT BY 2 NO MINVALUE NO MAXVALUE CACHE 1;';
			END IF;
			IF ( mod(sequence_now, 2) = 1 ) THEN
				 sequence_next = sequence_now + 1;
				 EXECUTE 'ALTER SEQUENCE ' || seq_record.relname || ' RESTART WITH ' || sequence_next || ' INCREMENT BY 2 NO MINVALUE NO MAXVALUE CACHE 1;';
			END IF;					
		END IF;
	END LOOP;
END;
$$
language plpgsql volatile
cost 100;
```

## 查询数据库指定模式下面的所有表列表

```sql
SELECT
            tablename AS "name"
        FROM
            pg_tables
        WHERE
            tablename NOT LIKE 'pg%'
        AND tablename NOT LIKE 'sql_%' AND schemaname = 'public'
				
				
				
				
				SELECT
            viewname AS "name"
        FROM
            pg_views
        WHERE
            schemaname = 'public'
```

## 经典的postgreSQL数据解决方案文章

https://my.oschina.net/Kenyon?tab=newest&catalogId=317791&sortType=time