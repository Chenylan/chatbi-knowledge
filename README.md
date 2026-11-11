# chatbi-knowledge

ChatBI 的知识语料仓库：表结构、指标口径、查询示例等，供 `database_schema_rag` 向量检索使用。
这里是文档的**唯一权威源**——改文档只改这里。

> 说明：向量索引存于 **PostgreSQL + pgvector**，通过应用仓库 ChatBI 里的
> `tools/ingest_pgvector.py` 全量重建，**不再**产出 `chroma_schema_db/` 这类本地二进制目录，
> 也不再需要把索引当 git 产物提交。重建命令见下文。

## 目录结构

- `knowledge/*.md` — 进向量库的知识文档

## 如何新增 / 修改文档

1. 在 `knowledge/` 下新增或修改 `.md` 文件。
2. 文件**必须**带 YAML front-matter，至少包含 `category` 字段：

   ```markdown
   ---
   title: 文档标题
   category: schema
   ---

   正文……
   ```

3. 提交并推送到 `main`。

4. 在应用仓库 ChatBI 里，把这里的语料重建进 PG 向量集合（`example_collection`）：

   ```bash
   cd ~/python_projects/ChatBI
   # 先启动 PostgreSQL（需已启用 pgvector 扩展），并配置好连接参数
   # 用国内 DashScope key：OPENAI_API_BASE_URL 留空（默认国内端点）
   # 用国际版 DashScope key：必须先设成国际端点，否则会 401 invalid_api_key
   export OPENAI_API_BASE_URL=https://dashscope-intl.aliyuncs.com/compatible-mode/v1
   python tools/ingest_pgvector.py --docs-dir ~/python_projects/chatbi-knowledge/knowledge
   ```

5. 索引直接写进 PostgreSQL，**无需**再提交任何索引产物到 git。

## 关于 GitHub CI（可选）

默认流程是本地重建。若要在 GitHub Actions 里重建，需同时满足：

1. runner 能连到 PostgreSQL（pgvector），用 service container 或托管 PG；
2. 用**国际版** DashScope key，并显式设置
   `OPENAI_API_BASE_URL=https://dashscope-intl.aliyuncs.com/compatible-mode/v1`
   （国内端点从 GitHub 海外 runner 连不上；国际 key 打到国内端点会 401 invalid_api_key）。

## category 取值

| 取值 | 含义 |
|---|---|
| `schema` | 单表结构 / 字段定义 |
| `reference` | 概览、表关系、查询示例、指标口径等参考文档 |

缺失 `category` 的文件会被索引脚本跳过并告警。以后可按需扩展 `owner` / `version` / `expires_at` 等字段做版本与过期控制。
