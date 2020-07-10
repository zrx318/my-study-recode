## 用处

redo log 和undo log是innodb的事务日志。redo log是重做日志，提供前滚操作，undo log是回滚日志，提供回滚操作。

1. redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，它用来恢复提交后的物理数据页(恢复数据页，且只能恢复到最后一次提交的位置)。
2. undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录。

### redo log

擦，下次再记录，回去打游戏了先~。

https://www.cnblogs.com/f-ck-need-u/archive/2018/05/08/9010872.html