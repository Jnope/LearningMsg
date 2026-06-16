# 智能体

## 基础

### 模型

```python
my_llm = ChatOpenAI(
    model=config.openai_model,
    temperature=0.3, # 控制模型输出的随机性。值越高，响应越具创造性；值越低，响应越确定性
    max_tokens=1000, # 输出长度
    api_key=config.openai_api_key,
    base_url=config.openai_api_base,
    timeout=1000, # 秒
    max_retries=1,
)

@wrap_model_call
def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:
    """中间件：动态模型选择器，根据对话复杂性选择模型。"""
    message_count = len(request.state["messages"])

    if message_count > 10:
        # 对较长的对话使用高级模型
        model = advanced_model
    else:
        model = basic_model

    request.model = model
    return handler(request)
```

### 工具

```python
@tool(args_schema=WeatherInput) # 输出格式 WeatherInput 为pydantic或json格式数据
def search(
    query: str,
    runtime: ToolRuntime # 运行时信息
) -> str:
    """搜索信息。"""

    messages = runtime.state["messages"]
    return f"结果：{query}"

@wrap_tool_call
def handle_tool_errors(request, handler):
    """中间件：工具的错误处理方法，使用自定义消息处理工具执行错误。"""
    try:
        return handler(request)
    except Exception as e:
        # 向模型返回自定义错误消息
        return ToolMessage(
            content=f"工具错误：请检查您的输入并重试。({str(e)})",
            tool_call_id=request.tool_call["id"]
        )

@tool
def clear_conversation() -> Command: # 用于更新代理的状态或控制图的执行流程
    """Clear the conversation history."""

    return Command(
        update={
            "messages": [RemoveMessage(id=REMOVE_ALL_MESSAGES)],
        }
    )
```

### 提示

#### system_prompt

系统提示，为字符串

```python
@dynamic_prompt
def user_role_prompt(request: ModelRequest) -> str:
    """中间件：动态提示，根据用户角色生成系统提示。"""
    user_role = request.runtime.context.get("user_role", "user")
    base_prompt = "你是一个有帮助的助手。"

    if user_role == "expert":
        return f"{base_prompt} 提供详细的技术响应。"
    elif user_role == "beginner":
        return f"{base_prompt} 简单解释概念，避免使用行话。"

    return base_prompt
```

### 状态

自定义状态模式必须扩展 AgentState 作为 TypedDict。

定义自定义状态有两种方式：

通过 中间件（推荐）
通过 create_agent 上的 state_schema

```python
class CustomState(AgentState):
    user_preferences: dict

class CustomMiddleware(AgentMiddleware):
    state_schema = CustomState
    tools = [tool1, tool2]

    def before_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
```

### 短期记忆 —— checkpointor

智能体通过消息状态自动维护对话历史
存储在状态中的信息可以被视为智能体的短期记忆：内存、数据库

```python
checkpointer_in_mem = InMemorySaver()
sqlite_conn = sqlite3.connect(str(config.checkpoint_db_path), check_same_thread=False)
checkpointer_sql = SqliteSaver(sqlite_conn)
```

自定义：扩展 AgentState

### 存储

store = InMemoryStore()
访问：@wrap_model_call中使用runtime.store | request.runtime.store访问数据并传入llm

### 上下文

在调用时传入的静态配置、上下文数据

```python
@dataclass
class UserContext:
    user_id: str
```

访问：runtime.context | request.runtime.context

### 中间件

作用：
在调用模型之前处理状态（例如消息裁剪、上下文注入）
修改或验证模型的响应（例如防护栏、内容过滤）
使用自定义逻辑处理工具执行错误
基于状态或上下文实现动态模型选择
添加自定义日志、监控或分析
中间件无缝集成到智能体的执行图中，允许您在关键点拦截和修改数据流，而无需更改核心智能体逻辑。

@before_agent,@after_agent
@before_model：调用前处理，参数 state: AgentState, runtime: Runtime
@after_model：调用后处理
@wrap_tool_call
@wrap_model_call
@dynamic_prompt
或实现AgentMiddleware类
预构建中间件：SummarizationMiddleware（摘要消息），HumanInTheLoopMiddleware（运行中人工介入），AnthropicPromptCachingMiddleware（缓存重复提示词），ModelCallLimitMiddleware（调用限制：thread_limit，run_limit, exit_behavior），ToolCallLimitMiddleware（工具限制：tool_name，thread_limit，run_limit）...

### 消息

```python
conversations = [
    {"role": "system", "content": "你是一个将英语翻译成法语的有用助手。"}, # SystemMessage
    {"role": "user", "content": "翻译：我喜欢编程。"}, # HumanMessage
    {"role": "assistant", "content": "J'adore la programmation."}, # AIMessage
]
```

### runtime

运行时信息，在tool中使用：
runtime.state
runtime.store
runtime.stream_writer # 流写入器

### Command

命令，在tool中使用：

```python
Command(
    "user_name": name,
    update={
        "messages": [RemoveMessage(id=REMOVE_ALL_MESSAGES)],
    }
)
```

### 输出格式

```python
from pydantic import BaseModel

# 自定义pydantic样式
class ContactInfo(BaseModel):
    name: str
    email: str
    phone: str

# 自定义TypedDict样式
from typing_extensions import TypedDict, Annotated

class MovieDict(TypedDict):
    """一部带有详细信息的电影。"""
    title: Annotated[str, ..., "电影标题"]
```

ToolStrategy(ContactInfo)
ProviderStrategy(ContactInfo)：更稳定，但要求模型支持--openai
type[StructuredResponseT]：模式类型（Schema type） - 会根据模型功能自动选择最佳策略

## agent

```python
agent = create_agent(
    model=my_llm, # 模型
    system_prompt="你是一个有帮助的助手。" # 系统提示
    tool=[search], # 工具
    middleware=[dynamic_model_selection, handle_tool_errors, user_role_prompt, CustomMiddleware()], # 中间件: 动态模型选择，工具错误处理，动态提示词，状态
    checkpointer=checkpointer, # 短记忆
    store=store # 存储
    response_format=ToolStrategy(ContactInfo), # 输出约束
    context_schema=UserContext, # 上下文
)
```

### 普通请求

```python
response = agent.invoke(
    {"messages": [{"role": "user", "content": "解释机器学习"}]},
    context=UserContext(user_id="user123"), # 上下文
    config={
        "run_name": "joke_generation",      # 此运行的自定义名称
        "tags": ["humor", "demo"],          # 用于分类的标签
        "metadata": {"user_id": "123"},     # 自定义元数据
        "callbacks": [my_callback_handler], # 回调处理程序
        "user_role": "expert",
        "recursion_limit": 2,               # 最大递归深度
    }
)
```

response.usage_metadata 为令牌使用数

### 批处理

```python
responses = agent.batch(["", "", ""], config={
        'max_concurrency': 5,  # 限制为 5 个并行调用
})
```

### 流式

```python
for chunk in agent.stream({
    "messages": [{"role": "user", "content": "搜索 AI 新闻并总结发现"}]
}, stream_mode="values"):
    for block in chunk.content_blocks:
        if block["type"] == "reasoning" and (reasoning := block.get("reasoning")): # 推理
            print(f"推理：{reasoning}")
        elif block["type"] == "tool_call_chunk":
            print(f"工具调用块：{block}")
        elif block["type"] == "text":
            print(block["text"])
        else:
            ...
    # 每个块包含该时间点的完整状态
    latest_message = chunk["messages"][-1]
    if latest_message.content:
        print(f"智能体：{latest_message.content}")
    elif latest_message.tool_calls:
        print(f"正在调用工具：{[tc['name'] for tc in latest_message.tool_calls]}")
```

# 优化

## 短期记忆优化

### 修剪消息

RemoveMessage 移除最前或最后n条消息

```python
@before_model
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """Keep only the last few messages to fit context window."""
    messages = state["messages"]

    if len(messages) <= 3:
        return None  # No changes needed

    first_msg = messages[0]
    recent_messages = messages[-3:] if len(messages) % 2 == 0 else messages[-4:]
    new_messages = [first_msg] + recent_messages

    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            *new_messages
        ]
    }
```

### 删除消息

RemoveMessage(id=REMOVE_ALL_MESSAGES) 删除全部消息；
RemoveMessage(id=123) 删除指定消息；

### 总结历史消息

```python
# 传入中间件
SummarizationMiddleware(
    model="xxx",
    max_tokens_before_summary=4000,  # Trigger summarization at 4000 tokens
    messages_to_keep=20,  # Keep last 20 messages after summary
)
```

### 自定义：消息过滤等
