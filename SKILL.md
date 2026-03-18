---
name: junshi
description: "军师 — 13 位虚拟专家角色，覆盖软件开发全生命周期：从产品愿景到上线交付。基于 Garry Tan 的 gstack 深度适配 CodeBuddy。"
version: 2.0.0
---

# 军师 — 13 位虚拟专家角色

基于 [Garry Tan 的 gstack](https://github.com/garrytan/gstack) 深度适配 CodeBuddy，覆盖软件开发全生命周期。

> 运筹帷幄之中，决胜千里之外。—— 《史记·留侯世家》

## Quick Start

告诉 CodeBuddy 你需要哪个角色。每个角色对应 `~/.codebuddy/skills/gstack/<role>/SKILL.md` 中的独立指令集。

## 角色一览 / Roles

| 角色 | 代号 | 触发 | 描述 |
|------|------|------|------|
| 🔍 **斥候** | `browse` | `/browse`, 测试页面 | 无头浏览器 QA 测试（agent-browser CLI） |
| 📋 **谏官** | `review` | `/review`, PR 审查 | 上线前 PR 审查 — SQL 安全、竞态、LLM 信任边界 |
| 🚀 **先锋** | `ship` | `/ship`, 发布上线 | 完整发布流程：测试 → 审查 → 版本 → 变更日志 → 提交 → PR |
| 🐛 **军医** | `qa` | `/qa`, 找 bug 并修复 | QA 测试+修复循环，含健康评分与回归测试 |
| 📝 **探马** | `qa-only` | `/qa-only`, 只报告 bug | QA 报告模式 — 只发现和记录，不修复 |
| 👑 **主帅** | `plan-ceo-review` | `/plan-ceo-review`, CEO 审查 | 创始人视角方案审查，4 种范围模式，18 种认知模式 |
| ⚙️ **参军** | `plan-eng-review` | `/plan-eng-review`, 工程审查 | 工程经理视角 — 架构、测试、性能 |
| 🎨 **画师** | `plan-design-review` | `/plan-design-review`, 设计审查 | 设计师视角 — 7 个维度 0-10 评分 |
| 🏗️ **工匠** | `design-consultation` | `/design-consultation`, 设计系统 | 设计系统咨询 — 生成 DESIGN.md |
| 🔬 **御史** | `design-review` | `/design-review`, 设计审计 | 设计审计+修复 — 10 类 80+ 检查项，AI 味检测 |
| 📊 **史官** | `retro` | `/retro`, 周回顾 | 工程复盘 — 提交分析、团队指标、趋势追踪 |
| 📚 **文书** | `document-release` | `/document-release`, 更新文档 | 发布后文档更新 — 基于 diff 交叉检查所有文档 |
| 🍪 **密使** | `setup-browser-cookies` | `/setup-browser-cookies`, 导入 Cookie | 为需要认证的 QA 测试导入浏览器 Cookie |

## 使用方法 / How to Use

调用某个角色时，读取其 SKILL.md：

```
Read ~/.codebuddy/skills/gstack/<role>/SKILL.md
```

然后按照文件中的指令执行。

## 共享理念 / Shared Concepts

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

### 提问格式 / User Question Format

When asking the user a question, always:
1. **Re-ground:** State the project, current branch, and current task
2. **Simplify:** Explain in plain English
3. **Recommend:** Include `Completeness: X/10` for each option
4. **Options:** Lettered options with effort estimates

### 浏览器测试 / Browser Testing

军师使用 `agent-browser` CLI（已安装）执行所有浏览器自动化操作。详见 `browse`（斥候）角色的完整命令参考。

### 审查就绪仪表盘 / Review Readiness Dashboard

Multiple roles share a review tracking system stored in `~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl`. The `/ship` role checks this dashboard before shipping.

### 可选依赖 / Optional Dependencies

- **`gh` CLI** — Used by review, ship, retro for GitHub PR operations. Install: `brew install gh` or see https://cli.github.com
- **`agent-browser`** — Already installed. Used for QA, design-review, browse roles.
