# Claude 使用知识库

> 本知识库综合了 **Boris Cherny（Claude Code 创始人）** 的第一手分享、Anthropic 官方报告、Reddit/GitHub 社区真实经验，旨在帮助工程师系统性地掌握 Claude 的高效使用方法。

---

## 知识库结构

| 文件 | 内容 | 适合谁看 |
|------|------|----------|
| [01_boris_cherny_deep_dive.md](./01_boris_cherny_deep_dive.md) | Boris 完整工作流、团队 10 个 Tips | 所有人 |
| [02_scenario_strategies.md](./02_scenario_strategies.md) | 10 个开发场景的具体策略 | 日常开发 |
| [03_community_insights.md](./03_community_insights.md) | Reddit/GitHub 社区踩坑与经验 | 想了解真实反馈 |
| [04_official_reports.md](./04_official_reports.md) | Anthropic 官方报告与研究摘要 | 深度学习 |
| [05_claude_md_guide.md](./05_claude_md_guide.md) | CLAUDE.md 写作指南与模板 | 配置优化 |

---

## 核心原则（TL;DR）

### 1. Plan First，Always
> 最重要的单一习惯：在 Plan Mode 中先规划，人工审查计划，然后执行。
>
> `Shift+Tab` 两次 → Plan Mode

### 2. 验证是质量倍增器
> 给 Claude 一种验证自己工作的方式（测试/截图/脚本），质量提升 2-3 倍。

### 3. 上下文是有限资源
> 70% 开始下降，85% 出现幻觉，90%+ 严重退化。
> 任务间 `/clear`，用子代理探索。

### 4. 并行最大化产出
> Boris 的方式：5 个本地 worktree + 5-10 个云端会话。

### 5. CLAUDE.md 是复利投资
> 每次 Claude 犯错都更新它，团队知识持续积累。

---

## 快速参考：不同场景用什么

| 场景 | 推荐做法 |
|------|----------|
| 新功能开发 | Plan Mode → 人工批注计划 → Normal Mode |
| 调试 Bug | 对话式调试 + 逐步增加日志 |
| 大规模重构 | 小步迭代 + 每步测试 |
| 代码审查 | 新的独立会话（避免偏见）|
| 学习新代码库 | 像请教高级工程师一样提问 |
| 重复工作 | 做成斜杠命令 / Skill |
| 上下文快满 | `/compact` 或 `/clear` |
| 大批量任务 | `claude -p` 循环脚本 |

---

## 官方资源

- [Claude Code 文档](https://code.claude.com/docs)
- [Claude Code 最佳实践](https://code.claude.com/docs/en/best-practices)
- [Prompt 工程指南](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Anthropic 研究报告](https://www.anthropic.com/research)
- [上下文工程博客](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Boris Cherny @bcherny](https://x.com/bcherny)

---

## 社区资源

- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) - 精选工具、插件、命令列表
- [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) - 社区参考实现
- [claude-code-tips（45个）](https://github.com/ykdojo/claude-code-tips) - 实用技巧合集
- [claude-code-ultimate-guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide) - 从入门到高级

---

*最后更新：2026年4月 | 维护者：Jack Zhao*
