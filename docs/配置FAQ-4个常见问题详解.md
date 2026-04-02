# golutra 配置 FAQ：4 个常见问题详解

> 本文档回答 4 个用户高频问题，每个问题基于源码分析给出准确答案，并附操作示例。

---

## 📖 目录

- [问题一：能创建探测技术栈并全局广播的成员吗？](#问题一能创建探测技术栈并全局广播的成员吗)
- [问题二：每个 Worker 能知道当前有哪些 Worker 吗？](#问题二每个-worker-能知道当前有哪些-worker-吗)
- [问题三：Worker、规则能全局配置，不限于单工作区吗？](#问题三worker规则能全局配置不限于单工作区吗)
- [问题四：Worker 配置详细指导（手把手实操）](#问题四worker-配置详细指导手把手实操)

---

## 问题一：能创建探测技术栈并全局广播的成员吗？

### 回答：✅ 可以，分两步完成

**golutra 本身不会自动探测技术栈**，但你可以：
1. 邀请一个 AI Worker（用 Claude Code），让它读取项目文件并输出技术栈摘要
2. 用 **@All** 功能把摘要广播给所有 Worker

### @All 广播的工作原理（基于源码）

当你在**群聊频道**发送包含 `@all` 的消息时：

```
消息路径：
ChatInput.vue → 检测到 @all → 设置 mentionAll: true
→ chatStore.ts → 调用 Tauri 命令 chat_send_message_and_dispatch
→ dispatch.rs → resolve_targets() 检查 mentions.mention_all
→ 如果 mention_all == true → 群内所有成员（除你之外）都作为目标
→ 每个有终端配置的成员 → 终端收到消息
```

**源码关键逻辑**（`src-tauri/src/orchestration/dispatch.rs`）：

```rust
fn resolve_targets(...) -> Vec<String> {
    // ...
    if mentions.mention_all {
        // @all: 所有群成员都是目标
        targets.extend(member_ids.iter().cloned());
    } else {
        // 只有被 @ 的成员是目标
        targets.extend(mentions.mention_ids.iter().cloned());
    }
    // 过滤掉发送者自己
    targets.into_iter()
        .filter(|id| id.as_str() != sender_id)
        .collect()
}
```

### 完整操作步骤

#### 步骤 1：邀请"技术栈探测员"

1. 点击**邀请助手** → 选 **Claude Code** → 数量 **1** → 无限制 **✅** → 沙盒 **❌** → 发送
2. 点击新成员头像 → 管理成员 → 改名为 **`技术栈探测员`**

#### 步骤 2：私聊让它探测

点击技术栈探测员的头像，进入私聊，发送：

```
请扫描当前项目根目录，读取以下文件并输出技术栈摘要：
- package.json（依赖列表和版本）
- tsconfig.json（TypeScript 配置）
- vite.config.ts / webpack.config.js（构建工具）
- tailwind.config.js（CSS 框架）
- Cargo.toml / go.mod / requirements.txt（其他语言）
- docker-compose.yml / Dockerfile（容器化）
- .github/workflows/（CI/CD）

输出格式：
项目名：xxx
主要语言：xxx
前端框架：xxx
构建工具：xxx
CSS 方案：xxx
测试框架：xxx
包管理器：xxx
核心依赖及版本：（列表）
编码规范：xxx
```

#### 步骤 3：复制结果，回群聊 @All 广播

```
@All 以下是本项目的技术栈信息，所有人后续工作请遵守：

项目名：my-app
主要语言：TypeScript
前端框架：Vue 3.4
构建工具：Vite 5.0
CSS 方案：Tailwind CSS 3.4
测试框架：Vitest 2.0 + Playwright
包管理器：pnpm 9.x

编码规范：
- <script setup lang="ts">
- Pinia 状态管理
- Tailwind CSS（不写 .css 文件）
- 暗色模式必须适配
```

> **详细的探测员 Worker 文档**请参考：[docs/workers/技术栈探测员.md](./workers/技术栈探测员.md)

---

## 问题二：每个 Worker 能知道当前有哪些 Worker 吗？

### 回答：⚠️ 间接可以，但不是自动的

### 当前架构（基于源码分析）

**成员数据存储位置**：
- 所有成员信息保存在 `.golutra/workspace.json`（或 `~/.golutra/{workspaceId}/project.json`）
- 数据结构（`src/features/workspace/projectStore.ts`）：

```typescript
type ProjectData = {
  projectId: string;
  version: number;
  members: Member[];  // ← 所有成员的完整列表在这里
  // ...
};
```

**每个成员的数据结构**（`src/features/chat/types.ts`）：

```typescript
type Member = {
  id: string;           // 唯一 ID
  name: string;         // 显示名称（如"编码助手-1"）
  roleType: MemberRole; // 'owner' | 'admin' | 'assistant' | 'member'
  terminalType?: TerminalType; // 'claude' | 'gemini' | 'codex' | 'shell' 等
  terminalCommand?: string;
  status: MemberStatus; // 'online' | 'working' | 'dnd' | 'offline'
  unlimitedAccess?: boolean;
  sandboxed?: boolean;
  // ...
};
```

### Worker 如何"看到"其他 Worker？

**在前端 UI 层**：
- 所有成员都显示在**成员列表面板**中
- 聊天输入框输入 `@` 时会弹出**全员列表**用于提及

**在 AI 终端层**：
- AI Worker **本身无法直接查询成员列表**
- AI 只能看到**发送到它的消息**
- 但你可以通过以下方式让 AI 知道队友：

#### 方法 1：在 @All 广播中列出团队

```
@All 当前团队成员如下，请记住：

| 成员名 | 角色 | CLI 类型 | 职责 |
|--------|------|---------|------|
| 编码助手-1 | assistant | Claude Code | 前端页面开发 |
| 编码助手-2 | assistant | Claude Code | 前端组件开发 |
| 测试专家 | assistant | Claude Code | 单元测试编写 |
| Shell执行器 | assistant | Terminal | 命令执行 |

当你需要其他成员帮忙时，请在回复中说明"建议交给 {成员名} 处理"。
```

#### 方法 2：让 AI 读取 workspace.json

```
@技术栈探测员 请读取 .golutra/workspace.json 文件，
列出 members 数组中所有成员的 name、terminalType 和 roleType。
```

> **注意**：这依赖 AI 有文件读取权限（关闭沙盒模式），且 `.golutra/workspace.json` 存在于项目目录中。

#### 方法 3：需求分析师协调

如果你用了方案七（需求驱动），需求分析师可以担任信息中枢：

```
@需求分析师 请读取 .golutra/workspace.json 获取团队成员列表，
然后基于每个成员的 terminalType 和名称，规划任务分配。
```

### 总结

| 问题 | 答案 |
|------|------|
| Worker 能自动知道其他 Worker？ | ❌ 不能 |
| Worker 能通过文件读取知道？ | ✅ 可以读取 workspace.json |
| Worker 能通过你告知？ | ✅ 你 @All 广播团队信息 |
| UI 上能看到全部成员？ | ✅ 成员列表面板始终显示 |

---

## 问题三：Worker、规则能全局配置，不限于单工作区吗？

### 回答：部分可以

### 当前的全局 vs 工作区配置（基于源码）

golutra 有**两层配置**：

| 层级 | 存储位置 | 作用域 | 包含什么 |
|------|---------|--------|---------|
| **全局设置** | `~/.golutra/global-settings.json` | 所有工作区共享 | 外观、语言、账户、通知、快捷键、聊天、**自定义终端** |
| **工作区配置** | `.golutra/workspace.json` | 仅当前工作区 | 成员列表、Roadmap、Skills、终端状态 |

### 全局设置详细内容（`src/features/global/settingsStore.ts`）

```typescript
type SettingsState = {
  appearance: {
    theme: 'dark' | 'light' | 'system';
  };
  locale: 'en-US' | 'zh-CN';
  account: {
    displayName: string;
    email: string;
    avatar: string;
    timezone: string;
    status: MemberStatus;
  };
  notifications: {
    desktop: boolean;
    sound: boolean;
    mentions: boolean;
    quietHoursEnabled: boolean;
    quietHoursStart: string;
    quietHoursEnd: string;
  };
  keybinds: { /* 快捷键 */ };
  chat: {
    streamOutput: boolean; // 是否流式输出 AI 回复
  };
  members: {
    defaultMemberIndex: number;  // ⭐ 默认选中哪个终端类型
    customMembers: CustomMember[]; // ⭐ 自定义终端列表
  };
};
```

### 哪些能全局配 / 哪些不能

| 配置项 | 能全局吗？ | 说明 |
|--------|-----------|------|
| **外观/主题** | ✅ 全局 | 所有工作区共享 |
| **语言** | ✅ 全局 | 所有工作区共享 |
| **自定义终端类型** | ✅ 全局 | 在设置中添加自定义终端，所有工作区的邀请弹窗都能看到 |
| **默认终端选择** | ✅ 全局 | 打开邀请弹窗时默认选中哪个终端 |
| **成员列表** | ❌ 仅工作区 | 每个工作区的团队不同 |
| **Roadmap 任务** | ❌ 仅工作区 | 每个工作区的路线图不同 |
| **Skills** | ❌ 仅工作区 | 每个工作区的技能配置不同 |
| **Rules/规则** | ⚠️ 无显式规则系统 | 详见下方说明 |

### 关于"规则"

**当前版本没有独立的"规则"配置系统**。代码中找不到 `customRules`、`rules` 或 `.rules` 文件的引用。

Worker 的行为规则通过以下方式实现：

1. **Prompt 注入**：你在聊天中发送的指令就是"规则"
2. **@All 广播**：对所有 Worker 生效的指令就是"全局规则"
3. **CLI 工具本身的配置**：如 Claude 的 `CLAUDE.md`、Codex 的 `codex.md` 等——这些是 CLI 工具自身的规则机制，不是 golutra 管理的

### 如何实现"全局规则"效果

虽然 golutra 没有内置全局规则系统，但你可以通过以下方式达到类似效果：

#### 方法 1：使用 CLI 工具自身的规则文件

各 CLI 工具支持在项目根目录放置规则文件：

| CLI 工具 | 规则文件 | 位置 |
|---------|---------|------|
| Claude Code | `CLAUDE.md` | 项目根目录 |
| Codex | `codex.md` 或 `AGENTS.md` | 项目根目录 |
| Gemini CLI | `GEMINI.md` | 项目根目录 |

**这些文件对所有使用该 CLI 的 Worker 自动生效**，不需要每次手动广播。

示例 `CLAUDE.md`：
```markdown
# 项目规范

## 技术栈
- Vue 3 + TypeScript + Tailwind CSS
- 构建工具：Vite 5
- 测试：Vitest + Playwright

## 编码规则
- 使用 <script setup lang="ts">
- 状态管理用 Pinia
- 样式用 Tailwind（不写 CSS 文件）
- 所有组件支持暗色模式
- API 调用统一放在 src/api/ 目录
```

#### 方法 2：每次开工时 @All 广播

在每个工作区开工时，第一条消息 @All 广播规则：

```
@All 工作规范：
1. TypeScript 严格模式
2. Vue 组件用 <script setup lang="ts">
3. 样式用 Tailwind CSS
4. 提交前确保 pnpm run lint 通过
```

#### 方法 3：自定义终端（全局生效）

在**设置 → 成员**中添加自定义终端，所有工作区都可以使用：

这属于全局配置（`settings.members.customMembers`），你在设置中添加的自定义终端会出现在**所有工作区**的邀请弹窗中。

---

## 问题四：Worker 配置详细指导（手把手实操）

### 核心认知：你只需要配 4 个东西

Worker 文档里写了很多"配置"（JSON、Skill、Prompt 模板等），这可能导致困惑。让我明确：

> **你在 golutra 页面上只能配 4 个东西。文档里的 JSON 配置是内部数据结构，不是你需要手动填写的。**

| # | 你需要配的 | 在哪配 | 怎么配 |
|---|-----------|-------|--------|
| ① | 模型/终端类型 | 邀请弹窗 | 点击选择 |
| ② | 实例数量 | 邀请弹窗 | +/- 按钮 |
| ③ | 无限制访问 | 邀请弹窗 | 开关按钮 |
| ④ | 沙盒模式 | 邀请弹窗 | 开关按钮 |

**其他所有的"配置"（角色名、职责、技术栈约束、Prompt 模板）都是通过你发消息来实现的。**

### 实操演示：从零配置一个"编码助手"

#### 第 1 步：打开邀请弹窗

在工作区聊天界面，找到成员区域的 **"+"** 或 **"邀请助手"** 按钮，点击。

#### 第 2 步：弹窗中的 4 个配置

弹窗会展示以下内容：

```
┌──────────────────────────────────┐
│          邀请助手                │
│   为工作区选择终端助手            │
│                                  │
│  ┌────────────────────────────┐  │
│  │ ● Gemini CLI      token   │  │  ← 6 个内置选项
│  │   Codex           code    │  │
│  │   Claude Code  psychology │  │  ← 写代码选这个
│  │   OpenCode        code    │  │
│  │   Qwen Code  model_train  │  │
│  │   Terminal      terminal  │  │  ← 纯命令选这个
│  └────────────────────────────┘  │
│                                  │
│  实例数量         [- 1 +]        │  ← 编码助手建议 2
│                                  │
│  无限制访问       [====○ ]       │  ← 建议开启
│  主要角色开启，频繁使用            │
│                                  │
│  沙盒模式         [○==== ]       │  ← 建议关闭
│                                  │
│  ┌────────────────────────────┐  │
│  │         发送邀请            │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

**编码助手的配置**：
1. 点击 **Claude Code**（橙色那行）
2. 实例数量调到 **2**（点两下 + 号）
3. 无限制访问：**开启**（滑块推到右边）
4. 沙盒模式：**保持关闭**
5. 点击 **发送邀请**

#### 第 3 步：等待 Worker 启动

邀请后 golutra 自动：
1. 创建 2 个成员（如 `myapp-assistant-claude-1` 和 `myapp-assistant-claude-2`）
2. 启动 2 个 Claude Code 终端
3. 自动发送 onboarding 消息（告诉 AI 它的名字）
4. 状态从 Offline → Pending → Connecting → Connected (Online)

#### 第 4 步：给 Worker 改名（强烈建议）

1. 在成员列表找到 `myapp-assistant-claude-1`
2. **点击头像** → 出现管理界面
3. **管理成员** → 修改**显示名称**为 `编码助手-1`
4. 点击**保存**
5. 对 `myapp-assistant-claude-2` 重复操作，改名为 `编码助手-2`

#### 第 5 步：通过消息定义角色

改名后，Worker 本身不知道它是"编码助手"。你需要发消息告诉它：

**群聊 @All 或私聊**：
```
@编码助手-1 你是一个前端编码助手，负责：
1. Vue 3 + TypeScript 组件开发
2. 页面布局和交互实现
3. Bug 修复和代码重构

技术栈约束：
- 使用 <script setup lang="ts">
- Pinia 状态管理
- Tailwind CSS 样式
- 支持暗色模式

当你完成任务后，请简要说明改动了哪些文件。
```

---

### 实操演示：配置完整的 4 人团队

#### 目标团队

| 成员 | 选什么模型 | 数量 | 无限制 | 沙盒 |
|------|-----------|------|--------|------|
| 编码助手 | Claude Code | 2 | ✅ | ❌ |
| 测试专家 | Claude Code | 1 | ✅ | ❌ |
| Shell 执行器 | Terminal | 1 | ✅ | ❌ |

#### 操作流程

**第一次邀请**（编码助手 ×2）：
1. 邀请助手 → Claude Code → 数量 2 → 无限制 ✅ → 沙盒 ❌ → 发送
2. 创建了 `xxx-assistant-claude-1` 和 `xxx-assistant-claude-2`
3. 改名为 `编码助手-1` 和 `编码助手-2`

**第二次邀请**（测试专家 ×1）：
1. 邀请助手 → Claude Code → 数量 1 → 无限制 ✅ → 沙盒 ❌ → 发送
2. 创建了 `xxx-assistant-claude-3`
3. 改名为 `测试专家`

**第三次邀请**（Shell 执行器 ×1）：
1. 邀请助手 → Terminal → 数量 1 → 无限制 ✅ → 沙盒 ❌ → 发送
2. 创建了 `xxx-assistant-terminal-1`
3. 改名为 `Shell执行器`

**广播团队信息和技术栈**：

```
@All 团队配置完成，各角色职责如下：

1. @编码助手-1 → 负责页面级组件开发
2. @编码助手-2 → 负责通用组件开发
3. @测试专家 → 负责为完成的组件编写 Vitest 单元测试
4. @Shell执行器 → 负责执行构建和测试命令

技术栈：
- Vue 3.4 + TypeScript 5.x
- Vite 5 构建
- Tailwind CSS 3.4
- Pinia 状态管理
- Vitest 测试框架

编码规范：
- <script setup lang="ts">
- 暗色模式必须适配
- 所有 Props 使用 defineProps + withDefaults
```

---

### Worker 文档中的 JSON 配置是什么？

你可能在 `docs/workers/编码助手.md` 中看到类似这样的内容：

```json
{
  "id": "{ULID}",
  "name": "{workspace}-assistant-claude-{N}",
  "roleKey": "members.roles.aiAssistant",
  "roleType": "assistant",
  "avatar": "css:canyon",
  "status": "online",
  "terminalType": "claude",
  "terminalCommand": "claude",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": false
}
```

**这不是你要手动填写的配置！** 这是 golutra **内部自动生成**的数据结构。

| JSON 字段 | 含义 | 你需要做什么 |
|-----------|------|-------------|
| `id` | 自动生成的唯一 ID | 自动生成，不用管 |
| `name` | 成员名称 | 创建后在"管理成员"中改名 |
| `roleType` | 角色类型 | 自动设为 `assistant` |
| `avatar` | 头像样式 | 自动分配 |
| `status` | 在线状态 | 自动管理 |
| `terminalType` | 终端类型 | = 你在弹窗中选的模型 |
| `terminalCommand` | 启动命令 | 自动根据模型填充 |
| `autoStartTerminal` | 自动启动 | 默认 true |
| `unlimitedAccess` | 无限制模式 | = 你在弹窗中的开关 |
| `sandboxed` | 沙盒模式 | = 你在弹窗中的开关 |

**换句话说**：弹窗中的 4 个选项就是在填充这些 JSON 字段。你不需要手动编辑 JSON。

---

### 文档中的 Prompt 模板怎么用？

Worker 文档（如 `docs/workers/编码助手.md`）中有很多 Prompt 模板，例如：

```
用 Vue 3 + TypeScript 实现 {组件名} 组件：

Props：
  {逐一列出}

功能：
  1. {功能点}

要求：
- <script setup lang="ts">
- 使用 Tailwind CSS
- 支持暗色模式
```

**使用方式**：
1. 复制模板
2. 把 `{xxx}` 占位符替换成你的实际内容
3. 在聊天中发给对应的 Worker

**示例**：

```
@编码助手-1 用 Vue 3 + TypeScript 实现 ProductCard 组件：

Props：
  - product: { id: string, name: string, price: number, image: string }
  - showActions: boolean (默认 true)

功能：
  1. 展示商品图片、名称、价格
  2. 悬浮时显示操作按钮（加入购物车、收藏）
  3. 点击卡片跳转商品详情

要求：
- <script setup lang="ts">
- 使用 Tailwind CSS
- 支持暗色模式
- 图片懒加载
```

---

### 总结速查表

| 你想做什么 | 在哪操作 | 怎么操作 |
|-----------|---------|---------|
| 添加新 Worker | 邀请助手弹窗 | 选模型 → 设数量 → 开关 → 发送 |
| 给 Worker 改名 | 成员头像 → 管理成员 | 修改显示名称 → 保存 |
| 定义 Worker 职责 | 聊天输入框 | @成员名 + 职责描述 |
| 广播给所有人 | 群聊输入框 | @All + 消息内容 |
| 私聊某个 Worker | 成员头像 → 发消息 | 在 DM 中对话 |
| 移除 Worker | 成员头像 → 管理成员 | 点击"移出群组" |
| 查看 Worker 终端 | 成员头像 | 点击查看终端输出 |
| 配置全局主题/语言 | 设置页面 | 直接修改 |
| 添加自定义终端 | 设置 → 成员 | 添加自定义命令 |
| 设置技术栈规则 | 聊天 @All 或 CLAUDE.md | 发消息或创建规则文件 |

---

*本文档基于 golutra 源码分析，与实际行为一致。最后更新：2026-04。*
