---
name: topic-mining-synthesis
description: 社区知识专题提炼 (Topic Mining) - 基于索引表，按专题关键词深度挖掘原始记录并更新 Wiki 知识库。
layer: workflow
metadata:
  hermes:
    dependencies:
      - community-daily-ingestion
    tags:
      - mining
      - wiki
      - synthesis
version: 2.0.0
---

# 社区知识专题提炼 (Topic Mining)

## 核心定位
本技能是社区的"知识提炼厂"。
- **输入**: 本地索引表 (`~/.hermes/community_chat_records.json`) + 专题清单 (`~/.hermes/topic_config.yaml`)。
- **输出**: 更新/创建对应的飞书 Wiki 页面（沉淀下来的干货）。

## 📦 核心资产
1. **配置清单**: `~/.hermes/topic_config.yaml` (用户可编辑，定义要提炼的专题)
2. **提炼脚本**: `~/.hermes/scripts/topic_miner.py` (执行逻辑)
3. **定时任务**: Cron Job `社区专题提炼周报` (每周一 08:00 自动运行)

## 🛠️ 实战工作流 (Execution Workflow)

**核心痛点解决**：
- `lark-cli docs +export` 经常报错或不支持某些参数。
- **替代方案**：直接使用 `lark-cli api GET '/open-apis/docx/v1/documents/{doc_id}/raw_content'` 获取原始 Markdown 文本。
- **提炼引擎**：不要试图用纯 Python 脚本做 LLM 提炼。使用 `delegate_task` 并行分派子代理，利用子代理的 `grep` + `read_file` + `lark-cli` 能力，效率最高且不易卡死。

### 第一步：准备数据 (索引与导出)
1. 确保 `~/.hermes/community_chat_records.json` 包含目标日期的 `doc_id`。
2. 批量导出原始内容到本地临时目录（如 `/tmp/raw_docs/`）：
   ```python
   # Python 脚本示例：遍历 index 调用 raw_content API
   cmd = f"lark-cli api --as user GET '/open-apis/docx/v1/documents/{doc_id}/raw_content'"
   ```

### 第二步：并行提炼 (Delegate Task Pattern)
不要串行处理。使用 `delegate_task` 的 `tasks` 数组（最多 3 个并行）。
- **子代理指令**：
  1. `grep` 本地 `/tmp/raw_docs/*.md` 文件查找关键词。
  2. 阅读匹配上下文。
  3. 提炼核心经验（3-5 条）。
  4. 调用 `lark-cli docs +create --wiki-space <ID>` 创建/追加内容。

### 第三步：Wiki 维护策略
- **固定标题**：Wiki 页面标题固定（如 `📌 飞书集成经验沉淀库`），不要带日期。
- **追加模式**：每次提炼的内容在页面顶部追加一个新的二级标题（如 `## 🤖 AI 自动提炼更新 (2026-04-04 ~ 2026-04-17)`），保持历史可追溯。
    *   **增量追加**: 
        *   顶部添加 `## 🤖 AI 自动提炼更新 (YYYY-MM-DD ~ YYYY-MM-DD)`。
        *   如果该页面已存在，将新内容**追加**到文档末尾（不要覆盖旧内容）。
        *   如果不存在，新建页面到指定 Wiki Space (`7629199638691597254`)。

## 🚀 运维建议
1.  **频率**: 推荐**每周一次**（Cron 设置为每周一 08:00），积累一周数据后提炼，密度更高。
2.  **API 额度**: 使用 `raw_content` 接口只读云文档，不消耗飞书"消息列表"接口额度。
3.  **人工干预**: 脚本运行完后，用户可在飞书端人工审阅，修正提炼偏差。
