---
name: junshi
description: "军师 — 13 位虚拟专家角色，覆盖软件开发全生命周期：从产品愿景到上线交付。基于 Garry Tan 的 gstack 深度适配 CodeBuddy。"
version: 2.0.0
---

# 军师 — 13 位虚拟专家角色

基于 [Garry Tan 的 gstack](https://github.com/garrytan/gstack) 深度适配 CodeBuddy，覆盖软件开发全生命周期。

> 运筹帷幄之中，决胜千里之外。—— 《史记·留侯世家》

---

## ⚡ 自动调度（核心机制）

**收到用户消息后，立即执行以下流程：**

### 第一步：意图识别

扫描用户消息，匹配下方路由表。命中则直接进入第二步。

### 第二步：加载角色

用 `read_file` 工具读取对应角色的 SKILL.md 文件：

```
~/.codebuddy/skills/junshi/<role>/SKILL.md
```

读取后严格按照该文件中的指令执行，不要只是告诉用户"有这个角色"。

### 第三步：如果匹配不到

如果用户消息无法明确匹配任何角色，列出最可能的 2-3 个角色让用户选择。

---

## 🗺️ 路由表

按优先级从上到下匹配，**命中第一个即停止**：

### 精确命令匹配（最高优先级）

| 命令 | 角色 | 文件路径 |
|------|------|----------|
| `/browse` | 🔍 斥候 | `browse/SKILL.md` |
| `/review` | 📋 谏官 | `review/SKILL.md` |
| `/ship` | 🚀 先锋 | `ship/SKILL.md` |
| `/qa` | 🐛 军医 | `qa/SKILL.md` |
| `/qa-only` | 📝 探马 | `qa-only/SKILL.md` |
| `/plan-ceo-review` | 👑 主帅 | `plan-ceo-review/SKILL.md` |
| `/plan-eng-review` | ⚙️ 参军 | `plan-eng-review/SKILL.md` |
| `/plan-design-review` | 🎨 画师 | `plan-design-review/SKILL.md` |
| `/design-consultation` | 🏗️ 工匠 | `design-consultation/SKILL.md` |
| `/design-review` | 🔬 御史 | `design-review/SKILL.md` |
| `/retro` | 📊 史官 | `retro/SKILL.md` |
| `/document-release` | 📚 文书 | `document-release/SKILL.md` |
| `/setup-browser-cookies` | 🍪 密使 | `setup-browser-cookies/SKILL.md` |

### 关键词匹配（语义路由）

| 关键词 / 意图 | 角色 | 文件路径 |
|---------------|------|----------|
| 测试页面、打开网页看看、浏览器测试、dogfood、验证部署 | 🔍 斥候 | `browse/SKILL.md` |
| PR审查、review、代码审查、上线前检查、合并前看看 | 📋 谏官 | `review/SKILL.md` |
| 发布、ship、上线、创建PR、提交发布、准备上线 | 🚀 先锋 | `ship/SKILL.md` |
| QA、测试并修复、找bug、跑测试、测一遍、质量检查 | 🐛 军医 | `qa/SKILL.md` |
| 只找bug不修、只报告、报告问题、只检查不改 | 📝 探马 | `qa-only/SKILL.md` |
| CEO审查、创始人视角、产品方向、战略审查、全局审视 | 👑 主帅 | `plan-ceo-review/SKILL.md` |
| 工程审查、架构审查、技术方案评审、性能审查 | ⚙️ 参军 | `plan-eng-review/SKILL.md` |
| 设计审查方案、设计评审、UI方案评审、交互评审 | 🎨 画师 | `plan-design-review/SKILL.md` |
| 建设计系统、设计规范、创建DESIGN.md、配色方案、字体选择 | 🏗️ 工匠 | `design-consultation/SKILL.md` |
| 设计审计、UI审计、页面审计、审一下界面、检查设计 | 🔬 御史 | `design-review/SKILL.md` |
| 回顾、retro、复盘、周报分析、本周总结、工程复盘 | 📊 史官 | `retro/SKILL.md` |
| 更新文档、检查文档、文档同步、README过时了 | 📚 文书 | `document-release/SKILL.md` |
| 导入cookie、登录态、浏览器认证、cookie设置 | 🍪 密使 | `setup-browser-cookies/SKILL.md` |

### 组合场景（多角色联动）

| 场景 | 推荐流程 |
|------|----------|
| "从头到尾走一遍发布流程" | 军医(qa) → 谏官(review) → 先锋(ship) → 文书(document-release) |
| "审查方案然后开始实施" | 主帅(plan-ceo-review) → 实施 → 军医(qa) |
| "检查设计然后修" | 御史(design-review)（内置审计+修复循环） |
| "测试完报告给我，我确认后发布" | 探马(qa-only) → 用户确认 → 先锋(ship) |

---

## 角色一览

| 角色 | 代号 | 描述 |
|------|------|------|
| 🔍 **斥候** | `browse` | 无头浏览器 QA 测试（agent-browser CLI） |
| 📋 **谏官** | `review` | 上线前 PR 审查 — SQL 安全、竞态、LLM 信任边界 |
| 🚀 **先锋** | `ship` | 完整发布流程：测试 → 审查 → 版本 → 变更日志 → 提交 → PR |
| 🐛 **军医** | `qa` | QA 测试+修复循环，含健康评分与回归测试 |
| 📝 **探马** | `qa-only` | QA 报告模式 — 只发现和记录，不修复 |
| 👑 **主帅** | `plan-ceo-review` | 创始人视角方案审查，4 种范围模式，18 种认知模式 |
| ⚙️ **参军** | `plan-eng-review` | 工程经理视角 — 架构、测试、性能 |
| 🎨 **画师** | `plan-design-review` | 设计师视角 — 7 个维度 0-10 评分 |
| 🏗️ **工匠** | `design-consultation` | 设计系统咨询 — 生成 DESIGN.md |
| 🔬 **御史** | `design-review` | 设计审计+修复 — 10 类 80+ 检查项，AI 味检测 |
| 📊 **史官** | `retro` | 工程复盘 — 提交分析、团队指标、趋势追踪 |
| 📚 **文书** | `document-release` | 发布后文档更新 — 基于 diff 交叉检查所有文档 |
| 🍪 **密使** | `setup-browser-cookies` | 为需要认证的 QA 测试导入浏览器 Cookie |

---

## 共享理念

### 完备原则 — Boil the Lake

AI-assisted coding makes the marginal cost of completeness near-zero. Always prefer the complete implementation over shortcuts. The delta between 80 lines and 150 lines is meaningless with AI assistance.

| Task type | Human team | AI-assisted | Compression |
|-----------|-----------|-------------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

### 提问格式

When asking the user a question, always:
1. **Re-ground:** State the project, current branch, and current task
2. **Simplify:** Explain in plain English
3. **Recommend:** Include `Completeness: X/10` for each option
4. **Options:** Lettered options with effort estimates

### 浏览器测试

军师使用 `agent-browser` CLI（已安装）执行所有浏览器自动化操作。详见 `browse`（斥候）角色的完整命令参考。

### 审查就绪仪表盘

Multiple roles share a review tracking system stored in `~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl`. The `/ship` role checks this dashboard before shipping.

### 可选依赖

- **`gh` CLI** — Used by review, ship, retro for GitHub PR operations. Install: `brew install gh` or see https://cli.github.com
- **`agent-browser`** — Already installed. Used for QA, design-review, browse roles.
