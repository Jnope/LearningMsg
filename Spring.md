# 异步任务
创建ThreadPoolTaskExecutor的bean
``` java
@Bean("MyTaskExcutor")
public ThreadPoolTaskExecutor MyTaskExcutor() {
         ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
         //配置核心线程数
        executor.setCorePoolSize(5);
        //配置最大线程数
        executor.setMaxPoolSize(20);
        //配置队列大小
        executor.setQueueCapacity(200);
        //线程池维护线程所允许的空闲时间
        executor.setKeepAliveSeconds(30);
        //配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix(ConstantFiledUtil.KMALL_THREAD_NAME_PREFIX);
        //设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
        executor.setWaitForTasksToCompleteOnShutdown(true);
        //设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住
        executor.setAwaitTerminationSeconds(60);
          // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是由调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //执行初始化
        executor.initialize();
        return executor;
}
```
使用: 需要@EnableAsync开启注解，不要在同一个类内调用@Async任务--aop不生效
``` java
@Async
void do(){};
@Aync("MyTaskExcutor")
void method(){}
```

# SpringBoot和SpringCloud
SpringBoot内置Tomcat、Jelly，完全注解话
SpringCloud依赖SpringBoot

# Springboot
## 网络
### 跨域
前端跨域请求失败本质是浏览器安全行为：在返回头中添加Access-Control-Allow-Origin
1.@CrossOrigin(origins = "*")注解controller类
2.implements WebMvcConfigurer复写addCorsMappings
``` java
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 所有接口
                .allowCredentials(true) // 是否发送 Cookie
                .allowedOriginPatterns("*") // 支持域
                .allowedMethods(new String[]{"GET", "POST", "PUT", "DELETE"}) // 支持方法
                .allowedHeaders("*")
                .exposedHeaders("*");
    }
```
3.实现CorsFilter的bean
4.在响应头增加"Access-Control-Allow-Origin": "*"

## 高并发
### 配置
最大线程数--server.tomcat.max-threads，默认200
### 缓存
前端接口缓存
redis缓存
内存缓存Cache
### 负载均衡
ribbon
Apache JMeter测试
@EnableDiscoveryClient注解application
``` java
# application.yaml
my-service:
  ribbon:
    listOfServers: http://localhost:8080,http://localhost:8081
    # 负载均衡策略为轮询
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    # 连接超时时间
    connectTimeout: 1000
    # 读取超时时间
    readTimeout: 1000
```
### 异步任务
耗时操作异步执行
### 数据库
分库分表分区分片
增大数据库活跃连接数
### 限流
redis: 全局限流，接口限流，用户限流，接口+用户限流
### 降级
失败降级

## SQL调优

## 应用
### 秒杀系统
1.controller层加redis锁，service层执行事务 --> 事务提交发生在方法执行完毕之后
2.数据库悲观锁 select * from myTable limit 1 for update; for update进行对查询数据加锁，加的是行锁
3.数据库直接使用update更新而不是查询
