## MySQL导数据
```
mysql -u USER -p --database DBNAME -h IP --port PORT -ss -e "sql" | sed 's/\t/","/g;s/^/"/;s/$/"/;s/\n//g' > file.csv;
```
## mysqldump
 mysqldump -u用戶名 -p密码 -d 數據库名 表名 脚本名;

1. 全表结构
```bash
mysqldump -uroot -p -d predb >db.sql;
```

2. 某张表
```bash
mysqldump -uroot -pdbpasswd -d dbname test>db.sql;
```
    
3. 表结构和数据
```bash
mysqldump -uroot -pdbpasswd  dbname > db.sql;
```

4. 导出數據库為dbname某张表(test)结构及表數據（不加-d）
```bash
mysqldump -uroot -p predb t_oracle_unit > unit.sql;
```