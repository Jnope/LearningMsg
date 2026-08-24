国智技术

# 设想一面

## event loop

JS 是单线程语言，依赖事件循环处理异步。

-   执行栈同步代码执行完毕
-   清空微任务队列（如 Promise.then、MutationObserver）
-   取一个宏任务（如 setTimeout、I/O）执行
-   微任务在当前宏任务结束后立即执行，优先级高于下一个宏任务。

处理大量数据计算时: 我会利用微任务进行状态更新，确保 UI 渲染前数据已就绪；耗时计算放入 Web Worker 避免阻塞主线程

## 闭包

函数能访问其词法作用域外的变量。
常见场景包括：数据私有化（模拟私有变量）、函数柯里化、防抖节流以及 React Hooks 的状态保持。
通用组件库时，用闭包缓存配置信息，避免重复读取。
会导致变量无法被 GC 回收，若处理不当易引发内存泄漏。
排查时可通过 Chrome DevTools 的 Memory 面板对比 Heap Snapshot。
预防措施包括：及时解绑事件监听器、清除定时器、避免在闭包中持有大对象引用，并在组件卸载时执行清理逻辑

## JS

weakMap: 键必须是对象，且为弱引用，不阻止 GC
weakRef: 持有对象的弱引用
可用于构建安全的缓存机制：用户存储海量 dom，当组件销毁、对象被 GC 后，缓存自动清理，无需手动管理生命周期，杜绝因缓存导致的内存膨胀。

## TS

### 核心

增加了静态类型系统和编译时检查，不改变运行时行为。
价值：接口契约化（定义清晰的 API 类型）、重构安全性（类型推导减少遗漏）、以及团队协作效率（类型即文档）。
最佳实践：

-   严格模式（strict: true）开启所有检查；
-   优先使用 interface 定义对象结构，type 用于联合类型和工具类型；
-   避免滥用 any，用 unknown 替代并做类型守卫；
-   利用 tsconfig 的 paths 配置路径别名；
-   在 monorepo 中共享类型包，确保前后端类型一致。

### 泛型

实现类型复用和类型安全。
从基础接口派生出表单组件的 props 类型，确保字段名与数据模型一致。
提升了开发体验，还在编译期拦截了大量类型错误，降低复杂表单的维护成本。

## ES6

### 原型链

JS 通过原型链实现继承：访问对象属性时会沿 **proto** 链向上查找，直到 null。
ES6 的 class 本质是原型继承的语法糖，constructor 对应构造函数，extends 通过中间对象实现继承链。虽然语法更清晰，但底层仍是 **proto** 和 prototype。
在排查 instanceof 失效或跨 iframe 类型判断问题时，原型链知识是定位根因的基础。

## Promise

Promise 立即执行
then 进入微任务
Promise.all: 任意失败返回
Promise.allSettled：全部结束后返回

## VUE

可变数据 + 模板编译，响应式系统自动追踪依赖，开发者只需声明数据变化，框架自动更新视图；
Proxy，可拦截对象的所有操作（包括 in、delete、数组索引等），且采用惰性响应式——仅在访问时才递归转换，大幅减少初始化开销；
更适合表单密集型场景，自动依赖追踪减少了手动优化的负担；

### pinia

完美支持 TS 类型推导，无需额外包装；
支持 Composition API 风格的 setup 语法；
模块化天然支持，无需 modules 嵌套；
体积更小（约 1KB）

### 通信

父子组件用 props/emit；
跨层级用 provide/inject，适合主题、全局配置等不频繁变更的数据；
兄弟或复杂状态用 Pinia；

## React

不可变数据 + 函数式，每次渲染都是全新快照，通过 useMemo/useCallback 显式优化；
单向数据流在复杂状态流转时更可预测；

-   数据流向可预测，调试简单；
-   状态变更集中，易于追踪和测试；
-   避免双向绑定导致的隐式依赖和循环更新

### hooks

链表结构存储状态，渲染时，按调用顺序遍历链表，将当前 hook 与链表节点一一对应。
不可在条件语句中使用：调用顺序可能改变，导致状态错位。

#### useEffect 和 useLayoutEffet

useEffect 在浏览器绘制后异步执行，不阻塞渲染，适合数据获取、订阅等副作用。
useLayoutEffect 在 DOM 变更后、绘制前同步执行，适合读取/修改 DOM 布局（如测量尺寸、防止闪烁），如表格列宽自适应，同步测量后设置样式，笔面列宽跳动。
误用 useLayoutEffect 会阻塞渲染，导致页面卡顿；误用 useEffect 处理布局会导致视觉闪烁。
SSR 环境中 useLayoutEffect 会报警告，需用 useIsomorphicLayoutEffect 兼容

### diff 算法

基于三个假设优化 O(n³) 到 O(n)：

-   比较同层节点，不跨层级移动；
-   不同类型元素产生不同树；
-   通过 key 标识列表元素稳定性。

具体策略：单节点比较时，类型不同直接替换；类型相同保留 DOM 节点，更新属性；列表比较时，有 key 则按 key 匹配复用，无 key 则按索引匹配（易出错）

### 性能优化

减少渲染次数 + 降低渲染成本：

-   React.memo 避免 props 未变时的重渲染；
-   useMemo/useCallback 缓存计算结果和函数引用；
-   React.lazy + Suspense 代码分割；
-   虚拟滚动（react-virtuoso）处理大列表；
-   状态拆分：将频繁变化的状态下沉到子组件，避免父组件重渲染；
-   使用 useTransition/useDeferredValue 将非紧急更新降级。

### 状态管理

组件局部状态用 useState/useReducer；
跨组件但层级浅用 Context；
全局复杂状态用 Zustand/Jotai/Redux/mobx。
Zustand 轻量、无 Provider、TS 友好，适合中小型项目；Jotai 原子化状态，适合细粒度响应式；Redux Toolkit 规范但较重，适合大型团队

### 并发模式

将渲染变为可中断、可恢复、可优先级调度的任务。
核心 API：useTransition 将状态更新标记为低优先级，保持高优先级更新（如输入）响应；
useDeferredValue 延迟更新值；
Suspense 支持数据获取与渲染解耦。

### React Server Components (RSC)

RSC 允许组件在服务端渲染并流式传输到客户端
优势：

-   客户端 JS 包体积——服务端组件代码不发送到浏览器；
-   直接访问后端资源（DB、文件系统），无需 API 层；
-   自动代码分割，按需加载。

它不是替代 SSR，而是补充：服务端组件负责数据获取和静态渲染，客户端组件负责交互。在数据治理平台中，报表详情页可用 RSC 直接查询数据库，避免 BFF 聚合开销，同时减少客户端 JS 体积，首屏加载更快。RSC 是 React 生态向全栈演进的重要方向

## 函数

Composition API：通过普通函数实现逻辑复用（Composables），彻底解决了 Mixins 的命名冲突、来源不清晰、类型推导差等问题

## DOM

### 虚拟 dom

价值在于：

-   提供跨平台抽象层（SSR、Native、Canvas）；
-   保证正确性——通过 Diff 算法确保最小化更新，避免开发者手动优化出错；
-   声明式编程提升开发效率。
-   Vue3 的编译时优化（静态提升、Patch Flags、Block Tree）使运行时 Diff 仅作用于动态节点。

大数据渲染：虚拟滚动 + 原生 dom

## 性能优化

大数据：虚拟滚动
CSS 优化：避免强制同步布局（读写分离）；使用 transform/opacity 触发 GPU 加速；contain 属性限制重排范围。、JS 优化：长任务拆分（requestIdleCallback/setTimeout）；Web Worker 处理计算密集型任务。
图片优化：懒加载、WebP 格式、响应式图片。
缓存策略：Service Worker 离线缓存、localStorage 缓存接口数据。
代码层面：防抖节流、事件委托、减少全局变量
首屏加速：路由分割，图片懒加载，API 请求并行，屏外加载

网络优化：减少请求数、降低传输体积、缩短 RTT

-   HTTP/2 多路复用，减少连接开销；
-   资源压缩（Gzip/Brotli）、图片格式优化（WebP/AVIF）；
-   缓存策略：强缓存（Cache-Control）+ 协商缓存（ETag）；
-   预加载/预连接关键资源；
-   CDN 加速静态资源；
-   BFF 层接口聚合，减少瀑布请求。

内存泄露：

-   Chrome DevTools Memory 面板拍摄多次 Heap Snapshot，对比增长对象；
-   未清理的定时器、事件监听器、闭包引用大对象；
-   全局搜索 addEventListener/setInterval，确保组件卸载时清理；
-   使用 WeakRef/WeakMap 替代强引用缓存；
-   引入 why-did-you-render 检测不必要重渲染

打包优化：

Webpack 优化：1）splitChunks 合理分包，避免单文件过大；2）thread-loader 多线程编译；3）cache 开启持久化缓存；4）esbuild-loader 替代 babel 提速；5）bundle-analyzer 分析并剔除冗余依赖。
Vite 优化：1）开发环境利用原生 ESM，无需打包；2）生产环境 Rollup 构建，配置 manualChunks 分包；3）optimizeDeps 预构建第三方依赖。

## 构建

Webpack 是打包型工具，开发环境需要将整个应用打包成 bundle 再启动，项目越大启动越慢；生产环境通过 tree-shaking、code splitting 等优化包体积。
Vite 开发环境利用浏览器原生 ES Module，无需打包，实现毫秒级冷启动和热更新（HMR 仅替换变更模块）；生产环境使用 Rollup 打包，性能优异。
Webpack 生态成熟、插件丰富，适合传统大型项目；Vite 开发体验极佳，适合新项目或需要快速迭代的项目。

## 组件库

API 设计：遵循 React/Vue 官方组件设计规范，props 命名统一、类型约束清晰，提供 TypeScript 类型导出；
样式方案：采用 CSS Modules 或 CSS-in-JS 实现样式隔离，提供主题定制能力（如通过 CSS 变量或 theme provider）；
测试覆盖：单元测试用 Jest + Testing Library 覆盖核心逻辑，视觉回归测试用 Chromatic 或 BackstopJS；
文档与演示：用 Storybook 搭建文档站，每个组件提供基础用法、高级用法和交互示例

### 包发布原则

ackage.json 配置：正确设置 name、version、main、module、types 入口，files 字段指定发布文件；
构建输出：同时输出 CJS 和 ESM 格式；
依赖管理：将框架依赖（Vue/React）放在 peerDependencies，避免与使用者版本冲突；
类型声明：确保 .d.ts 文件正确导出，TS 项目可自动推断类型；
版本管理：遵循语义化版本（SemVer），使用 Changesets 管理变更日志；
私有仓库：公司内部包发布到私有 npm，配置 .npmrc 自动鉴权。

## 微前端

将单体前端应用拆分为多个小型应用、各自独立开发部署、最终整合为一个完整应用的架构风格。
核心价值在于：

-   技术栈无关——不同子应用可使用 Vue、React 等不同框架；
-   独立开发部署——各团队独立迭代，互不影响；
-   渐进式升级——老旧模块可逐步替换，无需整体重写

缺点：
首次加载体积增大；
子应用间通信复杂；
调试困难；

qiankun（基于 Single-SPA）：应用隔离通过 JS 沙箱 + CSS 隔离实现，路由自动劫持，生态成熟，适合 Vue/React 混合场景；
Module Federation（Webpack5）：通过远程模块共享，运行时动态加载，适合同构技术栈、深度通信；
iframe：天然隔离但通信困难、体验差，仅适合简单场景；
wujie（基于 Web Component）：基于 Web Components 实现样式隔离。

### 通信

qiankun 提供 props 传递机制，主应用可通过 props 向子应用传递数据和回调函数；
路由传参、storage 共享；
子应用之间可通过主应用中转通信；
使用自定义事件总线：通过发布 / 订阅自定义事件 CustomEvent 来传递消息；
inGlobalState: 传入 state，返回 action 实例，onGlobalStateChange 监听，setGlobalState 修改状态，offGlobalStateChange 取消监听

### 样式隔离

qiankun 通过 JS 沙箱（Proxy 沙箱）隔离全局变量，通过 CSS Shadow DOM 或样式扫描（Scoped CSS）隔离样式。

## Backend for Frontend (BFF) 层的设计思路和实践。

介于前端和后端之间的适配层，核心目标是解决前端聚合多个后端接口的痛点。
设计思路：1）接口聚合——前端一个请求，BFF 并行调用多个后端接口，聚合后返回统一格式；2）数据裁剪——根据前端页面需求，BFF 只返回需要的字段，减少传输体积；3）格式适配——将后端统一返回格式转换为前端组件期望的格式，如分页结构、树形结构转换；4）缓存策略——BFF 层可缓存高频查询结果，减轻后端压力。

# 一面

## electron

## dom 渲染

## 横纵排中间

## postion

## spring 全家桶

Spring Framework：IoC 容器（依赖注入）和 AOP
Spring Boot：自动配置 + 起步依赖，内嵌 tomcat
Spring Cloud：微服务解决方案总集，服务注册发现、配置中心、负载均衡、熔断限流等一整套能力
Nacos：服务注册及配置
OpenFeign：声明式 HTTP 客户端
Spring Cloud LoadBalancer：客户端负载均衡，配合 OpenFeign 使用，将请求轮询/加权分发到多个服务实例
Sentinel：流量控制和熔断降级，支持 QPS/线程数限流、熔断策略（慢调用/异常比例）、系统自适应保护
MyBatis + PageHelper：SQL 半自动 ORM 框架
Spring Security：认证授权如 OAuth2，JWT
Spring Cloud Stream：基于发布-订阅模式的消息驱动，封装了 Kafka/RabbitMQ 等消息中间件，统一编程模型
Spring Integration：企业集成模式实现，支持消息路由、转换、聚合等
Spring Boot Actuator + Micrometer：生产级监控端点，暴露 /health、/metrics、/info 等端点，配合 Prometheus 抓取指标，实现 JVM\、系统、业务指标埋点
Spring Task / Quartz：@Scheduled 和复杂定时任务
Spring Cache：@Cacheable、@CacheEvict 缓存
Spring Validation:@Valid, @NotBlank 校验
Spring AOP：切面编程

## 网关及负载均衡

nginx nacos 模块动态从 nacos 获取 upstream 节点；
nacos 注册和发现服务；
loadbalancer 根据负载均衡策略从实例列表中选择一个目标实例；

## K8S

k8s 类型 pod，deployment，job
