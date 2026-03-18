# 军师 (junshi)

> 运筹帷幄之中，决胜千里之外。—— 《史记·留侯世家》

**13 位虚拟专家角色，覆盖软件开发全生命周期** — 从产品愿景到上线交付。

基于 [Garry Tan 的 gstack](https://github.com/garrytan/gstack) 深度适配 [CodeBuddy](https://www.codebuddy.ai/)。

## 安装

### 方式 1：npx skills（推荐）

```bash
# 安装全部 13 个角色
npx --yes skills add cvk0524/junshi --all

# 只安装特定角色
npx --yes skills add cvk0524/junshi --skill junshi-qa -g -y
```

### 方式 2：SkillHub CLI

```bash
skillhub install junshi --primary-download-url-template "https://你的域名/skills/{slug}.zip"
```

### 方式 3：手动安装

```bash
git clone https://github.com/cvk0524/junshi.git ~/.codebuddy/skills/junshi
```

## 角色一览

| 角色 | 代号 | 触发命令 | 描述 |
|------|------|----------|------|
| 🔍 **斥候** | `browse` | `/browse` | 无头浏览器 QA 测试（agent-browser CLI） |
| 📋 **谏官** | `review` | `/review` | 上线前 PR 审查 — SQL 安全、竞态、LLM 信任边界 |
| 🚀 **先锋** | `ship` | `/ship` | 完整发布流程：测试→审查→版本→变更日志→提交→PR |
| 🐛 **军医** | `qa` | `/qa` | QA 测试+修复循环，含健康评分与回归测试 |
| 📝 **探马** | `qa-only` | `/qa-only` | QA 报告模式 — 只发现和记录，不修复 |
| 👑 **主帅** | `plan-ceo-review` | `/plan-ceo-review` | 创始人/CEO 视角方案审查，4 种范围模式 |
| ⚙️ **参军** | `plan-eng-review` | `/plan-eng-review` | 工程经理视角 — 架构、测试、性能 |
| 🎨 **画师** | `plan-design-review` | `/plan-design-review` | 设计师视角 — 7 个维度 0-10 评分 |
| 🏗️ **工匠** | `design-consultation` | `/design-consultation` | 设计系统咨询 — 生成 DESIGN.md |
| 🔬 **御史** | `design-review` | `/design-review` | 设计审计+修复 — 10 类 80+ 检查项 |
| 📊 **史官** | `retro` | `/retro` | 工程复盘 — 提交分析、团队指标、趋势 |
| 📚 **文书** | `document-release` | `/document-release` | 发布后文档更新 — 基于 diff 交叉检查 |
| 🍪 **密使** | `setup-browser-cookies` | `/setup-browser-cookies` | 导入浏览器 Cookie |

## 使用方法

在 CodeBuddy 中直接对话：

```
帮我审查一下当前的 PR     → 自动调用 谏官（review）
跑一遍 QA 测试            → 自动调用 军医（qa）
准备发布                   → 自动调用 先锋（ship）
做一下周回顾               → 自动调用 史官（retro）
```

或者指定角色：

```
Read ~/.codebuddy/skills/junshi/qa/SKILL.md
```

## 依赖

| 依赖 | 必需? | 用途 | 安装 |
|------|-------|------|------|
| `agent-browser` | 推荐 | 浏览器自动化（斥候/军医/御史） | 通常已预装 |
| `gh` CLI | 推荐 | GitHub PR 操作（谏官/先锋/史官/文书） | `brew install gh` |

## 兼容性

- **CodeBuddy** — 完全适配 ✅
- **Claude Code** — 原版 gstack 请用 [garrytan/gstack](https://github.com/garrytan/gstack)
- **Cursor / Windsurf** — SKILL.md 协议通用，理论兼容

## 致谢

- [Garry Tan](https://github.com/garrytan) — gstack 原作者（Y Combinator CEO）
- 原始项目：https://github.com/garrytan/gstack

## 许可

基于 gstack 原项目许可。
