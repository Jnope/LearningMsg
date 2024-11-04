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
## 配置
hibernate.cfg.xml  
``` xml
<hibernate-configuration>
  <session-factory>
    <property name="">
    </property>
    <mapping resource="Employee.hbm.xml"/>
  </session-factory>
</hibernate-configuration>
```
### 属性
| 属性 | 描述 |
| :---- | :---- |
|hibernate.dialect|数据库类型|
|hibernate.connection.driver_class|JDBC驱动类|
|hibernate.connection.url|数据库url|
|hibernate.connection.username|用户名|
|hibernate.connection.password|密码|
|hibernate.connection.pool_size|连接池数量|
|hibernate.connection.autocommit|允许自动提交|
|hibernate.cache.provider_class|选择二级缓存支持类|
|hibernate.cache.use_query_cache|是否使用缓存|
|hibernate.jdbc.batch_size|批处理大小|

## 会话Session
Session用于获取物理连接

## 映射
将类与数据库表对应  
### 通过文件
文件名 Name.hbm.xml-- Name为类名
``` xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
 "-//Hibernate/Hibernate Mapping DTD//EN"
 "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd"> 
<hibernate-mapping>
  <class name="Name" table="NAME">
    <meta attribute="class-description">
         This class is mine. 
    </meta>
    <id name="id" type="int" column="id">
      <generator class="native">
    </id>
    <property name="pro" column="pro" type="string" />
  </class>
</hibernate-mapping>
```
### 通过注解
``` java
@Entity
@Table(name = "my_table")
public class MyClass {
  @Id @GenerateValue
  @Column(name = "id")
  private int id;

  @Column(name = "first_name")
  private String firstName;
}
```
### 类型
基本为小写java类型：character映射为java的String

## 执行
``` java
SessionFactory factory;
Session session = factory.openSession();
Transaction tx = null;
try {
  tx = session.begionTransaction();
  session.createQuery("FROM table_name").list(); // 查询
  session.get(Type.class, id); // 查询
  session.update(type); // 更新
  session.delete(type); // 删除
  session.save(type; // 插入
  tx.commit();
} catch (HibernateException e) {
  if (tx != null) {
    tx.rollback();
  }
} finally {
  session.close();
}
```
### 查询语言
不分大小写
#### FROM
Query query = session.createQuery("FROM my_table");  
query.list();  
#### AS
给类分配别名 Query query = session.createQuery("FROM my_table AS m");
#### SELECT
Query query = session.createQuery("SELECT m.col FROM my_table AS m");
#### WHERE
Query query = session.createQuery("SELECT m.col FROM my_table AS m WHERE m.id=1");
#### ORDER BY
Query query = session.createQuery("SELECT m.col FROM my_table AS m ORDER BY m.id DESC");
#### GROUP BY
Query query = session.createQuery("SELECT m.col FROM my_table AS m GROUP BY m.id");
#### 命名参数
Query query = session.createQuery("SELECT m.col FROM my_table AS m WHERE m.id= :id");
query.setParameter("id", 1);
#### UPDATE
Query query = session.createQuery("UPDATE FROM my_table set name=111");
#### DELETE
Query query = session.createQuery("DELETE FROM my_table WHERE id=1");
#### INSERT
Query query = session.createQuery("INSERT INTO my_table(name) ");
#### Query
执行query.executeUpdate();  
查询query.list();  
设置查询起点query.setFirstResult(1);  
设置查询大小query.setMaxResults(10);  

### 标准查询
Criteria c = session.createCriteria(MyClass.class);
#### 限制
c.add(Restrictions.eq("id", 1));  
Restrictions方法：eq, gt, lt, like, ilike, between, isNullm isNotNull, isEmpty, isNotEmpty
#### 分页
c.setFirstResult(1);  
c.setMaxResult(20);  
#### 排序
c.addOrder(Order.desc("id"));  
c.addOrder(Order.asc("id"));  
#### projection
c.setProjection(Projections.avg("cnt"));  
Projections方法有avg, rowCount, countDistinct, max, min, sum  

### 原生SQL
#### 标量查询--查1-多个列
SQLQuery query = session.createSQLQuery("SELECT first_name, salary FROM my_table");  
query.setResultTransformer(Criteria.ALIAS_TO_ENTITY_MAP);  
#### 实体查询--查正标
SQLQuery query = session.createSQLQuery("SELECT * FROM my_table");  
query.addEntity(MyClass.class);
#### 命名参数
query.setParameter("id", 1);


### 批处理
``` java
SessionFactory factory;
Session session = factory.openSession();
Transaction tx = null;
try {
  tx = session.begionTransaction();
  for (int i = 0; i < 1000; i++) {
    Type type = new Type();
    session.save(type);
    if (i % 50 == 0) { // 50为批处理大小
      session.flush();
      session.clear();
    }
  }
  tx.commit();
} catch (HibernateException e) {
  if (tx != null) {
    tx.rollback();
  }
} finally {
  session.close();
}
```

## 缓存
一级缓存为Session，强制  
### 二级缓存
在映射文件中定义并发策略：Transactional, Read-write, Nonstrict-read-write, Read-only
``` xml
<hibernate-mapping>
   <class name="Employee" table="EMPLOYEE">
      <cache usage="read-write"/>
   </class>
</hibernate-mapping>
```
在查询中启用缓存：query.setCacheable(true);

## 拦截器
继承EmptyInterceptor拦截器

## 一对多操作
实体类中类型需要为Set<Type> property
``` xml
<hibernate-mapping>
   <class name="Employee" table="EMPLOYEE">
      <set name="property">
        <key column="id /> <!-- 外键 -->
        <one-to-many class="path.to.Property"/>
      </set>
   </class>
</hibernate-mapping>
```
