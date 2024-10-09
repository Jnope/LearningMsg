# 星环面试
## 一面
1. mybatis和ibatis
2. interceptor和filter
3. 序列化反序列化
4. classnotfound异常
5. spring事务级别、事务失效
6. 设计模式
7. 怎么选型
8. 给需求后怎么怎么做
9. 服务规格、稳定性、监测、部署步骤、内存溢出-sql连接池
10. 鉴权控制和足迹上报
11. 性能优化：大象流、高并发、高可用、缓存
12. 慢接口怎么排查、sql优化
13. 页面加载失败、打不开排查
14. docker
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
4. 常用的检查系统failure的方法
5. java堆怎么排查错误
6. 工程

1. fibbonacci算法改进
2. redis怎么存储数据
3. redis存的key是什么值
4. MySQL的ORM方式
5. mybatis怎么对应实体
6. applicationcontext逻辑
7. bean的生命周期和种类
8. flink的检查点机制怎么改进？
9. flink窗口
10. flink的任务失败模型
11. 还有很多工程上的细节
