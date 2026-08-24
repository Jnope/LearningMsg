山西证券

# 设想一面--部门领导

## 量化平台深度追问

1. "你做的 TransQuant 平台，架构是怎么设计的？"
   基础设施层： K8S 服务部署和资源分配，Hadoop 分布式文件系统；
   数据：Mysql 存储用户等，Redis 存储临时数据、任务队列和锁；
   后端 springboot 微服务：认证、课题管理、任务编排；
   前端通过 react 构建页面，使用 nginx 进行反向代理；
   用户节点：独立的 K8S Pod，资源天然隔离，hdfs 保证文件安全；
2. "Redis 缓存策略具体怎么做的？为什么能降低 40% 响应时间？"
   缓存粒度：将课题报告完整结果拆分为根据图存储，对于频繁获取的数据存储于 redis，1h 过期；
   前端通过虚拟滚动和 mobx 持久化，实现按需加载数据，并杜绝重复加载；
   防穿透：先判断课题 id 是否存在；
3. "K8S 部署方案是怎样的？服务发现、配置管理用的什么？"
   CICD 自动构建服务镜像；
   jenkins 打包服务镜像、配置文件（包括 k8s 部署 freemark 文件）；
   在 manage 平台设置服务依赖和配置，下发后 k8s 进行 deployment；
4. "用户多课题任务流编排，具体怎么实现的？"
   前端使用 react-flow 实现课题任务流视图；
   用户进行课题编排后后端将任务流点线关系存储 mysql；
   执行时，获取起点，下一层级课题，并存储于 redis，循环执行至任务结束；

## Java / Spring Boot 基础与排错

1. "Spring Boot 自动装配原理了解吗？"
   SpringBootApplication -> EnableAutoConfiguration -> AutoConfigurationImportSelector -> 扫描所有 jar 包 -> 配置类注册@Been;

2. "你有没有遇到过线上 OOM 或者 CPU 100% 的情况？怎么排查的？"
   OOM 靠 Heap Dump + MAT 分析内存快照:-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/oom-heap.hprof 开启快照；# 抓堆转储（核心证据）jmap -dump:format=b,file=/tmp/heap-dump.hprof PID; # 抓 GC 状态 jstat -gcutil <PID> 1000 10 # 抓线程栈 jstack <PID> > /tmp/thread-stack.log;
   CPU 100% 靠 top 定位进程 → top -H 定位线程 → jstack 定位代码:
3. "Spring 事务失效的场景有哪些？"
   同类内方法调用；
   非 public 方法；
   类没有被 Spring 管理；
4. "你的代码阅读和排错能力体现在哪里？举一个具体例子。"

## MySQL 实战

1. "你做过哪些 SQL 优化？explain 怎么看？"
   索引优化：避免索引失效，覆盖索引查询字段；
   语句优化：避免 SELECT \*，减少不必要的 JOIN，子查询改 JOIN，深分页用 WHERE id > last_id LIMIT n，除去不必要 order by;
   表设计：大表拆分，redis 缓存；
   EXPLAIN 获取查询耗时：type 访问类型性能从优到劣：system > const > eq_ref > ref > range > index > ALL，key 索引，rows 扫描行数，extra 附加信息（Using index > where > index condition）
2. "慢查询怎么定位的？有没有遇到过锁等待？"
   explain 查看 type 是否为全表查询 ALL，是否使用索引，扫描行数等；
3. "索引失效的场景有哪些？"
   索引列被操作：substr、date 方法等；
   隐式类型转换：查询值和索引类型不匹配，如 123 和"123"；
   联合索引不匹配，联合索引右侧为范围查询；
   like 千导模糊；
   or 中存在非索引；

## Redis 深入

1. "Redis 过期策略是惰性删除还是定期删除？"
   任务队列定时扫描删除；
   缓存数据惰性删除；
2. "Redis 内存占用怎么排查？大 Key 问题怎么处理？"
   redis-cli info memory 查看内存指标；
   keyspace 查看 key 数量和过期 key，stats 查看过期情况；
   --bigkeys 查看大 key：unlink 异步删除，拆分；

3. "缓存穿透、缓存击穿、缓存雪崩分别怎么解决？"
   缓存穿透：参数前置校验，布隆过滤器（预存合法 key），缓存空值；
   缓存击穿：热点 Key 在缓存中过期的瞬间，大量并发请求同时到达，互斥锁仅一个线程能到数据库拿，热点 Key 永不过期；
   缓存雪崩：过期时间加随机值，多级缓存 db+redis+cache；

## 转行动机与稳定性

"你是微生物学硕士，怎么转到 IT 的？为什么选券商？"
"你做过全栈、AI Agent，为什么来面试后端与运维？"
"券商技术岗和互联网公司有什么不同？你怎么适应合规要求？"
第二梯队：大概率追问

## 短板应对（Kafka / Nacos / Linux）

1. "JD 里提到 Kafka，你用过的消息队列中间件有哪些？"（如果没有，准备好怎么表达学习路径）
2. "Nacos 你没用过，但服务注册发现你怎么理解？"
   Namespace + group + service
   服务注册；
   服务续约心跳；
   服务发现：
   服务下线：

3. "Linux 排查问题你一般用什么命令？"（至少准备好 top/ps/grep/awk/netstat 等常用命令的回答）
   top
   lsblk
   ps
   du
   tail

## 系统设计能力

1. "让你设计一个行情数据接入系统，你会怎么设计？"
   采集层：websocket 实时采集，rest 拉通历史数据
   缓冲层：消息队列
   处理层：数据清洗与计算
   服务层：websocket/gRPC 推送消息
2. "如果线上服务突然响应变慢，你的排查思路是什么？"
   定位前端渲染问题还是 API 响应：devtools TTFB 高即服务处理慢，Content Download 高则网络响应慢；
   渲染慢：大数据采用分页按需加载虚拟滚动渲染；
   API 慢：检查延迟和丢包，查看后端 cpu 内存使用，查看日志，慢 sql 排查；

## Python 数据处理

"你用 Python 做过哪些数据处理任务？Pandas 常用操作？"
"如果要你做行情数据补漏，你会怎么设计脚本？"

## 量化/金融概念

"你对 K 线、订单簿了解多少？"
"你知道基差和持仓量是什么意思吗？"
"你做量化平台时，数据是怎么流转的？"

## 运维与稳定性

1. "你有没有做过线上值班？故障复盘怎么做？"
2. "Docker 和 K8S 的区别是什么？K8S 里 Pod 和 Container 是什么关系？"
3. "Jenkins/GitLab CI/CD 你用过的？"
4. Prometheus
   Prometheus 拉取数据，存入时序数据库；
   Grafana 展示和告警
5. jvm 调优
   -Xms / -Xmx 初始堆/最大堆 设置一致；
   -Xmn 新生代设置堆 1/3~1/2;
   -Xss 线程栈 1M；
   -XX:MetaspaceSize / -XX:MaxMetaspaceSize 元空间大小显示设置；
   -XX:+HeapDumpOnOutOfMemoryError 内存泄漏检查
   -XX:HeapDumpPath=/data/logs/heapdump.hprof
   调优原则：
    - 每次只改 1~2 个参数，通过压测对比效果
    - 先跑基准，再优化，没有数据支撑的调优是盲调
    - 容器环境用 -XX:MaxRAMPercentage=75 替代固定内存参数，让 JVM 感知容器限制
6. GC
   G1 吞吐量优先；

    - -XX:+UseG1GC
    - -XX:MaxGCPauseMillis=200

ZGC 速度优先；
SerialGC：小服务；
Parallel GC：jdk8 默认，分新生代和老年代，回收时间不可预测；
GC 算法：

-   新生代：复制
-   早期老年代：标记清除
-   老年代：标记整理
-   堆：分代收集

7. 线程分析
   jstack 生成线程快照（Thread Dump） jstack <pid> > threads.txt
   jmap 生成堆内存快照（Heap Dump） jmap -dump:live,format=b,file=heap.bin <pid>
   jstat 监控 GC 和类加载统计

CPU 飙高：

-   top 找到 CPU 高的 Java 进程 PID
-   top -Hp <pid> 找到 CPU 高的线程 TID
-   将 TID 转为十六进制：printf "%x\n" <tid>
-   jstack <pid> | grep <十六进制 tid> -A 30 定位具体代码行

死锁：jstack
线程阻塞 / 等待：查看线程状态
线程池配置：
CPU 密集型 CPU 核数 + 1 减少上下文切换
IO 密集型 CPU 核数 × 2 ~ 4 线程大部分时间在等待 IO

# 一面
