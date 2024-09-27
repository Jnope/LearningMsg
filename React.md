https://blog.csdn.net/qq_16546829/article/details/137056845
# 对比vue
静态：数据驱动石头
单向数据流

# 虚拟DOM
## diff算法
diff找出增删改查最小节点: DFS后续遍历
fiber：任务划分，优先级调度 --> 优化DOM计算和渲染
fiber为链表

# 生命周期及Hook
## 生命周期
挂载前：构造函数, render，getDerivedStateFromProps
挂载后：componentDidMount
更新前：shouldComponentUpdate，getSnapshotBeforeUpdate
更新后：componentDidUpdate
卸载前：componentWillUnmount
## Hook
useEffect：componentDidMount, componentDidUpdate, componentWillUnmount -> DOM渲染后执行
useLayoutEffect: 所有 DOM 变更之后同步调用 -> 会改变当前渲染结果；依赖DOM尺寸的操作
useCallback：记忆化函数，避免函数重建
useRef：DOM挂载、记忆化变量
useState：初始化状态并返回值、更新函数
useRerducer: 当state之间存在依赖时，可以通过对象形式组合state，通过useRerducer获取状态值及更新方法
``` TypeScript
const reducer = (state1, state2) => {]
const [state, dispatch] = useReducer(reducer, {state1: val1, state2: val2});
dispatch(val3, val4);
```
自定义Hook：
``` JavaScript
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // do something
  }, []);
  return isOnline;
}
```

# 渲染过程
state更新 -> 合并状态 -> 异步处理(合并相同state的set) -> 批处理（合并多个set）—> 状态更新调度 -> 检查useEffect依赖 -> 计算虚拟DOM -> DOM更新->更新

# 事件代理
最外层容器绑定监听器（17之后挂载到dom节点） -> 事件合成 -> 冒泡到顶层 -> 根据目标判断组件并调用处理器

# 性能优化
## React.PureComponent || React.memo
浅比较组件props和state，相等时不更新；PureComponent用于类组件，memo用于函数组件
## useMemo
缓存计算结果
## useCallback
缓存回调函数
## state
减少不必要useState，合并多次state修改
## diff
减少树层级
为列表元素提供稳定唯一key
## 事件监听
使用合成事件减少监听数量
## 懒加载
lazy按需加载
## 渲染
将组件渲染到DOM外：createPortal
动画：requestAnimationFrame

# 高阶组件
包装其他组件的函数：抽取相同逻辑代码，增加组件额外功能，共享状态，天送检渲染

# 受控非受控
受控是用户输入直接改变state，通过onChange事件更新state
非受控是用户输入仅改变DOM属性，通过ref获取值

# 跨级通信
## useContext

## react-redux
store
state：store内数据
action：接收改变state命令
reducer：action的处理器
改变路径：view -> action -> reducer -> state

# react-router
基于hash路由，监听hashchange事件：location.hash
基于H5路由

# react 更新
fiber
批处理：减少刷新
