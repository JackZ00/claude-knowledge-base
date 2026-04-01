# CLAUDE.md 完全指南：让 Claude 记住你的偏好

> CLAUDE.md 是 Claude Code 中最重要的配置文件。每次会话开始时 Claude 都会自动读取它。本文从写作原则到完整模板，帮你打造高效的 CLAUDE.md。

---

## 一、CLAUDE.md 是什么

CLAUDE.md 是一个特殊的 Markdown 文件，Claude 在每次对话开始时自动读取，为其提供持久化的项目上下文——那些它无法从代码中推断出来的信息。

**本质上，它是你和 Claude 之间的"工作协议"。**

---

## 二、文件位置与作用范围

| 位置 | 作用范围 |
|------|----------|
| `~/.claude/CLAUDE.md` | 所有 Claude 会话（全局规则）|
| `./CLAUDE.md`（项目根目录）| 该项目的所有会话，**检入 Git 与团队共享** |
| `./subdir/CLAUDE.md` | 只在处理该子目录文件时加载 |
| 父目录中的 CLAUDE.md | 自动向上查找并加载 |

**推荐结构**（来自 Boris Cherny 团队实践）：
```
~/.claude/CLAUDE.md          ← 个人全局偏好
./CLAUDE.md                  ← 项目规则（检入 Git）
./frontend/CLAUDE.md         ← 前端特定规则
./backend/CLAUDE.md          ← 后端特定规则
```

---

## 三、黄金原则

### 原则 1：每行都要经得起考验

**黄金问题**：删掉这行，Claude 会犯错吗？

- 如果不会 → **删掉它**
- 如果 Claude 已经能正确做到 → **删掉它**
- 如果只有某些时候才需要 → 考虑改为 Skill

### 原则 2：短小精悍

- 目标：**200 行以内**（每个文件）
- CLAUDE.md 太长会导致 Claude 忽略其中的关键规则
- 定期审查：每次 Claude 犯了 CLAUDE.md 中已有规则描述的错误，就是时候精简了

### 原则 3：它是活文档

**Boris 团队的实践**：
> "每当 Claude 做错了什么，我们就把正确做法加进 CLAUDE.md，这样下次它就知道不要再犯。"

把它当代码一样维护：
- 出错时 review 它
- 定期 prune（删除过时规则）
- 通过观察 Claude 行为是否改变来测试修改效果

### 原则 4：用强调提高遵守率

```markdown
IMPORTANT: 永远不要直接修改 migrations/ 目录下的文件
YOU MUST: 每次修改数据库 schema 后运行 npm run db:check
```

---

## 四、内容模板（按项目类型）

### 4.1 通用 Web 项目模板

```markdown
# 代码风格
- 使用 ES modules（import/export），不用 CommonJS（require）
- 尽量解构导入（例如：import { foo } from 'bar'）
- TypeScript 严格模式，不允许 any 类型

# 工作流
- 修改完一系列代码后，运行类型检查
- 优先运行单个测试而不是整个测试套件（性能考虑）
- 提交前运行 npm run lint

# 测试
- 测试文件放在 __tests__/ 目录或与源文件同目录（.test.ts 后缀）
- 不要使用 mock，除非绝对必要
- 每个新函数都需要测试边缘情况

# 架构决策
- 状态管理：使用 Zustand，不要用 Redux
- API 调用：统一通过 src/api/ 目录
- 不要引入新的直接依赖，除非讨论过

# 重要提醒
IMPORTANT: 不要修改 src/legacy/ 目录下的任何文件
IMPORTANT: 数据库迁移文件只能通过 npm run migration:create 生成
```

### 4.2 Python 后端项目模板

```markdown
# 环境
- Python 3.11+，使用 poetry 管理依赖
- 格式化工具：black + isort
- 类型检查：mypy（严格模式）

# 代码风格
- 优先使用 dataclass 而非 dict 传递结构化数据
- 异步函数用 async/await，不用回调
- 错误处理：抛出具体异常类型，不要用裸 Exception

# 测试
- 框架：pytest
- Fixtures 放在 conftest.py
- 运行单个测试：pytest tests/test_auth.py::test_login -v

# 工作流
- 修改后运行：mypy src/ && pytest tests/ -x
- API 文档通过 docstring 自动生成，保持 docstring 更新

# 项目特定知识
- 数据库连接通过 src/db/connection.py 统一管理
- 认证逻辑在 src/auth/ 目录
- 外部 API 集成在 src/integrations/ 目录
```

### 4.3 全栈 Next.js 项目模板

```markdown
# 技术栈
- Next.js 15 App Router（不是 Pages Router）
- Prisma + PostgreSQL
- Tailwind CSS（不要写 CSS-in-JS）
- Shadcn/ui 组件库

# 代码规范
- Server Components 默认；需要客户端交互才加 'use client'
- 数据获取：用 Server Actions 或 Route Handlers，不要客户端 fetch
- 图片：始终用 next/image，设置 width 和 height

# 数据库
- Schema 修改：先改 schema.prisma，然后 npx prisma migrate dev
- YOU MUST 不要直接写 SQL，通过 Prisma Client 操作
- 查询优化：避免 N+1，使用 include 预加载关联数据

# 工作流
- 类型检查：npx tsc --noEmit
- 构建验证：npm run build
- 本地开发：npm run dev（端口 3000）

# 环境变量
- 敏感变量在 .env.local（不检入 Git）
- 新变量要同时更新 .env.example
```

### 4.4 移动端（React Native）模板

```markdown
# 技术栈
- React Native 0.74+ with Expo
- TypeScript 严格模式
- 状态管理：Zustand
- 导航：Expo Router

# 平台规则
- 平台特定代码用 .ios.ts / .android.ts 后缀
- 不要用 Platform.OS 条件判断超过 3 处，改为平台文件
- 样式：StyleSheet.create，不要内联样式

# 测试
- Jest + React Native Testing Library
- E2E：Detox（只在 CI 中运行）

# 常见陷阱
IMPORTANT: 不要在 Android 上使用 Shadow 属性，用 elevation 替代
IMPORTANT: 图片必须通过 require() 静态导入，动态 URI 用 { uri: ... }
```

---

## 五、CLAUDE.md 导入语法

CLAUDE.md 支持用 `@` 导入其他文件：

```markdown
# CLAUDE.md
参见 @README.md 了解项目概述
参见 @package.json 了解可用的 npm 命令

# 详细指南
- Git 工作流：@docs/git-instructions.md
- API 设计规范：@docs/api-conventions.md
- 个人覆盖：@~/.claude/personal-overrides.md
```

**使用场景**：
- 避免重复信息（链接到已有文档）
- 个人偏好与团队规范分离
- 长内容放独立文件，CLAUDE.md 保持简洁

---

## 六、CLAUDE.md vs 其他配置方式

| 配置方式 | 什么时候用 |
|----------|-----------|
| **CLAUDE.md** | 每个会话都需要的持久规则 |
| **Skills**（`.claude/skills/`）| 特定工作流，按需调用 |
| **Hooks**（`settings.json`）| 必须每次执行的操作（如格式化）|
| **Subagents**（`.claude/agents/`）| 专门能力（安全审查、代码简化）|
| **Slash Commands**（`.claude/commands/`）| 重复的工作流模板 |

**原则**：能用 hook 确保执行的，不要只写在 CLAUDE.md 里（CLAUDE.md 是建议性的，hook 是强制性的）。

---

## 七、诊断 CLAUDE.md 是否有效

| 症状 | 诊断 | 解决方案 |
|------|------|----------|
| Claude 还在重复 CLAUDE.md 里禁止的错误 | 文件太长，规则被淹没 | 大幅精简，只留真正关键的规则 |
| Claude 经常问 CLAUDE.md 里已经回答的问题 | 表述模糊 | 重写为更清晰、更具体的指令 |
| CLAUDE.md 超过 300 行 | 严重过度膨胀 | 把大多数内容移到 Skills 或独立文档 |
| 加了规则但 Claude 行为没变 | 规则表述不够强调 | 加 IMPORTANT / YOU MUST 前缀 |

---

## 八、团队协作最佳实践

**Boris Cherny 团队的做法**：
1. CLAUDE.md 检入 Git，让所有人可以贡献
2. 每次 PR 后，如果发现 Claude 犯了新错误，更新 CLAUDE.md
3. 团队把 CLAUDE.md 的更新视为 engineering 的一部分
4. 每月审查一次，删除过时规则

**代码审查 CLAUDE.md 的标准**：
- 新增的规则是否有真实的 Bug 支撑？
- 是否有比 CLAUDE.md 更好的方式（hook/skill）来执行这个规则？
- 删掉这行 Claude 会犯错吗？

---

*来源：[Claude Code Best Practices 官方文档](https://code.claude.com/docs/en/best-practices)、[Boris Cherny X 分享](https://x.com/bcherny)、[shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)*
