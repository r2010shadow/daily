# Stage3

## ORACLE冷备份及恢复

### 一、备份

冷备份发生在数据库已经正常关闭的情况下，当正常关闭时会提供给我们一个完整的数据库。

冷备份是将关键性文件拷贝到另外的位置，可以根据重要性文件克隆一份数据库。

> 冷备份还原注意事项：
>  两台数据库服务器的操作系统必须是同构的
>
> （即：aix->aix或者linux->linux），不能是异构的（linux->aix），
>
> 否则是没有用的。
>
> 如果是异构的，那么一般采用数据泵的方式。

1.编写备份脚本

通过操作系统的命令来实现的备份机制：cp、scp

```
1.1 查询所有的文件所在位置

（数据文件，控制文件，参数文件，重做日志文件，归档日志文件，初始化参数文件，密码文件）

--查看参数文件位置
 SQL> show parameter spfile;
 --查看实例和数据库的相关信息
 SQL> select instance_name,version,status,archiver,database_status from v$instance;

 1.数据文件
 2.控制文件
 3.重做日志文件
 4.参数文件
 5.监听文件tnsnames.ora listener.ora
 6.密码文件PWDfile

直接拷贝oracle目录下的admin、oradata(datafile， controlfile，redo)、flash_recovery_area三个文件夹，

db_1目录下database(PWDfile、pfile)、dbs(spfile)、NETWORK/ADMIN(listener.ora、tnsnames.ora)，到其他存储实现备份。
```

```sql
# 查看字符集，最好将字符集保持一致
SQL> select * from v$nls_parameters; 
# 查看实例和数据库的相关信息　　　　　
SQL> select instance_name,version,status,archiver,database_status from v$instance;
# 控制文件
select * from v$controlfile;
# 数据文件
select name from v$datafile;
# REDO LOG文件
select * from v$logfile;
# 参数文件
select * from V$PARAMETER where name like '%spfile%';
show parameter spfile
# 临时文件
select name from v$tempfile;
```

1.2 创建备份的路径

```bash
mkdir /u01/app/backup
```

1.3 通过select语句构建拷贝脚本

```sql
set trim on        #截取掉不必要的空格
set trims on       #每个行的空格
set heading off     #去掉标题
set echo off        #去掉反馈
set feedback off    #去掉回显
set term off        #关闭终端信息

spool /home/oracle/cp_db.sh
     select  'cp '||name||' /u01/app/backup' from v$datafile               
     union
     select  'cp '||name||' /u01/app/backup' from v$controlfile
     union
     select  'cp '||member||' /u01/app/backup' from v$logfile
spool off
```

```bash
# 授权，构建拷贝命令
chmod +x cp_db.sh
/home/oracle/cp_db.sh  --停止数据库，执行脚本

或者通过awk构建命令：
ls -lSr /test | awk '{print "mv " $NF" /tmp/test/t"NR".conf"}' | bash
```

### 二、恢复

1.脱机恢复到原来位置的步骤：

方法一：最简单的方法（需建库）

```
 1.创建一个和原来一样的数据库。(安装路径和数据库名必须和原来一致)
 2.停止数据库 shutdown immediate；
 3.复制安装目录下的admin、oradata、flash_recovery_area覆盖，复制database(PWDfile、pfile) 覆盖
 4.启动数据库 startup;
```

方法二：（不需建库，稍麻烦点）

```
 (1)：操作系统重装，如果做冷备恢复，要保证相同操作系统，相同的数据库版本。
 (2)：正常安装oracle软件，只需要安装软件，不用建实例。
 (3)：数据覆盖，包括数据文件、参数文件、控制文件、日志文件、pwd文件，放在与原系统相同的目录。如果目录有所改变，则需要另外建立控制文件，修改pfile。
 (4)：建立服务：
 使用oradim 命令 cmd下 oradim -new -sid ggdb(实际sid名) ，表示建立一个服务，sid为ggdb。
 如果是在linux下，不需要此步。
 oradim -new -sid xxx -intpwd xxx -startmode a -pfile E:\xxx\xxx\xxx\init.ora
 (5)：建立监听： netca 来建立(建议将源系统的network下的文件拷过来，根据实际情况修改)。
 (6)：打开数据库： cmd
 set oracle_sid=orcl;
 sqlplus /nolog;
 conn / as sysdba;
 startup;
 验证数据库
 SQL> select count(*) from user_tables;
 至此，冷备份恢复成功。
```

2.脱机恢复到非原来位置的步骤：

```
有时候储存数据文件的磁盘坏了，可能需要改变数据文件的恢复位置。

1. 将备份文件恢复到正常的磁盘上
2. 将数据库加载为mount状态（startup mount）
3. alter database rename file 'u01/xxx.DBF'  to 'u02/xxx.DBF'    (u01磁盘损坏)
4. alter database open
```
