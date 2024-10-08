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

# 项目
## 结合TS
npx create-react-app my-project --template typescript创建
### 配置
安装依赖 npm install -D @types/react @types/react-dom  
tsconfig.json
``` json
{
    "compilerOptions": {
        "target": "es6",
        "allowJs": true,
        "declaration": true,
        "noImplicitAny": false,
        "outDir": "./out",
        "skipLibCheck": true,
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "strict": true,
        "module": "es6",
        "lib": ["es6", "dom", "es2016"],
        "moduleResolution": "node",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "jsx": "react"
    },
    "exclude": [
        "test",
        "dist",
        "out",
        "*.css",
        "*.scss",
        "node_modules",
        "webpack.config.js"
    ]
}
```
lib内必须要dom  
## 结合react-router
npm i react-router-dom  
创建router.tsx，index.tsx中直接使用<MyRouter />
``` tsx
import { BrowserRouter, Navigate, Route, Routes } from "react-router-dom";
import { Home } from "../home/Home";
import { About } from "../about/About";

export const MyRouter: React.FunctionComponent = () => (
    <BrowserRouter>
        <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/home" element={<Home />} />
            <Route path="/about/:param?" element={<About />} />
            <Route path='*' element={<Navigate to="/" />} />
        </Routes>
    </BrowserRouter>
);
```
### 使用
``` tsx
import { Link, useParams } from 'react-router-dom';
import style from './About.module.scss';

export const About: React.FunctionComponent = () => {
    const { param } = useParams();

    return <div className={style['about']}>
        <div>This is About: {param ?? 'nothing'}</div>
        <div><Link to={'/'}>Home</Link></div>
    </div>;
}
```

## 结合react-redux
npm i react-redux @reduxjs/toolkit  
创建store.ts, hook.ts, modelus  
``` tsx
// store.ts 配置store
import { configureStore } from "@reduxjs/toolkit";
import counter from "./modules/counter";
export const store = configureStore({
    reducer: {
        counter,
    }
})
export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch

// hook.ts 映射数据和方法，避免重复定义
import { TypedUseSelectorHook, useDispatch, useSelector } from "react-redux";
import { AppDispatch, RootState } from "./store";
export const useMyDisPatch = () => useDispatch<AppDispatch>();
export const useMySelector: TypedUseSelectorHook<RootState> = useSelector;

// modules文件夹内定义各个state
import { createSlice } from "@reduxjs/toolkit";

interface ICounter {
    count: number;
}

const initialState: ICounter = {
    count: 0
}

export const counterSlice = createSlice({
    name: 'counter',
    initialState,
    reducers: {
        increment: state => {
            state.count += 1;
        },
        decrement: state => {
            state.count -= 1;
        }
    }
})

export const { increment, decrement } = counterSlice.actions
export default counterSlice.reducer;
```
### 使用
``` tsx
// index.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import reportWebVitals from './reportWebVitals';
import { MyRouter } from './router/router';
import { Provider } from 'react-redux';
import { store } from './store/store';
const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <MyRouter />
    </Provider>
  </React.StrictMode>
);

// Home.tsx
import { NavLink } from 'react-router-dom';
import style from './Home.module.scss';
import { useDispatch, useSelector } from 'react-redux';
import { AppDispatch, RootState } from '../store/store';
import { useMyDisPatch, useMySelector } from '../store/hook';
import { counterSlice, decrement } from '../store/modules/counter';
export const Home: React.FunctionComponent = () => {
    const counter = useSelector<RootState, number>(state => state.counter.count); // 或 const counter = useMySelector(state => state.counter.count);
    const dispatch = useDispatch<AppDispatch>(); // 或 const dispatch = useMyDisPatch();
    return (<div className={style['home']}>
        <div>This is Home</div>
        <div>Current count is: {counter}</div>
        <div onClick={() => dispatch(counterSlice.actions.increment())}>Add</div>
        <div onClick={() => dispatch(decrement())}>Minus</div>
        <div><NavLink to={'/about/123'}>About</NavLink></div>
    </div>);
}
```
## 结合webpack
