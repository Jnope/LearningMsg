# 星环面试
## 一面
1. mybatis和ibatis等框架区别
2. interceptor和filter
首先进入filter再进入interceptor，最后aspect  
filter可以修改请求，需要在servlet容器中实现，基于回调，只能在servlet请求（包括jsp、servlet、静态资源）前后使用  
interceptor依赖spring，基于反射实现，只能在controller前后处理
3. 序列化反序列化
序列化把Java对象转换为字节：可以进行深复制、写入文件  
需要实现Serializable接口  
注意问题：需要父类实现Serializable或父类有无参构造函数；如果父类覆写了tostring方法，子类需要不同时也需要覆写；类内所有属性必须都可序列化；第一次序列化后实例属性变化不影响再次序列化  
4. classnotfound异常
缺少依赖项，存在版本冲突
5. spring事务级别、事务失效
级别：  
DEFAULT 以数据库为准  
READ_COMMITTED 读已提交，可以读到其他事务结果--导致不一致--不可重复读  
READ_UNCOMMITTED 读未提交，读到其他事务未提交数据--可能回滚--脏读  
REPEATABLE_READ 可重复读，唯一性--其他事务进行中不能插入事务--幻读，公司采用  
SERIALIZABLE 串行化--效率低  
事务失效：public方法，类内非Transactional注解方法调用；数据库引擎不支持事务；异常被捕获；事务传播设置；非runtimeException或非error异常
6. jvm内存
堆：可动态扩展或缩减，新生代频繁GC-Xmx 和 -Xms  
栈：线程私有  
方法区：存类结构信息、常量、静态变量、代码  
程序结束前：线程执行字节码行号指示器  
7. jdk版本优势，升级过程中问题及解决
8. 设计模式
9. 项目规格，需求来源
10. 框架和技术选型，怎么选型
11. 给需求后怎么做，需要考虑哪些，是否引入新框架
12. 服务规格、稳定性、监测、部署步骤、内存溢出-sql连接池
内存溢出定位：jstat -class PID；jstat -gc PID；启用远程调试；jmap保存内存数据  
数据库连接池：netstat -pan | grep 3306 | wc -l 排查最大连接数和空闲连接数  
13. 鉴权控制和足迹上报
14. 性能优化：大象流、高并发、高可用-保证请求可用、缓存
高可用性：负载均衡、数据库主从主备复制、异常处理及熔断、服务自动重启和快速回滚  
15. 多线程编程，其导致的问题，集合类型保证多线程

16. 慢接口怎么排查、sql优化
DNS解析
是否出现负载不均衡、无缓存  
第三方接口超时  
逻辑太复杂  
线程满了  
锁设计不合理  
慢sql：分页问题--利用id替代offset；索引；join或子查询过多；in、like过多；数据量过大  
17. 页面加载失败、打不开排查
查看控制台js、html、css文件是否正常加载，查看服务器中文件是否被破坏，ningx配置  
接口请求：是否发送，响应码，查看是否到达tomcat  
18. 前后端都会的优点，前后端感受的区别
19. 怎么理解微服务、微服务之间交互
20. docker
## 二面-笔试
### 

### 分配任务
规定时间内可以完成的任务数以及总价值：最高价值，价值相同时选中任务数最多
DFS + 贪心 + 判断
任务信息：耗时、价值
思路：将任务列表进行排序，耗时低优先，相同耗时价值高优先
全局变量res记录价值及数量；
遍历任务列表，满足耗时 < 规定时长时，往后寻找下一个任务；每次满足条件下与res比较价值及数量

## 三面-线下

# 网题
1. 工程flink的checkpoint的具体过程
2. checkpoint对系统有什么影响
3. int和integer的区别,装箱与拆箱
4. 怎么调用integer的方法，具体过程
5. java内部类和静态内部类
6. 二叉树前序遍历
7. best time to buy and sell stock最多二次买卖
8. 常见设计模式

1. c++static
2. java static
3. 操作系统栈和堆区别
4. 存储代码段
5. 缓存，缓存不一致性
6. tcp与udp应用场景
7. 聚簇索引与非聚簇索引
8. join的种类以及实际操作
9. 编程：二叉树按层遍历

1. 子数组和为0 dp算法
2. flink怎么保证exactly-once语义
3. 高并发环境怎么做测试
Jmeter模拟、压力测试、性能测试、稳定性测试
4. 常用的检查系统failure的方法
5. java堆怎么排查错误
6. 工程

1. fibbonacci算法改进
2. redis怎么存储数据
string  
hash集合  
list  
set、zset  

3. redis存的key是什么值
字符串值
4. MySQL的ORM方式
5. mybatis怎么对应实体
xml中指定type；namespace指定dao层  
6. applicationcontext逻辑
7. bean的生命周期和种类
8. flink的检查点机制怎么改进？
9. flink窗口
10. flink的任务失败模型
11. 还有很多工程上的细节
