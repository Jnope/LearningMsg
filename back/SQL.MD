# MYSQL
## 死锁
``` sql
insert into table(a, b, c) values (1, 2, 3) on duplicate key update b = b+1; # 主键a已存在时更新b的值
replace into table(a, b, c) values (1, 2, 3); # 主键a存在时先删除旧数据再写
```
问题：  
on duplicate key update在5.7版本中存在bug，高并发时会造成死锁，可见 https://cloud.tencent.com/developer/article/1609770  
replace into在数据库设置repeatable_read级别时，高并发会死锁--需要read_commit；且其删除旧数据而不是替换或覆写值，比较麻烦  
解决：  
``` sql
if not exists (select a from table where a = 1)
      insert into table(a,b,c) values(1,2,3)
   else
      update table set b=2
```
以上使用判断；或先select，不存在时insert，存在时update；或直接insert捕获错误后update  

# Hibernate
