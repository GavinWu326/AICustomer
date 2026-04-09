# AICustomer

<div align="center">

智能 AI 客服后端，基于 FastAPI + OpenAI + RAG，支持知识库同步、搜索提示、会话历史与反馈闭环。

<p>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/FastAPI-API-009688?style=flat-square&logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/OpenAI-RAG-412991?style=flat-square&logo=openai&logoColor=white" alt="OpenAI">
  <img src="https://img.shields.io/badge/ChromaDB-Vector_Search-F97316?style=flat-square" alt="ChromaDB">
  <img src="https://img.shields.io/badge/Redis-Session_Cache-DC382D?style=flat-square&logo=redis&logoColor=white" alt="Redis">
  <img src="https://img.shields.io/badge/Strapi-Knowledge_Base-4945FF?style=flat-square&logo=strapi&logoColor=white" alt="Strapi">
</p>

</div>

## 项目概览

`AICustomer` 是一个面向业务知识库场景的 AI 客服服务端项目。它将 Strapi 中的知识内容同步到本地向量库，通过 RAG 生成更贴近业务语境的回答，并配合 Redis、搜索提示和反馈回流形成完整的客服闭环。

适合用于：

- 软件产品内嵌客服问答
- FAQ / 帮助中心智能检索
- 带知识库同步能力的企业客服系统
- 需要保存会话与反馈数据的 AI 助手后端

## 核心能力

| 能力 | 说明 |
| --- | --- |
| RAG 智能问答 | 基于知识库检索结果生成回答，减少纯大模型幻觉 |
| 知识库同步 | 支持启动时全量构建与接口触发的增量 / 全量更新 |
| 搜索提示 | 根据用户输入返回问题补全建议，降低提问门槛 |
| 会话历史 | 聊天记录写入 Redis，并异步回传 Strapi |
| 反馈闭环 | 接收点赞 / 点踩反馈，关联会话上下文存档 |
| 定时任务 | 默认每 30 分钟执行一次知识库更新任务 |

## 架构示意

![系统架构图](pic/架构图.png)

![UML 设计图](pic/UML.png)

## 技术栈

| 分类 | 方案 |
| --- | --- |
| Web 框架 | FastAPI、Uvicorn |
| AI 能力 | OpenAI API |
| 检索层 | ChromaDB、Jieba |
| 缓存层 | Redis |
| CMS / 内容源 | Strapi |
| 调度 | `schedule`、APScheduler |
| 数据建模 | Pydantic |

## 项目结构

```text
AICustomer
├── app
│   ├── api
│   │   └── routes.py
│   ├── core
│   │   └── config.py
│   ├── models
│   │   └── schemas.py
│   ├── services
│   │   ├── hint_service.py
│   │   ├── openai_service.py
│   │   ├── rag_service.py
│   │   ├── redis_service.py
│   │   ├── scheduler_service.py
│   │   └── strapi_service.py
│   └── main.py
├── pic
├── requirements.txt
└── run_app.py
```

## 快速开始

### 1. 安装依赖

```bash
git clone https://github.com/GavinWu326/AICustomer.git
cd AICustomer
pip install -r requirements.txt
```

### 2. 配置环境变量

新建 `.env` 文件，至少补齐以下配置：

```bash
APP_NAME=AICustomer

OPENAI_API_KEY=your_openai_api_key
OPENAI_API_URL=your_openai_api_url
OPENAI_AUTH_KEY=your_openai_auth_key

REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=

STRAPI_API_URL=your_strapi_api_url
STRAPI_API_TOKEN=your_strapi_api_token

LOCAL_STRAPI_API_URL=http://localhost:1337/
LOCAL_STRAPI_API_TOKEN=

DEBUG_MODE=false
SKIP_STRAPI_FETCH=false
SKIP_CHROMA_UPDATE=false
CLEAR_CHROMA_ON_STARTUP=false
```

### 3. 启动服务

```bash
python run_app.py
```

如果需要在启动前清空现有 ChromaDB 数据：

```bash
python run_app.py --clear-db
```

也可以直接使用 Uvicorn：

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

启动后可访问：

- `http://localhost:8000/`
- `http://localhost:8000/docs`

## 配置说明

| 环境变量 | 必填 | 说明 |
| --- | --- | --- |
| `OPENAI_API_KEY` | 是 | OpenAI 接口密钥 |
| `OPENAI_API_URL` | 是 | OpenAI 或兼容接口地址 |
| `OPENAI_AUTH_KEY` | 否 | 额外鉴权字段，默认已内置 |
| `REDIS_HOST` / `REDIS_PORT` / `REDIS_DB` | 是 | Redis 连接配置 |
| `REDIS_PASSWORD` | 否 | Redis 密码 |
| `STRAPI_API_URL` | 是 | 线上 Strapi 内容源地址 |
| `STRAPI_API_TOKEN` | 否 | Strapi 访问令牌 |
| `LOCAL_STRAPI_API_URL` | 否 | 本地 Strapi 地址 |
| `LOCAL_STRAPI_API_TOKEN` | 否 | 本地 Strapi 访问令牌 |
| `DEBUG_MODE` | 否 | 是否启用调试模式 |
| `SKIP_STRAPI_FETCH` | 否 | 启动时跳过 Strapi 抓取 |
| `SKIP_CHROMA_UPDATE` | 否 | 启动时跳过向量库更新 |
| `CLEAR_CHROMA_ON_STARTUP` | 否 | 启动时清空 ChromaDB |

## API 一览

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| `POST` | `/chat` | 发起对话，返回 AI 回复 |
| `POST` | `/searchHint` | 获取输入补全建议 |
| `POST` | `/feedback` | 提交用户满意度反馈 |
| `POST` | `/update-knowledge` | 触发知识库增量更新 |
| `POST` | `/update-knowledge/full` | 触发知识库全量重建 |
| `POST` | `/refresh-search-hints` | 刷新搜索提示列表 |
| `GET` | `/scheduler-jobs` | 查看当前调度任务状态 |
| `GET` | `/` | 基础欢迎接口 |

### 请求示例

```http
POST /chat
Content-Type: application/json

{
  "query": "如何重置密码？",
  "session_id": "session-demo-001"
}
```

```http
POST /searchHint
Content-Type: application/json

{
  "query": "如何导",
  "limit": 10
}
```

```http
POST /feedback
Content-Type: application/json

{
  "satisfaction": "good",
  "tag": "answered",
  "commit": "问题已解决",
  "session_id": "session-demo-001",
  "feedback_id": "feedback-demo-001"
}
```

## 运行流程

服务启动时会按顺序执行以下动作：

1. 根据环境变量决定是否清空 ChromaDB。
2. 从 Strapi 拉取知识内容并生成本地 JSON 数据。
3. 将解析后的知识写入 ChromaDB 向量库。
4. 启动调度线程并初始化搜索提示服务。

这意味着它不仅是一个对话接口，也是一个自带知识同步与运维入口的客服后端。

## License

This project is licensed under the [MIT License](LICENSE).
