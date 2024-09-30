# 分库分表
## 概念
https://blog.csdn.net/wuhuagu_wuhuaguo/article/details/113241886
## Sharding-Sphere
https://www.cnblogs.com/cg-ww/p/16614454.html

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
``` java
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
