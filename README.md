# 🤖 Hermes Agent 社区知识管理 Skills

> 让 AI 自动采集、整理、提炼社区讨论中的高价值知识 —— 告别"聊完就忘"。

本仓库包含两个用于飞书社区知识库管理的 [Hermes Agent](https://github.com/nousresearch/hermes) Skills（技能），基于真实生产环境打磨，已在中文社区持续运行。

## 📦 Skills 清单

### 1. 📰 社区日报采集 (community-daily-ingestion)

**一句话介绍**：每天自动从飞书群聊中采集原始记录，生成 6 大板块结构化的社区日报。

**能力**：
- 📥 自动拉取飞书群聊原始记录（JSON 格式）
- 📊 识别重要事件、技术分享、踩坑实录、热榜话题
- 📝 生成 6 板块日报：今日动态 / 干货分享 / 踩坑实录 / 热榜话题 / 群友原声 / 活跃 TOP5
- 📡 自动播报到飞书群，写入知识库 Wiki

**适用场景**：技术社区、学习小组、项目团队的每日知识沉淀。

### 2. ⛏️ 专题知识提炼 (topic-mining-synthesis)

**一句话介绍**：按专题关键词深度挖掘社区聊天记录，自动提炼成 Wiki 知识页面。

**能力**：
- 🔍 按关键词/专题在原始记录中检索匹配内容
- 🧠 利用子代理并行提炼核心经验（3-5 条/专题）
- 📚 自动创建/追加飞书 Wiki 页面，增量积累
- 🔄 每周自动运行，持续沉淀

**适用场景**：从碎片化讨论中提炼可复用的经验、最佳实践、避坑指南。

## 🚀 快速开始

### 前置条件

- [Hermes Agent](https://github.com/nousresearch/hermes) v0.8.0+ 已安装并配置
- 飞书机器人已接入 Hermes Gateway
- 飞书应用具备文档读取权限（`docx:document:readonly`）

### 安装步骤

```bash
# 1. 克隆本仓库
git clone https://github.com/liuxiaoqianglongxia/hermes-community-skills.git
cd hermes-community-skills

# 2. 将 skills 复制到 Hermes 技能目录
cp -r skills/* ~/.hermes/skills/

# 3. 重启 Hermes Gateway（如有活跃会话，请等待空闲后执行）
hermes gateway restart

# 4. 验证技能已加载
hermes chat
/skills list
```

### 配置日报

编辑 `~/.hermes/config.yaml`，添加定时任务：

```yaml
cron:
  - name: 中文社区日报
    schedule: "30 22 * * *"  # 每天 22:30 生成
    prompt: "执行 community-daily-ingestion 技能，生成今日社区日报"
```

### 配置专题提炼

创建专题配置文件 `~/.hermes/topic_config.yaml`：

```yaml
topics:
  - name: "飞书集成"
    keywords: ["飞书", "lark", "feishu", "webhook", "卡片"]
  - name: "AI Agent"
    keywords: ["agent", "子代理", "delegate", "工具调用"]
  - name: "部署运维"
    keywords: ["部署", "docker", "nginx", "端口", "重启"]
```

Cron 默认每周一 08:00 自动运行专题提炼。

## 📖 使用方式

### 手动触发日报

在飞书群或 Hermes CLI 中发送：

```
生成今天的社区日报
```

### 手动触发专题提炼

```
提炼本周的飞书集成专题
```

## 🏗️ 架构说明

```
飞书群聊 ──→ Hermes Gateway ──→ 子代理(并行) ──→ Wiki 知识库
                │
                ├── 日报采集 (每天)
                └── 专题提炼 (每周)
```

两个 Skill 共享同一套数据源（飞书群聊记录索引表），日报负责"广度采集"，提炼负责"深度加工"。

## 📢 关注我们

> 🎯 **麦尖 AI** 微信公众号
> 
> 一个教育从业者的 AI 实战记录。不写教程，只分享真实折腾经历。
> 
> 👉 微信搜索「**麦尖AI**」或扫描下方二维码关注
> 
> 我们在这里分享：
> - AI Agent 实战踩坑复盘
> - 自动化工作流搭建经验
> - 给教育/管理者的 AI 工具箱

## 📄 License

MIT

## 🙏 致谢

- [Hermes Agent](https://github.com/nousresearch/hermes) by [Nous Research](https://nousresearch.com)
- 中文社区每一位贡献讨论的成员
