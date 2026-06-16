StateGraph (状态图)
├── State (状态)
├── Nodes (节点)
├── Edges (边)
│ ├── Normal Edges (普通边)
│ └── Conditional Edges (条件边)
└── Checkpointer (检查点)

# graph结构

## 状态

TypedDict或BaseModel
初始化；
更新：根据节点返回值，更新对应属性值；
规约器：属性更新方式，默认覆盖属性值

```python
from typing import Annotated
from typing_extensions import TypedDict
from operator import add

class State(TypedDict):
    foo: int # 默认覆盖
    bar: Annotated[list[str], add] # bar的更新方式为向list尾部添加值
```

MessagesState类：message属性为AnyMessage列表，使用add_messages规约器；

## 节点

通常是 Python 函数，第一个参数为State，第二个为RunnableConfig
START节点：指定首步执行节点
END节点：终止节点

```python
def my_node(state: State, config: RunnableConfig):
    print("In node: ", config["configurable"]["user_id"])
    return {"results": f"Hello, {state['input']}!"}
```

### 缓存

- 编译图时（或指定入口点时）指定缓存。
- 为节点指定缓存策略。每个缓存策略支持：
  key_func 用于根据节点输入生成缓存键，默认为使用 pickle 对输入进行 hash。
  ttl，缓存的存活时间（秒）。如果未指定，缓存将永不过期。

## 边

一个节点可以有多个出边

### 普通边

从一个节点到下一个节点

### 条件边

调用函数来确定接下来要到哪个（或哪些）节点
`python graph.add_conditional_edges("node_a", routing_function, {True: "node_b", False: "node_c"}) `

#### Send

边无法确切时使用，第一个参数为节点，第二个参数为传入对象

```python
def continue_to_jokes(state: OverallState):
    return [Send("generate_joke", {"subject": s}) for s in state['subjects']]

graph.add_conditional_edges("node_a", continue_to_jokes)
```

### Command

更新state 且 指向下一节点

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    return Command(
        # state update
        update={"foo": "bar", modifications = interrupt("请输入修改内容：")}, # interrupt人工介入
        # control flow
        goto="my_other_node",
        graph=Command.PARENT # optional，可以指向父图的节点
    )
```

## 配置

将图的某些部分标记为可配置，通常是为了方便在模型或系统提示之间切换

```python
class ConfigSchema(TypedDict):
    llm: str

config = {
  "recursion_limit": 5, # 递归限制，默认25
  "configurable": {
    "llm": "anthropic",
    "thread_id": "1", # 线程id
    "checkpoint_id": "xxxx" # 检查点id
  }
}
```

config可以在node中获取：config.get("configurable", {}).get("llm", "openai")

## 构建 & 编译

```python
builder = StateGraph(State, config_schema=ConfigSchema, input=InputState, output=OutputState)
builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
builder.add_node("node_3", node_3)
builder.add_edge(START, "node_1")
builder.add_edge("node_1", "node_2")
builder.add_edge("node_2", "node_3")
builder.add_edge("node_3", END)

checkpointer = InMemorySaver()
graph = builder.compile(checkpointer=checkpointer)
graph.invoke(inputs, config=config)
```

### 检查点

# 持久化

graph.get_state(config) 获取最新或指定检查点的状态
graph.get_state_history(config) 获取状态历史
