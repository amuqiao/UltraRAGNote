# UltraRAG 检索模块工作流程

## 检索模块高层次流程图

```mermaid
flowchart TD
    subgraph 初始化阶段
        A[配置参数输入<br>backend/model_path等] --> B[Retriever初始化<br>retriever_init]
        B --> C{选择后端类型}
        C -->|infinity| D1[加载infinity模型]
        C -->|sentence_transformers| D2[加载sentence_transformers模型]
        C -->|openai| D3[配置OpenAI API]
        C -->|bm25| D4[初始化BM25模型]
        D1 --> E[设备配置<br>GPU/CPU]
        D2 --> E
        D3 --> E
        D4 --> E
        E --> F[加载语料数据<br>JSONL文件解析]
        F --> G[初始化索引<br>faiss/BM25等]
    end

    subgraph 嵌入生成与索引构建
        G --> H[retriever_embed<br>文档向量化]
        H --> I{嵌入类型}
        I -->|文本嵌入| J1[文本向量生成]
        I -->|图像嵌入| J2[图像向量生成]
        J1 --> K[retriever_index<br>构建向量索引]
        J2 --> K
        G --> L[bm25_index<br>构建BM25索引]
    end

    subgraph 检索查询阶段
        M[用户查询输入] --> N{选择检索方式}
        N -->|本地向量检索| O1[retriever_search<br>向量相似度搜索]
        N -->|ColBERT检索| O2[retriever_search_colbert_maxsim<br>多向量模型检索]
        N -->|BM25检索| O3[bm25_search<br>关键词检索]
        N -->|远程检索| O4[retriever_deploy_search<br>API调用]
        N -->|外部搜索引擎| O5{选择搜索引擎}
        O5 -->|EXA| P1[retriever_exa_search]
        O5 -->|Tavily| P2[retriever_tavily_search]
        O5 -->|智谱AI| P3[retriever_zhipuai_search]
        O1 --> Q[返回检索结果<br>ret_psg]
        O2 --> Q
        O3 --> Q
        O4 --> Q
        P1 --> Q
        P2 --> Q
        P3 --> Q
    end

    subgraph 并行处理
        R[批量查询] --> S[_parallel_search<br>并发查询处理]
        S --> Q
    end
```

## 核心功能模块说明

### 1. 初始化模块 (Retriever)
- **功能**: 初始化检索系统，配置后端、加载模型和语料
- **支持的后端**: infinity、sentence_transformers、openai、bm25
- **关键配置**: 设备选择(GPU/CPU)、模型路径、语料路径

### 2. 嵌入生成模块
- **retriever_embed**: 将文本/图像转换为向量表示
- **支持**: 文本嵌入、图像嵌入、多向量模型(ColBERT/ColPali)

### 3. 索引构建模块
- **retriever_index**: 构建向量索引(faiss)
- **bm25_index**: 构建BM25关键词索引

### 4. 检索查询模块
- **本地检索**: 向量相似度搜索、ColBERT多向量检索、BM25关键词检索
- **远程检索**: 通过API调用远程检索服务
- **外部搜索引擎**: 集成EXA、Tavily、智谱AI等外部搜索服务

### 5. 并行处理模块
- **_parallel_search**: 支持批量查询的并发处理，提高检索效率

## 工作流程说明

1. **初始化流程**: 根据配置参数选择后端，加载相应模型，配置计算设备，加载语料数据，初始化索引系统
2. **向量化流程**: 将文档内容转换为向量表示，支持文本和图像两种类型
3. **索引流程**: 构建向量索引或BM25索引，为后续检索做准备
4. **检索流程**: 根据用户查询，选择合适的检索方式，返回最相关的文档段落
5. **并行处理**: 对批量查询进行并发处理，提高系统吞吐量

这个流程图展示了UltraRAG检索模块的核心工作流程，包括初始化、嵌入生成、索引构建和检索查询等关键环节，同时涵盖了多种检索后端和检索方式的集成。