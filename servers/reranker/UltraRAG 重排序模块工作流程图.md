# UltraRAG 重排序模块工作流程图

```mermaid
flowchart TD
    subgraph 初始化阶段
        A[初始化请求] --> B[Reranker初始化]
        B --> C{选择后端类型}
        C -->|infinity| D1[加载AsyncEngineArray]
        C -->|sentence_transformers| D2[加载CrossEncoder]
        C -->|openai| D3[配置API参数]
        D1 --> E[初始化完成]
        D2 --> E
        D3 --> E
    end

    subgraph 重排序请求处理
        F[重排序请求] --> G[接收查询和候选文档]
        G --> H[参数验证]
        H --> I[格式化查询]
        I --> J{根据后端类型处理}
        
        J -->|infinity| K1[异步调用rerank方法]
        J -->|sentence_transformers| K2[调用rank方法]
        J -->|openai| K3[异步API请求]
        
        K1 --> L[获取top_k结果]
        K2 --> L
        K3 --> L
        L --> M[返回重排序文档]
    end

    subgraph 数据流向
        N[用户查询列表] --> G
        O[候选文档列表] --> G
        M --> P[重排序后的文档列表]
    end

    subgraph 关键处理步骤
        Q[批量处理] --> J
        R[并发控制] --> J
        S[错误处理] --> J
        T[设备管理] --> C
    end
```

## 重排序模块工作流程说明

### 初始化阶段
1. 模块通过 `reranker_init` 方法进行初始化，接收模型路径、后端配置、批量大小和GPU ID等参数
2. 根据指定的后端类型（infinity、sentence_transformers或openai），加载相应的模型或配置
3. 配置设备环境（CPU/GPU）和处理参数

### 重排序请求处理
1. 通过 `reranker_rerank` 方法接收查询列表和候选文档列表
2. 验证输入参数的有效性（查询和文档列表长度必须匹配）
3. 根据配置的后端类型，选择不同的处理路径：
   - **infinity**: 使用异步引擎进行重排序
   - **sentence_transformers**: 使用CrossEncoder进行重排序
   - **openai**: 调用外部API进行重排序
4. 对每个查询-文档对进行相关性评分，返回前top_k个最相关的文档

### 多后端支持
模块支持三种后端实现，适应不同的部署环境和性能需求：
- **infinity**: 高性能嵌入引擎，支持异步处理
- **sentence_transformers**: 基于Transformer的跨编码器，使用CPU/GPU处理
- **openai**: 基于外部API的重排序服务，支持并发控制

### 关键特性
- 异步处理：大部分操作使用异步模式，提高并发性能
- 批量处理：支持批量处理多个查询-文档对
- 并发控制：特别是在使用OpenAI后端时，通过信号量控制并发请求数
- 设备管理：根据配置选择CPU或GPU处理
- 错误处理：包含完整的参数验证和错误处理机制