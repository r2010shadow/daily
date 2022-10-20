# Stage3

## ORACLE冷备份及恢复

### 一、备份

冷备份发生在数据库已经正常关闭的情况下，当正常关闭时会提供给我们一个完整的数据库。

冷备份是将关键性文件拷贝到另外的位置，可以根据重要性文件克隆一份数据库。

冷备份还原注意事项：
 两台数据库服务器的操作系统必须是同构的

（即：aix->aix或者linux->linux），不能是异构的（linux->aix），

否则是没有用的。如果是异构的，那么一般采用数据泵的方式。

1.编写备份脚本

通过操作系统的命令来实现的备份机制：cp、scp



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

## ORACLE热备份

### 一、热备份

初始化日志：重置redo日志号
首先关闭数据库

```
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
```

进入到mount状态

```
SQL> startup mount;
ORACLE instance started.
Total System Global Area 838860800 bytes
Fixed Size 8626240 bytes
Variable Size 348131264 bytes
Database Buffers 473956352 bytes
Redo Buffers 8146944 bytes
Database mounted.
```



查看当前redo log号

```
SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /u01/app/oracle/archive2/
Oldest online log sequence 4
Next log sequence to archive 6
Current log sequence 6
```



重置redo log号

```
SQL> recover database until cancel;
Media recovery complete.

SQL> alter database open resetlogs;
Database altered.
```



重置完成

```
SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /u01/app/oracle/archive2/
Oldest online log sequence 1
Next log sequence to archive 1
Current log sequence 1
SQL>
```



创建测试数据

创建hr的t1表，并插入两行数据

```
SQL> create table hr.t1(id number);
Table created.

SQL> insert into hr.t1 values(1);
1 row created.

SQL> insert into hr.t1 values(2);
1 row created.

SQL> commit;
Commit complete.
```



执行日志切换

```
SQL> alter system switch logfile;
System altered.

切换后redo log号变为了2.

SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /u01/app/oracle/archive2/
Oldest online log sequence 1
Next log sequence to archive 2
Current log sequence 2
SQL>
```



查看归档记录：归档生成

此时我们再插入两条数据：

```
SQL> insert into hr.t1 values(3);
1 row created.
SQL> insert into hr.t1 values(4);
1 row created.
SQL> commit;
Commit complete.
```



日志切换，查看redo备份，

```
SQL> alter system switch logfile;
System altered.
SQL> host ls -l /u01/app/oracle/archive2/
total 1336
-rw-r----- 1 oracle oinstall 1343488 Feb 14 04:34 arch_orcl_1_1_1064464371.dbf
-rw-r----- 1 oracle oinstall 22016 Feb 14 06:43 arch_orcl_1_2_1064471384.dbf
```



redo log号更新

```		
SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /u01/app/oracle/archive2/
Oldest online log sequence 1
Next log sequence to archive 3
Current log sequence 3
```



初始化日志完成，开始备份

开始备份
开始备份之前我们查看一下当前的scn号

```
SQL> select current_scn from v$database;
CURRENT_SCN
.----------
2350352
SQL>

从当前scn号开始进行备份

SQL> alter database begin backup;
Database altered.

查看状态：ACTIVE

SQL> select * from v$backup;
FILE# STATUS CHANGE# TIME CON_ID
.---------- ------------------ ---------- ------------------- ----------
1 ACTIVE 2350453 2021-02-14 06:51:33 0
2 ACTIVE 2350453 2021-02-14 06:51:33 0
3 ACTIVE 2350453 2021-02-14 06:51:33 0
4 ACTIVE 2350453 2021-02-14 06:51:33 0
5 ACTIVE 2350453 2021-02-14 06:51:33 0
7 ACTIVE 2350453 2021-02-14 06:51:33 0
8 ACTIVE 2350453 2021-02-14 06:51:33 0
9 ACTIVE 2350453 2021-02-14 06:51:33 0
10 ACTIVE 2350453 2021-02-14 06:51:33 0
11 ACTIVE 2350453 2021-02-14 06:51:33 0
10 rows selected.
SQL>

此时scn号已经改变

SQL> select current_scn from v$database;
CURRENT_SCN
.-----------
2350489
SQL>

确认当前数据文件的路径（执行命令，以实际路径为主）

SQL> select name from v$datafile;
NAME
.--------------------------------------------------------------------------------
/u01/app/oracle/oradata/orcl/system01.dbf
/u01/app/oracle/oradata/orcl/undotbs01.dbf
/u01/app/oracle/oradata/orcl/sysaux01.dbf
/u01/app/oracle/oradata/orcl/testbs_01.dbf
/u01/app/oracle/oradata/orcl/undotbs02.dbf
/u01/app/oracle/oradata/orcl/users01.dbf
/u01/app/oracle/oradata/orcl/testbs_02.dbf
/u01/app/oracle/oradata/orcl/testbs_03.dbf
/u01/app/oracle/oradata/orcl/bigtbs_01.dbf
/u01/app/oracle/oradata/orcl/data1_1.dbf
10 rows selected.
SQL>
```



创建备份目录：

[oracle@MiWiFi-R4CM-srv archive2]$ mkdir -p $ORACLE_BASE/hot/fulldb_20210214/archive/

将除了控制文件外的所有数据文件进行拷贝到我们创建的备份目录：（控制文件使用二进制备份）

```
[oracle@MiWiFi-R4CM-srv archive2]$ cp $ORACLE_BASE/oradata/orcl/*.dbf $ORACLE_BASE/hot/fulldb_20210214/archive/
[oracle@MiWiFi-R4CM-srv archive2]$ cp $ORACLE_BASE/oradata/orcl/redo0* $ORACLE_BASE/hot/fulldb_20210214/archive/
```



此时的备份目录：

备份控制文件：（备份二进制的控制文件）

```
SQL> alter database backup controlfile to ‘$ORACLE_BASE/hot/fulldb_20210214/archive/control01.ctl’;
Database altered.

备份成创建语句的文件：

SQL> alter database backup controlfile to trace as ‘$ORACLE_BASE/hot/fulldb_20210214/ctl_create.sql’;
Database altered.
SQL>
```



此时的备份目录状态：


此时我们的数据文件已经备份完成，结束数据文件的备份，还需要备份归档文件和密码文件以及spfile

结束数据文件的备份

```
SQL> alter database end backup;
Database altered.
SQL>

记录一下redo log号

SQL> select current_scn from v$database;
CURRENT_SCN
.-----------
2351333
SQL>

备份归档日志

SQL> srchive log list;
SP2-0734: unknown command beginning “srchive lo…” - rest of line ignored.
SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /u01/app/oracle/archive2/
Oldest online log sequence 1
Next log sequence to archive 3
Current log sequence 3

切换日志

SQL> alter system switch logfile;
System altered.

SQL> archive log list;
Database log mode Archive Mode
Automatic archival Enabled
Archive destination /u01/app/oracle/archive2/
Oldest online log sequence 2
Next log sequence to archive 4
Current log sequence 4
SQL>
```



将归档日志进行拷贝到备份目录，进行备份：

```		
SQL> host ls -l /u01/app/oracle/archive2/
total 7604
-rw-r----- 1 oracle oinstall 1343488 Feb 14 04:34 arch_orcl_1_1_1064464371.dbf
-rw-r----- 1 oracle oinstall 22016 Feb 14 06:43 arch_orcl_1_2_1064471384.dbf
-rw-r----- 1 oracle oinstall 6417920 Feb 14 07:11 arch_orcl_1_3_1064471384.dbf
SQL> host cp /u01/app/oracle/archive2/* $ORACLE_BASE/hot/fulldb_20210214/archive/
SQL>

最后备份密码文件和spfile

[oracle@MiWiFi-R4CM-srv archive2]$ cp $ORACLE_HOME/dbs/spfile* $ORACLE_BASE/hot/fulldb_20210214/archive/
[oracle@MiWiFi-R4CM-srv archive2]$ cp $ORACLE_HOME/dbs/orapw* $ORACLE_BASE/hot/fulldb_20210214/archive/
```

所有备份完成

