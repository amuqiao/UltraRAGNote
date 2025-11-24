# UltraRAG 数据处理与文档解析工作流程

```mermaid
flowchart TD
    subgraph 输入层
        A[原始数据/文档输入] --> B{输入类型判断}
        B -->|单个文件| C[单文件处理]
        B -->|目录| D[遍历目录文件]
        D --> C
    end

    subgraph 文档处理层
        C --> E{文档类型判断}
        
        %% 文本处理分支
        E -->|文本文件| F[文本文件处理]
        F --> G[文本提取与标准化]
        
        %% PDF处理分支
        E -->|PDF文档| H{PDF处理模式}
        H -->|文本提取| I[PDF文本提取]
        I --> J[页面文本合并]
        H -->|图像提取| K[PDF图像渲染]
        K --> L[图像保存与验证]
        H -->|高级解析| M[MinerU解析]
        M --> N[Markdown与图像提取]
        
        E -->|不支持格式| O[跳过处理]
    end

    subgraph 数据结构化层
        G --> P[构建文本条目]
        J --> P
        N --> P
        L --> Q[构建图像条目]
    end

    subgraph 分块处理层
        P --> R[文档分块处理]
        R --> S{分块策略选择}
        S -->|Token分块| T[基于Token的分块]
        S -->|Sentence分块| U[基于句子的分块]
        S -->|Recursive分块| V[递归分块]
    end

    subgraph 输出存储层
        P --> W[保存文本语料库JSONL]
        Q --> X[保存图像语料库JSONL]
        T --> Y[保存分块结果JSONL]
        U --> Y
        V --> Y
    end

    %% 添加连接关系
    style 输入层 fill:#f9f,stroke:#333,stroke-width:2px
    style 文档处理层 fill:#bbf,stroke:#333,stroke-width:2px
    style 数据结构化层 fill:#bfb,stroke:#333,stroke-width:2px
    style 分块处理层 fill:#fbb,stroke:#333,stroke-width:2px
    style 输出存储层 fill:#ffb,stroke:#333,stroke-width:2px
```

## 流程说明

### 1. 输入层
- 支持单个文件或整个目录作为输入源
- 自动遍历目录中的所有文件进行处理

### 2. 文档处理层
- 根据文件类型选择不同的处理分支
- 文本文件(.txt, .md)直接读取并标准化
- PDF文档支持三种处理模式：
  - 文本提取：使用pymupdf提取文本内容
  - 图像提取：渲染PDF页面为图像并保存
  - 高级解析：使用MinerU工具进行深度解析

### 3. 数据结构化层
- 将提取的内容转换为统一的数据条目格式
- 文本条目包含ID、标题和内容
- 图像条目包含ID、图像ID和图像路径

### 4. 分块处理层
- 对文本内容进行分块以优化后续检索
- 支持三种分块策略：
  - Token分块：基于token数量进行分块
  - Sentence分块：基于句子边界进行分块
  - Recursive分块：使用递归策略进行智能分块

### 5. 输出存储层
- 将处理结果保存为JSONL格式，便于后续处理
- 分别保存文本语料库、图像语料库和分块结果