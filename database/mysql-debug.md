- 查询是否锁表
```sql
show OPEN TABLES where In_use > 0;
```
- 查询进程
```sql
show processlist;
```
 查询到相对应的进程,然后 kill id

- 查看正在锁的事务
```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
```
- 查看等待锁的事务

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```