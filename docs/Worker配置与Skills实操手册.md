# Worker 配置与 Skills 实操手册

> **本文档面向**：想搞清楚 golutra 到底怎么配、Skill 到底是什么、Worker 之间怎么协作的用户  
> **核心原则**：只讲你能操作的东西，不贴内部 JSON 数据结构  
> **基于**：golutra 源码逐行分析

---

## 目录

1. [golutra 的三层架构](#1-golutra-的三层架构)
2. [Skill 到底是什么](#2-skill-到底是什么)
3. [Skill 完整操作流程](#3-skill-完整操作流程)
4. [Worker 从创建到干活的完整链路](#4-worker-从创建到干活的完整链路)
5. [规则体系：怎么约束 Worker 行为](#5-规则体系怎么约束-worker-行为)
6. [完整协作示例：技术栈探测 → 广播 → 编码 → 测试](#6-完整协作示例技术栈探测--广播--编码--测试)
7. [常见误区澄清](#7-常见误区澄清)

---

## 1. golutra 的三层架构

golutra 的配置分三层，理解这个是前提：

```
┌─────────────────────────────────────────────────────┐
│  第一层：全局（跨所有工作区共享）                       │
│  存储位置：~/.golutra/                               │
│                                                     │
│  包含：                                              │
│  ├── global-settings.json  ← 主题、语言、账户、快捷键  │
│  ├── global-data.json      ← 已安装的 Skill 列表     │
│  ├── skills/               ← Skill 文件夹库（实体）   │
│  │   ├── my-vue-rules/                               │
│  │   ├── react-patterns/                             │
│  │   └── ...                                         │
│  └── chat-db/              ← 聊天记录数据库           │
├─────────────────────────────────────────────────────┤
│  第二层：工作区（每个项目独立）                         │
│  存储位置：{你的项目}/.golutra/                        │
│                                                     │
│  包含：                                              │
│  ├── workspace.json        ← 成员列表、Roadmap、Skills│
│  └── skills/               ← 符号链接（指向全局库）    │
│      ├── my-vue-rules → ~/.golutra/skills/my-vue-rules│
│      └── ...                                         │
├─────────────────────────────────────────────────────┤
│  第三层：运行时（仅存在于内存，关闭就没了）              │
│                                                     │
│  包含：                                              │
│  ├── 终端会话（PTY 进程）                             │
│  ├── 成员在线状态                                     │
│  ├── 消息派发队列                                     │
│  └── 语义解析 Worker                                  │
└─────────────────────────────────────────────────────┘
```

**关键理解**：
- Worker（成员）属于**第二层**，每个工作区的成员列表独立
- Skill 文件夹属于**第一层**，所有工作区都能用
- Skill 通过**符号链接**挂载到工作区，一个 Skill 可以被多个工作区共用
- 你在 UI 上改了 Worker 配置，最终写入的是 `.golutra/workspace.json`

---

## 2. Skill 到底是什么

### 一句话：Skill = 一个文件夹

这个文件夹里可以放任何东西——规则文件、代码模板、Prompt 模板、配置文件。golutra 会把它**符号链接**到工作区的 `.golutra/skills/` 目录下。

当 AI Worker（比如 Claude Code）在工作区里运行时，它能**看到**这些文件。

### Skill 不是什么

| 误解 | 事实 |
|------|------|
| Skill 是 golutra 内置的功能模块 | ❌ 不是，Skill 只是你管理的文件夹 |
| Skill 会自动注入到 Worker 的上下文 | ❌ 不会自动注入，只是放在目录里供 AI 读取 |
| Skill 有固定的格式要求 | ❌ 没有，任何文件夹都行 |
| Skill 库有预置内容 | ❌ 目前为空，完全由用户自己导入 |

### Skill 的实际用途

因为 AI CLI 工具（Claude Code、Gemini CLI 等）启动时的工作目录是你的项目根目录，它们能看到 `.golutra/skills/` 下的所有内容。所以你可以利用这个机制：

```
你的项目/
├── src/
├── package.json
└── .golutra/
    ├── workspace.json
    └── skills/
        ├── coding-rules/          ← 符号链接，指向全局库
        │   ├── vue3-conventions.md
        │   ├── naming-rules.md
        │   └── forbidden-patterns.md
        └── prompt-templates/       ← 符号链接，指向全局库
            ├── component-dev.md
            ├── unit-test.md
            └── code-review.md
```

然后你在聊天中可以这样指示 Worker：

```
@编码助手-1 请先阅读 .golutra/skills/coding-rules/ 下的所有规范文件，
然后按照规范开发 UserList 组件。
```

---

## 3. Skill 完整操作流程

### 第一步：准备 Skill 文件夹

在你电脑任意位置创建一个文件夹，放入你的规则/模板。例如：

```bash
mkdir ~/my-skills/vue3-team-rules
```

在里面写入规则文件：

```bash
cat > ~/my-skills/vue3-team-rules/编码规范.md << 'EOF'
# Vue 3 编码规范

## 组件规范
- 必须使用 <script setup lang="ts">
- Props 使用 defineProps + withDefaults
- 事件使用 defineEmits 并定义类型

## 样式规范
- 使用 Tailwind CSS，不写 .css 文件
- 暗色模式必须适配（dark: 前缀）

## 命名规范
- 组件文件：PascalCase.vue
- composables：useCamelCase.ts
- stores：xxxStore.ts
EOF
```

### 第二步：导入到全局 Skill 库

1. 打开 golutra
2. 进入 **Skill 商店**（左侧导航或成员管理弹窗的 Skills Tab）
3. 切换到 **"已安装"** 标签页
4. 点击 **"导入文件夹"** 按钮
5. 选择你刚才创建的 `vue3-team-rules` 文件夹
6. 完成 → 文件夹被**复制**到 `~/.golutra/skills/vue3-team-rules/`

> ⚠️ 注意：是**复制**，不是引用。导入后修改原始文件夹不会同步。要修改请在 golutra 的 Skill 库里直接改（点击"打开文件夹"按钮）。

### 第三步：链接到工作区

1. 打开你的工作区
2. 打开 **Skill 管理弹窗**（成员管理入口 → Skills）
3. 在 **"当前"** 标签页，点击 **"导入"**
4. 弹出可用 Skill 列表（你全局库里有的）
5. 点击要链接的 Skill 旁边的 **"链接"** 按钮
6. 完成 → 在 `.golutra/skills/vue3-team-rules` 创建了符号链接

### 第四步：让 Worker 使用

在聊天中告诉 Worker 去读这些文件：

```
@编码助手-1 请阅读 .golutra/skills/vue3-team-rules/编码规范.md，
后续所有编码工作请严格遵守其中的规范。
```

### 操作汇总图

```
你的电脑上的文件夹
  │
  │ ① 导入（复制）
  ▼
~/.golutra/skills/vue3-team-rules/    ← 全局 Skill 库
  │
  │ ② 链接（符号链接）
  ▼
{项目}/.golutra/skills/vue3-team-rules/  ← 工作区中可见
  │
  │ ③ Worker 可读取
  ▼
AI Worker 在项目目录运行，能看到 .golutra/skills/ 下的文件
```

---

## 4. Worker 从创建到干活的完整链路

### Worker 的本质

Worker = 一个在你项目目录下运行的 **CLI 终端进程**。

| Worker 类型 | 实际运行的进程 | 干什么的 |
|------------|--------------|---------|
| Claude Code | `claude` 或 `claude --dangerously-skip-permissions` | AI 编程 |
| Gemini CLI | `gemini` | AI 编程 |
| Codex | `codex` | AI 编程 |
| OpenCode | `opencode` | AI 编程 |
| Qwen Code | `qwen` | AI 编程 |
| Terminal(Shell) | `bash` / `zsh` / `powershell` | 执行命令 |

### 创建 Worker 的完整链路

你在邀请弹窗中选了 Claude Code、数量 2、无限制开启，golutra 内部发生了什么：

```
你点击"发送邀请"
  │
  ▼
① 前端调用 Rust 后端的 invite_members()
  │  参数：terminal_type="claude", instance_count=2, unlimited_access=true
  │
  ▼
② 后端生成 2 个成员记录，写入 .golutra/workspace.json
  │  成员名自动生成：{工作区名}-assistant-claude-1, {工作区名}-assistant-claude-2
  │  终端命令自动填充：claude
  │
  ▼
③ 为每个成员创建终端会话 (terminal_create)
  │  查找 claude 可执行文件路径
  │  因为 unlimited_access=true → 追加 --dangerously-skip-permissions 参数
  │  最终命令：claude --dangerously-skip-permissions
  │
  ▼
④ 启动 PTY（伪终端）进程
  │  工作目录 = 你的项目根目录
  │  相当于在你项目里开了个终端，运行了 claude
  │
  ▼
⑤ 发送入职消息（onboarding）
  │  自动给 AI 发一条消息告诉它自己的名字：
  │  "xxx-assistant-claude-1，这是你的名字，现在正在和团队解决问题"
  │
  ▼
⑥ 启动语义解析 Worker（后台线程）
  │  持续监控 PTY 输出
  │  当输出稳定（>1.2秒没有新内容）→ 截取输出写入聊天
  │
  ▼
⑦ 成员上线，状态变为 Online
```

### 消息如何到达 Worker

你在群聊发了 `@编码助手-1 实现 UserList 组件`，内部链路：

```
你的消息
  │
  ▼
① 前端检测到 @mention → 提取 mentionIds: ["编码助手-1的id"]
  │
  ▼
② 消息写入聊天数据库
  │
  ▼
③ 触发消息派发 (orchestrate_chat_dispatch)
  │  从 workspace.json 读取成员列表
  │  根据 mentionIds 确定目标成员
  │  检查目标是否有终端配置 → 有 → 继续
  │  检查目标是否处于勿扰状态 → 否 → 继续
  │
  ▼
④ 消息进入批处理队列 (ChatDispatchBatcher)
  │  如果同一 Worker 有多条消息 → 合并
  │  如果 Worker 正在处理上一条 → 排队等待
  │
  ▼
⑤ 消息写入 PTY 标准输入
  │  等 100ms
  │  发送回车 "\r" 确认
  │
  ▼
⑥ Claude CLI 开始处理
  │
  ▼
⑦ 语义解析 Worker 监控输出
  │  输出稳定后 → 截取 → 过滤 → 写入聊天
  │
  ▼
⑧ 你在聊天中看到 Worker 的回复
```

### @All 广播如何工作

你发了 `@All 所有人注意...`：

```
前端检测到 @all → 设置 mentionAll: true
  │
  ▼
后端 resolve_targets() 函数：
  if mentions.mention_all {
      targets = 群内所有成员（除了你自己）
  }
  │
  ▼
每个有终端配置的成员都收到这条消息
```

---

## 5. 规则体系：怎么约束 Worker 行为

golutra **没有内置的规则引擎**。Worker 的行为通过以下三种方式约束：

### 方式一：CLI 工具自身的规则文件（推荐，最持久）

各 AI CLI 工具会自动读取项目中的特定文件作为系统规则：

| CLI 工具 | 规则文件 | 效果 |
|---------|---------|------|
| Claude Code | `CLAUDE.md`（项目根目录） | 所有 Claude Worker 自动加载 |
| Codex CLI | `AGENTS.md` 或 `codex.md` | 所有 Codex Worker 自动加载 |
| Gemini CLI | `GEMINI.md` | 所有 Gemini Worker 自动加载 |

**示例**：在项目根目录创建 `CLAUDE.md`：

```markdown
# 项目规范

## 技术栈
- Vue 3.4 + TypeScript 5.x + Vite 5
- Tailwind CSS 3.4（不写 .css 文件）
- Pinia 状态管理
- Vitest + Playwright 测试

## 编码规则
- 组件必须使用 <script setup lang="ts">
- 所有 Props 用 defineProps + withDefaults 定义类型
- 暗色模式必须适配
- API 调用统一放在 src/api/ 目录
- 每个组件必须有对应的单元测试

## 禁止事项
- 禁止使用 any 类型
- 禁止使用 Options API
- 禁止在组件中直接写 fetch
```

**优点**：
- ✅ 所有使用该 CLI 的 Worker 自动遵守
- ✅ 跟随项目（git 版本控制）
- ✅ 不需要每次手动发送
- ✅ Worker 重启后依然有效

### 方式二：@All 广播 Prompt（即时生效，但不持久）

在群聊中发送 `@All` 消息让所有 Worker 同时收到：

```
@All 以下是项目编码规范，所有人必须遵守：
1. 使用 <script setup lang="ts">
2. 样式用 Tailwind CSS
3. 暗色模式必须适配
```

**优点**：灵活、即时  
**缺点**：Worker 重启后失效，需要重新发

### 方式三：Skill 文件 + Prompt 引用（结构化，可复用）

把规则写成 Skill 文件夹，链接到工作区，然后在 Prompt 中引用：

```
@编码助手-1 请先阅读 .golutra/skills/coding-rules/ 下的所有文件，
然后按照规范开发 UserList 组件。
```

**优点**：结构化管理、跨工作区复用  
**缺点**：需要在 Prompt 中手动引用

### 三种方式对比

| | CLI 规则文件 | @All 广播 | Skill + Prompt |
|--|------------|----------|---------------|
| 自动生效 | ✅ | ❌ 需手动发 | ❌ 需手动引用 |
| 持久性 | ✅ 跟随项目 | ❌ 重启失效 | ✅ 文件持久 |
| 跨工作区 | ❌ 每个项目单独放 | ❌ 每个工作区单独发 | ✅ Skill 可复用 |
| 灵活修改 | 中等 | ✅ 随时改 | ✅ 改文件即可 |
| 适用 CLI | 仅特定 CLI | 所有 Worker | 所有 Worker |

**推荐组合**：`CLAUDE.md`（基础规范）+ `@All`（临时指令）+ Skill（可复用模板）

---

## 6. 完整协作示例：技术栈探测 → 广播 → 编码 → 测试

以下是一个完整的从零开始配置团队的流程。

### 场景：你有一个 Vue 3 + TypeScript 项目，想配一个 4 人 AI 团队

### 第一步：准备 CLAUDE.md（2 分钟）

在项目根目录创建 `CLAUDE.md`，写入基础规范（参考方式一的示例）。这样所有 Claude Worker 自动遵守。

### 第二步：准备 Skill 文件夹（可选，5 分钟）

如果你有 Prompt 模板或详细规范想复用：

```bash
# 创建 Skill 文件夹
mkdir -p ~/my-skills/vue3-prompts

# 写入组件开发 Prompt 模板
cat > ~/my-skills/vue3-prompts/组件开发模板.md << 'EOF'
# 组件开发 Prompt 模板

用 Vue 3 + TypeScript 实现 {组件名} 组件：

Props：
  {逐一列出，包含类型和默认值}

功能：
  {编号列出}

要求：
- <script setup lang="ts">
- Tailwind CSS 样式
- 支持暗色模式
- defineProps + withDefaults
- 提供完整的类型定义
EOF

# 写入单元测试 Prompt 模板
cat > ~/my-skills/vue3-prompts/单元测试模板.md << 'EOF'
# 单元测试 Prompt 模板

为 {组件路径} 编写 Vitest 单元测试：

覆盖场景：
1. 默认 Props 渲染
2. 自定义 Props 渲染
3. 事件触发
4. 边界条件

要求：
- 使用 @testing-library/vue
- 测试文件放在 __tests__/ 目录
- 覆盖率目标 > 80%
EOF
```

然后在 golutra 里导入并链接到工作区（参考第 3 节）。

### 第三步：邀请 Worker 团队（3 分钟）

打开你的工作区，分三次邀请：

**第一次**：技术栈探测员
- 邀请助手 → **Claude Code** → 数量 **1** → 无限制 **✅** → 沙盒 **❌** → 发送
- 改名为 `探测员`

**第二次**：编码助手
- 邀请助手 → **Claude Code** → 数量 **2** → 无限制 **✅** → 沙盒 **❌** → 发送
- 改名为 `编码助手-1` 和 `编码助手-2`

**第三次**：测试专家
- 邀请助手 → **Claude Code** → 数量 **1** → 无限制 **✅** → 沙盒 **❌** → 发送
- 改名为 `测试专家`

### 第四步：技术栈探测（1 分钟）

**私聊**探测员（点击头像进入 DM），发送：

```
请扫描当前项目的配置文件，输出技术栈摘要。扫描这些文件（存在则读取）：
package.json, tsconfig.json, vite.config.ts, tailwind.config.js,
.eslintrc*, vitest.config.ts, playwright.config.ts

输出格式（只输出摘要，不要解释）：

---
项目名：{name}
语言：{language}
前端框架：{framework} {version}
构建工具：{tool} {version}
CSS：{css framework}
测试：{test framework}
包管理器：{pm}
关键依赖：{list}
---
```

### 第五步：@All 广播技术栈 + 分工（1 分钟）

等探测员输出后，**复制摘要**，回到**群聊**，发送：

```
@All 本项目的技术栈和分工如下：

{粘贴技术栈摘要}

分工安排：
- @编码助手-1 → 负责页面级组件开发
- @编码助手-2 → 负责通用组件和工具函数
- @测试专家 → 负责为完成的组件编写单元测试

编码规范已在 CLAUDE.md 中定义，所有人自动遵守。
Prompt 模板在 .golutra/skills/vue3-prompts/ 目录下，需要时参考。
```

### 第六步：开始协作

**给编码助手-1 下发任务**：

```
@编码助手-1 请开发 UserList 组件。

需求：
- 路径：src/components/UserList.vue
- 展示用户列表（表格形式）
- 支持搜索过滤
- 支持分页
- 点击行跳转详情

参考 .golutra/skills/vue3-prompts/组件开发模板.md 的格式要求。
```

**编码助手-1 开发完成后，给测试专家下发任务**：

```
@测试专家 @编码助手-1 已完成 src/components/UserList.vue。
请为该组件编写 Vitest 单元测试。

参考 .golutra/skills/vue3-prompts/单元测试模板.md 的格式要求。
```

**让 Shell 执行器跑构建**（如果你加了一个 Terminal 类型的 Worker）：

```
@Shell执行器 请运行以下命令确认构建正常：
pnpm run build && pnpm run lint && pnpm run test
```

### 完整协作流程图

```
你
├── ① 创建 CLAUDE.md → 所有 Claude Worker 自动读取
├── ② 导入 Skill 文件夹 → 链接到工作区
├── ③ 邀请 4 个 Worker
│
├── ④ 私聊探测员 → 探测技术栈
│       └── 输出技术栈摘要
│
├── ⑤ 群聊 @All → 广播技术栈 + 分工
│       ├── 编码助手-1 收到 ✓
│       ├── 编码助手-2 收到 ✓
│       └── 测试专家 收到 ✓
│
├── ⑥ @编码助手-1 → 开发 UserList 组件
│       └── 输出完成的代码
│
├── ⑦ @测试专家 → 为 UserList 写测试
│       └── 输出测试代码
│
├── ⑧ @编码助手-2 → 开发 UserDetail 组件
│       └── 输出完成的代码
│
└── ⑨ 循环...
```

---

## 7. 常见误区澄清

### ❌ 误区 1："Worker 文档里的 JSON 配置需要我手动填写"

**事实**：那些 JSON 是 golutra 内部的数据结构，**自动生成的**。你在 UI 弹窗中选模型、设数量就够了。

你能操作的只有 4 个选项：**模型类型**、**数量**、**无限制访问**、**沙盒模式**。

### ❌ 误区 2："Skill 是 golutra 预置的功能插件"

**事实**：Skill 就是你自己管理的**文件夹**。golutra 只负责把文件夹从全局库**符号链接**到工作区目录。Skill 库目前是空的，完全由你自己导入。

### ❌ 误区 3："Worker 之间能自动协作、自动感知彼此"

**事实**：Worker 之间**不能直接通信**。每个 Worker 只是一个独立的 CLI 进程。协作完全靠你在聊天中**手动调度**——你看到 A 完成了，再把任务发给 B。

如果你想让 Worker 知道其他 Worker 的存在，需要在消息中**手动告知**，或者让 Worker 读取 `.golutra/workspace.json` 文件。

### ❌ 误区 4："规则可以全局配置，自动应用到所有工作区"

**事实**：
- **Worker 规则**：没有全局规则系统。每个工作区独立。
- **CLI 规则文件**（如 `CLAUDE.md`）：跟随项目，不跟随 golutra。
- **Skill 文件夹**：可以跨工作区复用（全局库），但需要**手动链接**到每个工作区。
- **唯一真正全局的**：主题、语言、自定义终端类型（在设置页面配）。

### ❌ 误区 5："探测员能自动广播技术栈给其他 Worker"

**事实**：你需要手动完成"复制 + @All 粘贴"这一步。探测员只是输出到它自己的聊天窗口。广播是你的手动操作。

---

## 附录：快速操作速查

| 我想... | 操作 |
|---------|------|
| 添加 Worker | 邀请助手弹窗 → 选模型 → 设数量 → 发送 |
| 给 Worker 改名 | 点击头像 → 管理成员 → 改名 → 保存 |
| 私聊 Worker | 点击头像 → 进入 DM |
| 广播给全体 | 群聊输入 `@All` + 内容 |
| 指定某 Worker | 群聊输入 `@成员名` + 内容 |
| 导入 Skill | Skill 商店 → 已安装 → 导入文件夹 |
| 链接 Skill 到工作区 | Skill 管理 → 当前 → 导入 → 选择 → 链接 |
| 设置全局规则 | 在项目根目录创建 `CLAUDE.md` / `GEMINI.md` |
| 发送临时规则 | 群聊 `@All` + 规范内容 |
| 查看 Worker 终端 | 点击成员头像 → 查看终端 |
| 设置勿扰 | 成员管理 → 手动状态 → 勿扰（Worker 暂停接收消息） |

---

*本文档基于 golutra 源码逐行分析编写。Skill 系统参考 `src-tauri/src/ui_gateway/project_skills.rs`，消息派发参考 `src-tauri/src/orchestration/dispatch.rs`，成员创建参考 `src-tauri/src/message_service/project_members.rs`。*
