# 实战场景策略：不同情况下如何使用 Claude

> 本文整合 Boris Cherny、官方文档、社区经验，按具体开发场景分类，给出对应策略。

---

## 场景一：开发新功能

### 推荐策略：Interview → Plan → Implement → Verify

**Step 1：让 Claude 采访你（明确需求）**

```
我想构建 [简短描述]。请详细采访我。
问关于技术实现、UI/UX、边缘情况、权衡取舍的问题。
持续采访直到覆盖所有方面，然后把完整规格写到 SPEC.md。
```

**Step 2：Plan Mode 制定计划**（Shift+Tab 两次进入）

```
[Plan Mode] 读取相关文件，理解现有架构。
我想添加 Google OAuth。哪些文件需要改动？创建一个实施计划。
```

**Step 3：人工审查计划**（Ctrl+G 在编辑器中打开）
- 检查 Claude 的假设是否正确
- 添加约束（如：不能引入新依赖）

**Step 4：切换 Normal Mode 实施 → 验证**

### 常见陷阱
- ❌ 不要让 Claude 直接写代码（无规划会解决错误问题）
- ❌ 不要一次性给太大任务（Context 超 60% 质量下降）
- ✅ 功能复杂时，拆分为多个子任务，各用独立会话

---

## 场景二：调试 Bug

### 推荐策略：症状 → 根因 → 验证

| ❌ 模糊 | ✅ 具体 |
|--------|--------|
| "修复登录 bug" | "用户报告 session 超时后登录失败。检查 src/auth/，写一个可复现问题的失败测试，然后修复" |
| "构建失败了" | "构建报这个错误：[粘贴]。修复并验证构建成功，解决根本原因" |

**调试循环策略**（社区验证有效）：
```
1. 让 Claude 在关键路径添加详细日志
2. 运行代码，粘贴终端输出给 Claude
3. 还没定位到？让 Claude 继续增加日志
4. 找到核心问题后修复，并写回归测试
```

```bash
cat error.log | claude   # 直接 pipe 错误日志
```

**隔离调试**（保持主上下文干净）：
```
用子代理调查 src/auth/ 中 session 超时的问题，
读取相关文件后报告发现，不要修改任何文件。
```

---

## 场景三：代码重构

### 推荐策略：理解 → 小步迭代 → 测试验证

> "不要一次性重构大文件——分解成更小的增量改动，每步之后运行测试。"

**大规模迁移自动化**（来自社区）：
```bash
for file in $(cat files_to_migrate.txt); do
  claude -p "将 $file 从 React 迁移到 Vue。返回 OK 或 FAIL。" \
    --allowedTools "Edit,Bash(git commit *)"
done
```

**重构后验证清单**：
- [ ] 所有测试通过
- [ ] TypeScript 类型检查（`tsc --noEmit`）
- [ ] Linter 无新警告
- [ ] 性能基准未退化

---

## 场景四：代码审查

### 推荐策略：Writer / Reviewer 双会话模式

```
会话 A（Writer）：实现 API 速率限制功能

会话 B（Reviewer，新的干净上下文）：
审查 @src/middleware/rateLimiter.ts。
寻找边缘情况、竞态条件，以及与现有中间件模式的一致性。
```

**新会话的价值**：不会偏向刚写的代码，审查更客观。

---

## 场景五：学习新代码库

### 推荐策略：像请教高级工程师一样提问

```
- "日志系统是怎么工作的？"
- "foo.rs 第 134 行的 async move { ... } 是什么意思？"
- "为什么第 333 行调用 foo() 而不是 bar()？"
```

**Claude 生成可视化解释**（来自 Boris 团队）：
```
帮我生成一个 HTML 幻灯片，解释 AuthManager 模块的架构。
画一个 ASCII 图，展示认证系统的请求流程。
```

---

## 场景六：写测试

### 推荐策略：测试与实现分离

```
会话 A：写 validateEmail 的测试用例（先只写测试，不要实现函数）
  - user@example.com → true
  - invalid → false

会话 B：看 @tests/validateEmail.test.ts，实现函数让所有测试通过。
```

---

## 场景七：上下文管理

**健康阈值**：
- 0-60%：正常运作
- 70-85%：精度下降，考虑 `/compact`
- 85%+：幻觉增加，建议 `/clear` 重开

```bash
/clear                          # 任务间清空
/compact 专注于 API 的变更      # 有针对性压缩
```

---

## 场景八：CI/CD 自动化

```bash
claude -p "分析这个 PR 是否有安全问题"
claude -p "列出所有 API endpoints" --output-format json
claude --permission-mode auto -p "修复所有 lint 错误"
```

---

## 场景九：与设计稿协作

```bash
claude mcp add figma

查看 Figma 中 Dashboard 组件的设计，
参照 @src/components/widgets/ 中现有 widget 的实现模式实现。
完成后截图对比 Figma 设计和实现结果。
```

---

## 场景十：跨会话保持上下文

```bash
/rename oauth-migration   # 给会话命名
claude --resume           # 选择恢复历史会话
claude --continue         # 恢复最近会话
```

**最佳实践**：在 SPEC.md 写规格，TODO.md 追踪进度，新会话加载它们继续。

---

## 快速决策矩阵

| 情况 | 策略 |
|------|------|
| 新功能，多文件 | Plan Mode → 人工审查 → Normal Mode |
| Bug 难定位 | 逐步加日志 + 粘贴输出 |
| 大规模重构 | 小步迭代 + 每步测试 |
| 代码审查 | 新会话 + 子代理 |
| 上下文快满 | `/compact` 或 `/clear` |
| 重复工作流 | 做成斜杠命令/Skill |
| 同类多文件任务 | `claude -p` 循环脚本 |

---

*来源：[Boris Cherny X 帖子](https://x.com/bcherny)、[Claude Code 官方最佳实践](https://code.claude.com/docs/en/best-practices)*
