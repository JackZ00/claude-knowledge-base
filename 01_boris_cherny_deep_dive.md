# Boris Cherny 深度解析：Claude Code 创始人的完整使用方式

> **Boris Cherny**（[@bcherny](https://x.com/bcherny)）是 Claude Code 的创始人，2024 年 9 月将其作为个人项目创建，现任职于 Anthropic（@anthropicai）。他在 X/Threads 上多次详细分享自己及团队的使用方式。本文综合他的所有公开分享，呈现完整画像。

---

## 一、基本理念

Boris 的核心哲学只有一句话：

> "我的设置可能出乎意料地朴素。Claude Code 开箱即用就很好用，所以我个人不会过多自定义它。使用 Claude Code 没有唯一正确的方式——每个人的设置都不同，你应该自己实验找到最适合自己的方法。"

**最重要的数据点**：在分享这个帖子的前 30 天内，他对 Claude Code 仓库 **100% 的代码贡献都是由 Claude Code 本身写的**。

---

## 二、Boris 个人工作流（详细版）

### 2.1 Plan Mode 先行（最重要的单一习惯）

**操作方式**：
- 按 `Shift+Tab` **两次**进入 Plan Mode
- 所有任务在 Plan Mode 中开始，Claude 只读文件和回答问题，**不会做任何修改**

**完整流程**：

```
[Plan Mode] 探索阶段
↓ 告诉 Claude 目标，让它读取相关文件，充分理解上下文
↓
[Plan Mode] 规划阶段
↓ 让 Claude 生成详细实施计划（不写代码）
↓ 按 Ctrl+G 在编辑器中打开计划
↓ Boris 手动在计划文档里添加内联批注：
  - 纠正 Claude 的错误假设
  - 拒绝某种技术方案
  - 补充约束条件
  - 提供 Claude 没有的领域知识
↓
[Normal Mode + 自动接受编辑]
↓ Claude 对照计划执行，通常可以一次性完成
```

**什么时候跳过 Plan Mode**：
- 任务范围清晰且改动很小（如修复拼写错误、添加日志行）
- 如果你能用一句话描述这个 diff，就跳过规划

### 2.2 并行会话管理

Boris 同时维持大量并行工作流：

| 运行位置 | 数量 | 隔离方式 |
|----------|------|----------|
| 本地终端 | **5 个实例** | 5 个独立的 git checkout（同一仓库） |
| claude.ai/code 云端 | **5-10 个会话** | 云端隔离 VM |

**实践技巧**：
- 终端标签页标记为 1-5，便于快速切换
- 使用系统通知了解哪个 Claude 需要输入
- 每个 worktree 独立工作，互不干扰

### 2.3 模型选择

Boris 所有任务都使用 **Claude Opus（含扩展思考模式）**：

> "虽然它比 Sonnet 更大、更慢，但需要引导的次数更少，工具使用能力也更强，所以实际上几乎总是更快。"

从 Opus 4.5 开始，他认为这是他用过的最好的编程模型。

### 2.4 斜杠命令（自动化内循环）

```
.claude/commands/          ← 检入 Git，团队共享
├── commit-push-pr.md      ← 每天使用数十次
├── fix-issue.md
├── techdebt.md
└── ...
```

**Boris 的黄金法则**：每天重复超过一次的工作流都应该变成斜杠命令。

### 2.5 CLAUDE.md：团队共享知识库

**Boris 团队的实践方式**：
- 整个 Claude Code 仓库共享**一个** CLAUDE.md
- 检入 Git，整个团队每周贡献**多次**
- **核心规则**：每当 Claude 做错了什么，立即把正确做法加进去

### 2.6 验证机制（质量倍增器）

> "获得 Claude Code 优质输出**最重要**的一件事，就是给 Claude 一种验证其工作的方式。有了这个反馈循环，最终结果的质量会提升 **2-3 倍**。"

### 2.7 子代理（Subagents）

| 子代理名 | 功能 | 触发方式 |
|----------|------|----------|
| `code-simplifier` | 在 Claude 完成工作后自动简化代码 | 自动（PostToolUse）|
| `verify-app` | Claude Code 端到端测试 | 手动调用 |
| `security-reviewer` | 安全漏洞审查 | 手动调用 |

### 2.8 Hooks（钩子）

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{ "type": "command", "command": "prettier --write $FILE" }]
    }]
  }
}
```

---

## 三、Boris 团队的 10 个 Tips（2026 年 1 月）

### Tip 1：并行是团队最大的生产力提升
> "我们的头号建议是同时开启 3-5 个 git worktree，每个都运行自己的 Claude 会话。"

### Tip 2：遇到问题时重新规划，不要硬撑
- 让 Claude A 写计划，Claude B 以"staff engineer"身份审查
- 连续纠正两次还是不对，`/clear` 重新开始

### Tip 3：让 CLAUDE.md 自我学习
- 每个任务维护笔记目录，每次 PR 后更新
- "复利式工程"：AI 基于自己的失败来训练自己

### Tip 4：重复超过一次就做成 Skill
- `/techdebt`：找出重复代码
- `/context-sync`：从 Slack/GDrive/Asana/GitHub 同步上下文

### Tip 5：用 Subagent 处理计算，保持主上下文干净

### Tip 6：MCP 是团队协作的核心
- 连接 Figma 直接从设计实现功能
- 连接数据库直接查询

### Tip 7：Code Review 用独立 Claude 会话
- 新上下文审查，不会偏向刚写的代码

### Tip 8：截图验证 UI 变更

### Tip 9：GitHub Issues 工作流
```bash
/fix-issue 1234  # 给 Claude 一个 issue 号，完整处理到 PR
```

### Tip 10：用 Claude 学习新代码
- 生成 HTML 幻灯片讲解不熟悉的代码
- 画 ASCII 图解释新协议和代码架构

---

## 四、Boris 记录的重要功能时间线

| 时间 | 事件 |
|------|------|
| 2024-09 | Boris 将 Claude Code 作为个人项目创建 |
| 2025-06 | Claude Code SDK 正式支持 Python 和 TypeScript |
| 2025-08 | Matt Turck 采访：谈 Claude Code "意外走红" |
| 2025-10 | 发布 Claude Code **Plugins**（插件系统）|
| 2026-01 | 发布内置 **git worktree** 支持（多 Agent 并行）|
| 2026-03 | 发布 **Code Review** 功能（Agent 团队自动审查 PR）|

---

## 五、关键数据

- Anthropic 内部工程师人均代码产出 **同比提升 200%**
- 过去 30 天，Boris 对 Claude Code 仓库 **100% 的代码贡献**由 Claude Code 完成

---

*来源：[@bcherny on X](https://x.com/bcherny)、[Threads](https://www.threads.com/@boris_cherny)*
