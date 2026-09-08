# TinyStar (Trendex) · 全域商业洞察平台

> **Full-Domain Business Insight Platform for Enterprises**


TinyStar 是面向企业的全域商业洞察平台。基于多源公开产业信息与 AI 解析能力，完成**商机识别、市场监测、竞争分析、赛道趋势研判**，覆盖业务线索挖掘、前沿机会研究、行业复盘、企业战略规划等多类经营决策场景，助力企业精准捕捉市场机遇、预判行业变化、科学制定发展策略。

More than short-term business lead mining — TinyStar focuses on industry insight, frontier opportunity research and enterprise strategic decision-making, building an all-weather, enterprise-level business intelligence & AI analysis infrastructure.

---

## 目录

- [核心能力](#-核心能力)
- [在线演示](#-在线演示)
- [系统架构](#-系统架构)
- [技术栈](#%EF%B8%8F-技术栈)
- [快速开始](#-快速开始)
- [配置说明](#%EF%B8%8F-配置说明)
- [项目结构](#-项目结构)
- [API 一览](#-api-一览)
- [管理后台](#-管理后台)
- [安全与生产建议](#-安全与生产建议)
- [FAQ](#-faq)
- [许可协议](#-许可协议)

---

## ✨ 核心能力

| 能力 | 说明 |
|---|---|
| **智能商机识别** | 对海量产业信息进行结构化解析与智能筛选（分类 / 关键词 / 实体 / 情感），精准挖掘高价值商业线索 |
| **全域市场监测** | 持续追踪行业动态、产业政策、市场供需变化；多源 RSS/JSON 采集 + 定时调度，实现常态化情报预警 |
| **竞争格局分析** | 基于语义聚类将同一事件的多来源报道去重聚合，梳理行业主体动态、布局动作与市场行为，清晰研判竞品态势 |
| **赛道趋势研判** | 基于 Embedding + AI 数据分析行业热度、发展走向、增长潜力；提供热度趋势、词云、词项共现网络与关键词深度报告 |
| **全场景决策支撑** | 日报 / 周报 / 月报自动生成、专题时间轴复盘、新闻智能体自然语言检索，适配一线拓客、新兴赛道布局与中长期战略规划 |

### 功能模块

**新闻采集与处理**
- 通过 `data/news_sources.json` 配置多新闻源，支持启用状态、来源权重与地址维护；兼容 RSS/XML、JSON 接口与部分网页热点源。
- Crawl4AI / Playwright 正文补抓，支持动态页等待、超时、重试与并发控制；支持微博 Cookie、忽略域名、关注关键词过滤。
- 新闻源健康监控：最近抓取结果、失败次数、错误信息可视化。

**热点列表与语义搜索**
- 按热度/时间排序，支持分页、时间范围（`today` / `24h` / `7d` / `month` / 自定义区间）、分类、地区、来源筛选。
- 关键词搜索优先向量召回，结合文本匹配提升召回率。
- 新闻详情弹窗：摘要、来源、关键词、实体、情感、关联报道、相似新闻。
- 支持生成热点新闻图片与智能体新闻卡片图片（Pillow），便于分享归档。

**AI 分析与聚类**
- 自动生成 AI 摘要，正文不足时使用来源摘要兜底；自动补全分类、地区、情感、关键词、实体。
- Embedding 相似度 + AI 二次核验的事件级去重聚合（可配置相似度阈值与时间窗口）。
- 主力模型 / 备用模型自动切换，AI 路由可按功能（摘要、聚类、专题、报告、对话等 13 类任务）独立指定模型节点。

**专题追踪**
- 自动从近期高热新闻中发现候选事件簇，经 AI 审核生成专题；亦支持手动创建。
- 专题列表、详情、事件时间轴、相关新闻与趋势仪表盘。
- 支持刷新专题综述、刷新单节点摘要、专题匹配扫描（召回池、批次、质量等级等均可调）。

**报告与图表**
- 综合报告与关键词深度报告，可按时间、分类、地区、来源、样本量筛选。
- 来源分布、词云、情感分布、正负面关键词、热度趋势、词项共现网络（ECharts）。
- 日报/周报/月报缓存与历史管理；AI 报告流式输出；关键词深度报告围绕事件演变、观点光谱、风险机会与后续观察生成。

**新闻智能体**
- 基于 PydanticAI 的连续对话智能体，支持工具调用事件流（SSE）。
- 内置 13+ 工具：热点查询、语义搜索、新闻/专题详情、报告数据、创建关键词报告、创建事件专题、词项趋势、网页搜索、正文抓取、图片生成等。
- 管理端可添加自定义 HTTP 工具（GET/POST、URL/Header/Body 模板、结果路径截取、返回体压缩）。


---

## 🏗 系统架构

```text
                    ┌────────────────────────────────────────────────┐
                    │              定时调度 (pipeline_service)         │
                    └────────────────────────────────────────────────┘
 新闻源配置                │                                        │
 (news_sources.json) ──▶ 抓取层 ──▶ 正文补抓 ──▶ 语义聚类 ──▶ AI 分析 ──▶ 专题发现
                          aiohttp     Crawl4AI     Embedding   摘要/分类      │
                          RSS/JSON    Playwright   相似度+核验  情感/关键词    │
                                                                              ▼
   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐
   │  热点列表/搜索  │   │  专题追踪     │   │  报告与图表    │   │  新闻智能体问答    │
   └──────┬───────┘   └──────┬───────┘   └──────┬───────┘   └────────┬─────────┘
          └──────────────────┴───── FastAPI REST + SSE ───────┬───────┘
                                                              ▼
                                        SQLite (默认) / PostgreSQL (asyncpg)
```

**数据流水线**：抓取 → 清洗 → 正文补抓（轻量抓取 → Crawl4AI → Playwright 三级兜底）→ 标题净化 → Embedding 向量化 → 聚类去重 → 摘要/分类/情感/实体分析 → 专题匹配与时间轴 → 报告生成。全流程由 `pipeline_service` 按可配置间隔（默认 30 分钟）自动调度，也可在管理后台手动触发。

---

## 🛠 技术栈

| 层级 | 技术 |
|---|---|
| Web 框架 | FastAPI、Starlette、Uvicorn、Jinja2 |
| 数据库 | SQLAlchemy 2.x (Async)，默认 SQLite，可切换 PostgreSQL (asyncpg) |
| AI 接入 | OpenAI-compatible API（`openai` SDK + `pydantic-ai`） |
| 向量能力 | Embedding API（示例使用 SiliconFlow `BAAI/bge-large-zh-v1.5`） |
| 爬虫 | aiohttp、BeautifulSoup、Crawl4AI、Playwright (Chromium) |
| 前端 | 原生 JS + ECharts + Graphology/Sigma（关系图谱）+ CodeMirror |
| 图片生成 | Pillow |
| 部署 | Docker / Docker Compose |

---

## 🚀 快速开始

### 方式一：Docker Compose（推荐）

**1. 准备目录与配置**

```bash
mkdir -p data
# 参考仓库示例文件：
#   docker/data/config.yaml       -> data/config.yaml
#   docker/data/news_sources.json -> data/data/news_sources.json
```

编辑 `data/config.yaml`，至少填写：

- `DATABASE_URL`：数据库连接（默认 SQLite）
- `SILICONFLOW_API_KEY`：Embedding 服务密钥
- `MAIN_AI_API_KEY` / `MAIN_AI_BASE_URL` / `MAIN_AI_MODEL`：主力生成模型
- `BACKUP_AI_*`：备用生成模型（可选，但强烈建议配置）

**2. 启动**

```bash
cd docker
docker compose up -d
```

**3. 访问**

- 首页：<http://localhost:8193>
- 管理后台：<http://localhost:8193/admin>（初始密码由环境变量 `ADMIN_PASSWORD` 控制，请务必修改默认值 `admin123456`）

### 方式二：源码运行

```bash
# Python 3.11+
pip install -r requirements.txt
python -m playwright install chromium

# 使用默认 config.yaml 启动（可用 -c 指定其他配置路径）
python main.py
# 或
uvicorn main:app --host 0.0.0.0 --port 8193
```

启动后可登录管理后台在线调整配置、新闻源与提示词；修改 `config.yaml` 保存后服务会自动重启加载。

---

## ⚙️ 配置说明

主配置文件为根目录 `config.yaml`（可用 `--config` / `-c` 或环境变量 `TRENDSONAR_CONFIG` 指定路径）。关键配置组：

### AI 模型

```yaml
SILICONFLOW_API_KEY: "sk-..."        # Embedding & Rerank
EMBEDDING_MODEL: BAAI/bge-large-zh-v1.5

MAIN_AI_API_KEY: "sk-..."            # 主力生成模型（摘要/情感/关键词/对话）
MAIN_AI_BASE_URL: https://api.siliconflow.cn/v1
MAIN_AI_MODEL: THUDM/GLM-4-9B-0414
MAIN_AI_CONCURRENCY: 10

BACKUP_AI_API_KEY: "sk-..."          # 备用模型（聚类/专题/报告，主力失败时自动切换）
BACKUP_AI_MODEL: deepseek-ai/DeepSeek-V3.2
```

任何 OpenAI-compatible 服务均可（SiliconFlow、DeepSeek、ModelScope、自建 vLLM 等）。

### AI 路由

按功能指定模型节点（`main` / `backup`）：

```yaml
AI_ROUTE:
  SUMMARY: "main"          # 新闻摘要
  SENTIMENT: "main"        # 情感分析
  KEYWORDS: "main"         # 关键词/实体
  CLUSTERING: "backup"     # 聚类去重
  TOPIC_NAME: "backup"     # 专题命名
  TOPIC_EVAL: "backup"     # 专题质量评估
  TOPIC_MATCH: "backup"    # 专题匹配核验
  TOPIC_TIMELINE: "backup" # 时间轴事件生成
  TOPIC_OVERVIEW: "backup" # 专题综述
  TOPIC_INITIAL_SUMMARY: "main"
  REPORT: "backup"         # 日报/周报
  CHAT: "main"            # 智能体对话
```

### 采集与算法调优（节选）

| 参数 | 默认 | 说明 |
|---|---|---|
| `CRAWLER_CONCURRENCY` | 2 | 浏览器正文抓取并发（控制内存峰值） |
| `CRAWLER_FETCH_TIMEOUT_SECONDS` | 45 | 单次正文抓取硬超时 |
| `CLUSTERING_THRESHOLD` | 0.72 | 聚类相似度阈值（越高越严格） |
| `CLUSTERING_TIME_WINDOW_HOURS` | 18 | 聚类时间窗口 |
| `SCHEDULE_INTERVAL_MINUTES` | 30 | 自动调度间隔 |
| `AUTO_SUMMARY_TOP_N` | 30 | 每日自动摘要条数 |
| `AUTO_ANALYSIS_TOP_N` | 200 | 每日自动深度分析条数 |
| `ANALYSIS_BATCH_SIZE` | 20 | AI 分析批次大小 |
| `WEIBO_COOKIE` | — | 微博详情抓取 Cookie（可选） |

完整参数及中文注释见 `config.yaml`，均可在管理后台在线修改。

---

## 📁 项目结构

```text
tinystar/
├── app/
│   ├── api/              # FastAPI 路由（news / topics / reports / graph / system / prompts）
│   ├── core/             # 配置、数据库、日志、提示词默认值
│   ├── models/           # SQLAlchemy 模型（News / Topic / Report / ClusteringHistory）
│   ├── schemas/          # Pydantic 请求/响应模型
│   ├── services/         # 业务服务：pipeline（编排）、crawler、cluster、ai、topic、
│   │                     #   report、agent、graph、admin、task_manager、source_health 等
│   └── utils/            # 检索、图片、重试、配置读写等通用能力
├── data/                 # 运行数据：news_sources.json、SQLite 库、调度状态
├── docker/               # Docker Compose 示例与容器内配置
├── docs/images/          # 文档截图
├── static/               # 前端静态资源（JS / CSS / 图表库）
├── templates/            # Jinja2 页面模板（首页/专题/报告/图谱/管理后台）
├── main.py               # 应用入口（生命周期、页面路由、管理员登录）
├── config.yaml           # 主配置文件
└── requirements.txt      # Python 依赖
```

---

## 🔌 API 一览

REST API 挂载于 `/api` 前缀，管理端点需管理员 Cookie 鉴权。主要端点：

| 模块 | 端点（节选） |
|---|---|
| 新闻 | `GET /api/news/sources` `GET /api/news` `GET /api/news/top` `GET /api/news/{id}` `GET /api/news/{id}/similar` `POST /api/generate_summary/{id}` `GET /api/news_image` |
| 专题 | `GET /api/topics/list` `GET /api/topics/{id}` `GET /api/topics/{id}/trends` `POST /api/topics/manual_create` `PATCH /api/topics/{id}` `POST /api/topics/{id}/regenerate_overview` |
| 报告 | `GET /api/recent` `GET /api/history` `GET /api/load/{id}` `GET /api/analysis` `GET /api/chart-data` `GET /api/term-analysis` `POST /api/generate` `GET /api/stream_ai`（SSE 流式） |
| 图谱 | `GET /api/graph/overview` `GET /api/graph/expand` `GET /api/graph/node/{term}` |
| 系统 | `GET /api/app_info` `POST /api/trigger_crawl` `GET /api/chat` `GET /api/agent/chat`（智能体 SSE） |
| 管理 | `GET/PUT /api/admin/config` `POST /api/admin/ai/test` `GET/PUT /api/admin/news_sources` `GET /api/admin/tasks` `GET /api/admin/logs` `POST /api/admin/agent_tools/custom` 等 |

启动后访问 <http://localhost:8193/docs> 查看完整 OpenAPI 文档。

---

## 🛡 管理后台

后台地址 `/admin`，管理员密码通过环境变量 `ADMIN_PASSWORD` 或 `.env` 设置，登录含失败锁定机制。支持：

- 在线维护运行配置、新闻源（卡片式增删改测）、提示词、智能体自定义工具
- 查看后台任务状态、当天内存日志与历史日志文件，手动触发抓取/分析/历史数据补全
- 测试 Embedding、主力与备用模型连通性
- 修改 `config.yaml` 后自动重启服务加载新配置

---

## 🔒 安全与生产建议

1. **务必修改 `ADMIN_PASSWORD` 默认值**（`admin123456`）。
2. API Key 等敏感信息不要提交到公开仓库；生产环境建议通过环境变量或挂载的私有配置注入。
3. 智能体的网页抓取与自定义 HTTP 工具已内置 SSRF 基础校验（拦截本机、内网与云元数据地址），自建部署仍建议置于内网或加反向鉴权层。
4. 对外公开部署时，建议在反向代理层（Nginx/Caddy）追加限流与访问控制。
5. AI 分析结果依赖新闻源质量、模型能力与数据积累时间，请作为辅助分析工具使用，**重要结论应回看原文核验**。
6. PostgreSQL 适合规模化部署；SQLite 适合个人/小团队，注意定期备份数据目录。

---

## ❓ FAQ

**Q: 免费模型够用吗？**
主力模型可用 SiliconFlow 免费 GLM-4-9B；聚类/专题/报告建议配置能力更强的备用模型（如 DeepSeek），专题和报告质量差距明显。

**Q: 抓取不到正文？**
动态页可调大 `CRAWLER_DYNAMIC_WAIT_SECONDS`（知乎/头条建议 5-8）；微博详情需更新 `WEIBO_COOKIE`；在管理后台查看新闻源健康状态定位问题。

**Q: 聚类把不同事件合并了 / 同一事件没合并？**
调低/调高 `CLUSTERING_THRESHOLD`（默认 0.72），并检查 `CLUSTERING_TIME_WINDOW_HOURS` 时间窗口。

**Q: 如何切换 PostgreSQL？**
`DATABASE_URL: postgresql+asyncpg://user:pass@host:5432/db`，重启即自动建表。

---

## 📄 许可协议

见 [LICENSE](LICENSE)。

---

<p align="center">TinyStar · 让每一条产业信号都成为决策依据</p>
