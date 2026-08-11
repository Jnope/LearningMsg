兴业证券

# mcp

## 三个功能

tool工具暴露，resource资源管理，prompt-预定义

## mcp传参校验，错误处理：

Schema 强校验，结构化错误信息返回。

## mcp不同用户权限和安全保证

OAuth2 认证，RBAC权限拦截，human-in-the-Loop (HITL)审核，Allowlist与注册表，数据校验

# 大模型幻觉处理

RAG+混合检索，提高知识库，引用溯源；
思维链先思考再回答，Few-Shot少样本学习，自我反思与修正，降低Temperature;
语法与规则校验,人工介入。

# 自动任务规划ai agent 实现

思维链意图识别，ReAct循环任务拆解；
工具注册和调用（mcp/Function Calling）；
短记忆-上下文，长记忆-rag知识库；反思+结果校验。

# 上下文处理

持久化；筛选和动态加载；多 Agent 上下文隔离；长对话历史进行滚动摘要，长结果摘要。

java + ai ？
开源代码阅读习惯；
