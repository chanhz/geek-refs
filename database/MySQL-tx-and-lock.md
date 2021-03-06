根据加锁的范围，MySQL 里面的锁大致可以分成**全局锁、表级锁和行锁**三类。

InoDB 引擎支持行锁， MyIASM 只支持表锁

## 全局只读锁

Flush tables with read lock (FTWRL)

做全库逻辑备份时使用，实现一致性读。前提是数据库存储引擎支持此隔离级别


**全局锁问题**：
如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆；
如果在从库上备份，那么备份期间从库不能执行主库同步过来的 binlog，会导致主从延迟。


当 mysqldump 使用参数–single-transaction 的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。


## 表级锁
- 表锁
- 元数据锁（meta data lock，MDL)

在 MySQL 5.5 版本中引入了 MDL，
当对一个表做增删改查操作的时候，加 MDL 读锁；当要对表做结构变更操作的时候，加 MDL 写锁。


## ACID

## 隔离性与隔离级别


- 读未提交是指，一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交是指，一个事务提交之后，它做的变更才会被其他事务看到。
- 可重复读是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的（即使其他事务进行了对这部分数据进行了更改）。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
- 串行化，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。


## 查询超过 60 s 的事务记录：
```sql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60;
```

## 关于 autocommit
开启时，每次查询都自动提交。显示地使用 begin rollback commit 来处理复杂事务

关闭时，每次SQL执行都需要显示地 begin,rollback,commit

commit work and chain
如果执行 commit work and chain，则是提交事务并自动启动下一个事务，这样也省去了再次执行 begin 语句的开销。同时带来的好处是从程序开发的角度明确地知道每个语句是否处于事务中。


## 如何避免长事务对业务的影响？
**开发端**
1. 确认是否使用了 set autocommit=0。这个确认工作可以在测试环境中开展，把 MySQL 的 general_log 开起来，然后随便跑一个业务逻辑，通过 general_log 的日志来确认。一般框架如果会设置这个值，也就会提供参数来控制行为，你的目标就是把它改成 1。

2. 确认是否有不必要的只读事务。有些框架会习惯不管什么语句先用 begin/commit 框起来。我见过有些是业务并没有这个需要，但是也把好几个 select 语句放到了事务中。这种只读事务可以去掉。

3. 业务连接数据库的时候，根据业务本身的预估，通过 SET MAX_EXECUTION_TIME 命令，来控制每个语句执行的最长时间，避免单个语句意外执行太长时间。（为什么会意外？在后续的文章中会提到这类案例）


**从数据库端**
1. 监控 information_schema.Innodb_trx 表，设置长事务阈值，超过就报警 / 或者 kill；
2. Percona 的 pt-kill 这个工具不错，推荐使用；
3. 在业务功能测试阶段要求输出所有的 general_log，分析日志行为提前发现问题；
4. 如果使用的是 MySQL  5.6 或者更新版本，把 innodb_undo_tablespaces 设置成 2（或更大的值）。如果真的出现大事务导致回滚段过大，这样设置后清理起来更方便。

