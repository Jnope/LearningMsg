# 分库分表

## 概念

https://blog.csdn.net/wuhuagu_wuhuaguo/article/details/113241886

## Sharding-Sphere

https://www.cnblogs.com/cg-ww/p/16614454.html
分片键和主键分离，主键由Sharding-Sphere自动生成，分片键由业务指定
主键：雪花 ID，1位符号位 + 41位时间戳 + 10位机器ID + 12位序列号；解决多库自增 ID 冲突

```yaml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          t_order:
            actual-data-nodes: ds_${0..1}.t_order_${0..3}
            # 1. 分表策略：基于 user_id 取模，保证同用户数据在一起
            table-strategy:
              standard:
                sharding-column: user_id
                sharding-algorithm-name: table-mod
            # 2. 主键策略：基于 order_id 自动生成雪花ID
            key-generate-strategy:
              column: order_id
              key-generator-name: snowflake
        key-generators:
          snowflake:
            type: SNOWFLAKE
            props:
              worker-id: 1 # 每个微服务实例必须配置不同的 worker-id
        sharding-algorithms:
          table-mod:
            type: MOD
            props:
              sharding-count: 4
```

# 多线程

https://www.cnblogs.com/crazymakercircle/p/14655412.html

## volatile, synchronized, atomic

### volatile

作用于变量
可见性，不能保证原子性，不阻塞

### synchronized

作用于类、方法、块
可见性、原子性，但会阻塞，为悲观锁

### atomic

原子性，CAS自旋比较

## 多线程传参

https://www.cnblogs.com/jpfss/p/10783847.html

# UT

JUnit5 + Mockito
@ExtendWith(MockitoExtension.class) 作用于测试类头部

## 模拟构造函数

待测试方法中存在new获取实例，需要模拟实例返回：

```java
  @InjectMocks
  private TestClass testClass;

	@Test
	public void testHasName() {
		try (MockedConstruction<MyClass> mocked = Mockito.mockConstruction(MyClass.class, (mock, context) -> {
			Mockito.when(mock.do()).thenReturn("111");
		})) {
			assertEquals("111", testClass.do());
		}
	}
```
