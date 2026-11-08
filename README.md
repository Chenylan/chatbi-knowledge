# chatbi-knowledge

ChatBI 的知识语料仓库：表结构、指标口径、查询示例等，供 `database_schema_rag` 向量检索使用。
这里是文档的**唯一权威源**——改文档只改这里。

> 说明：向量索引不在 CI 里重建（DashScope 国内端点从 GitHub Actions 海外 runner 连不上），
> 改为**本地构建后提交**到应用仓库。流程见下文。

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

4. 在应用仓库 ChatBI 里，用这里作为语料**本地重建索引**：

   ```bash
   cd ~/python_projects/ChatBI
   python tools/ingest_chromadb.py \
     --docs-dir ~/python_projects/chatbi-knowledge/knowledge \
     --out chroma_schema_db
   ```

5. 把重建出的 `chroma_schema_db/` 提交并推送到应用仓库：

   ```bash
   git add chroma_schema_db && git commit -m "重建 schema 索引" && git push
   ```

## category 取值

| 取值 | 含义 |
|---|---|
| `schema` | 单表结构 / 字段定义 |
| `reference` | 概览、表关系、查询示例、指标口径等参考文档 |

缺失 `category` 的文件会被索引脚本跳过并告警。以后可按需扩展 `owner` / `version` / `expires_at` 等字段做版本与过期控制。
