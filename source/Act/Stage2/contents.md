# Stage2
## Mysql Backup
### 文件种类划分

- 物理备份
  - 以xtrabackup为代表的物理备份是最常用的备份方法，经常被用在备份大的数据库上面。

- 逻辑备份
  - 以mysqldump为代表的逻辑备份，小于50G的数据量，用该工具备份起来比较合适。

### 按照备份内容划分

- 全量备份
  - 这个容易理解，就是数据库完整的一个备份

- 增量备份
  - 基于全量备份的基础之上的，例如全量备份备份截止日期是昨天，那么昨天到今天这部分数据就是增量备份

- 日志备份
  - 日志备份是在备份文件的基础上，再对mysql的binlog进行备份，我们知道binlog中记录了mysql的DDL和DML操作，利用binlog能够还原数据库的某个中间状态。

## xtrabackup备份知识点

1、当我们使用xtrabackup备份的时候，对于单机多实例的机器，最好分开时间备份，因为同一时间备份多个MySQL实例吗，会对磁盘IO影响较大，此时如果还有其他数据写入，会产生影响

2、可以利用NFS挂载的方式，将本地的数据备份在远程机器上，这样可以缓解本地的备份压力，同时可以保证本地硬盘出现问题不影响数据恢复。应用此种方法的时候，需要考虑机器之间的网络带宽。架构图如下：

3、众所周知，如果xtrabackup备份的备份文件用于恢复，需要两个步骤，第一步是apply_log，第二步是copy back。建议备份完成后，直接进行应用日志操作。如果失败，可以重新备份。否则如果默认可用，恢复时很难发现问题。

4、基于binlog做增量恢复也是一种比较方便的做法，既按照一定的周期进行全量备份，然后定时备份binlog，这期间的任意时间点的数据都可以通过备份的binlog进行恢复

5、如果有多台联机机，可以结合mysqldump和xtrabackup进行备份，设置一个阈值，比如50G，数据量在50G以下，mysqldump性能更好，数据量在50G以上，可以考虑使用xtrabackup进行备份。

6、备份数据的目的是为了恢复，最好能在备份完成之后，尝试恢复一把，保证备份数据的可用性。

7、对数据量很大的实例，有时备份时间特别长，恢复起来也很不方便，可采用跨机房从库备份的方法，此时可建立一个延迟从库，这样当主库误操作时，从库可以及时停止，跳过相关误操作步骤。

