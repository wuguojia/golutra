# 前端开发者 golutra 实战配置指南

> **目标读者**：前端开发者（Vue / React / Next.js / 小程序等方向）  
> **使用方式**：直接使用 golutra 桌面端成果物（Release 安装包），不折腾源码  
> **目标**：把 AI 多智能体编排能力稳定接入日常开发工作流，提升 1300%+ 协作效率

---

## 目录

- [第一部分：源码工作原理深度解析](#第一部分源码工作原理深度解析)
  - [1.1 整体架构](#11-整体架构)
  - [1.2 核心数据模型](#12-核心数据模型)
  - [1.3 消息分发与编排引擎](#13-消息分发与编排引擎)
  - [1.4 终端会话生命周期](#14-终端会话生命周期)
  - [1.5 数据持久化层](#15-数据持久化层)
  - [1.6 成员邀请流程全链路](#16-成员邀请流程全链路)
- [第二部分：角色体系与配置参数详解](#第二部分角色体系与配置参数详解)
  - [2.1 四大角色类型](#21-四大角色类型)
  - [2.2 终端类型全表](#22-终端类型全表)
  - [2.3 成员配置字段完整说明](#23-成员配置字段完整说明)
- [第三部分：前端开发者多套落地配置方案](#第三部分前端开发者多套落地配置方案)
  - [方案一：个人前端全栈工作流（推荐入门）](#方案一个人前端全栈工作流推荐入门)
  - [方案二：前端 Code Review 监工流水线](#方案二前端-code-review-监工流水线)
  - [方案三：前端组件工厂（批量生产模式）](#方案三前端组件工厂批量生产模式)
  - [方案四：前端测试自动化军团](#方案四前端测试自动化军团)
  - [方案五：全栈独立开发者（一人公司模式）](#方案五全栈独立开发者一人公司模式)
  - [方案六：前端技术调研 & 文档军团](#方案六前端技术调研--文档军团)
- [第四部分：工作流集成最佳实践](#第四部分工作流集成最佳实践)
  - [4.1 日常开发接入流程](#41-日常开发接入流程)
  - [4.2 Prompt 模板库（前端专用）](#42-prompt-模板库前端专用)
  - [4.3 多 Agent 协作模式](#43-多-agent-协作模式)
  - [4.4 常见陷阱与排障](#44-常见陷阱与排障)
- [第五部分：进阶建议与演进方案](#第五部分进阶建议与演进方案)
  - [5.1 golutra-mcp 集成](#51-golutra-mcp-集成)
  - [5.2 自定义工作流模板](#52-自定义工作流模板)
  - [5.3 团队推广策略](#53-团队推广策略)

---

## 第一部分：源码工作原理深度解析

### 1.1 整体架构

golutra 是一个基于 **Vue 3 + Tauri (Rust)** 的本地优先桌面应用，核心理念是 **"一个人 + 一个 AI 军团"**。

```
┌─────────────────────────────────────────────────────────┐
│                    golutra 桌面应用                       │
│  ┌────────────────────┐    ┌──────────────────────────┐ │
│  │   Vue 3 前端层      │    │     Rust + Tauri 后端层   │ │
│  │                    │    │                          │ │
│  │  • Pinia 状态管理   │◄──►│  • 终端引擎              │ │
│  │  • xterm.js 终端    │IPC │  • 编排调度器             │ │
│  │  • Vue i18n 国际化  │    │  • 消息服务              │ │
│  │  • Tailwind UI     │    │  • 项目成员管理           │ │
│  │                    │    │  • ReDB 聊天数据库        │ │
│  └────────────────────┘    └──────────────────────────┘ │
│                                       │                  │
│                              ┌────────┴────────┐        │
│                              │   本地 PTY 终端   │        │
│                              └────────┬────────┘        │
│                    ┌──────────────────┼──────────────┐  │
│                    ▼                  ▼              ▼   │
│              Claude Code        Gemini CLI       Codex   │
│              OpenCode           Qwen Code       Any CLI  │
└─────────────────────────────────────────────────────────┘
```

**关键设计**：golutra 不直接调用 AI API，而是通过 **PTY（伪终端）** 驱动你已安装的 CLI 工具。这意味着：
- 你继续用自己的 API Key / 订阅
- CLI 工具的所有能力（上下文、MCP 插件等）完整保留
- golutra 只负责编排和监控

### 1.2 核心数据模型

#### Member（成员）类型定义

这是你在 golutra 里配置 AI 军团的核心数据结构：

```typescript
type MemberRole = 'owner' | 'admin' | 'assistant' | 'member';
type MemberStatus = 'online' | 'working' | 'dnd' | 'offline';
type TerminalType = 'shell' | 'codex' | 'gemini' | 'claude' | 'opencode' | 'qwen';

type Member = {
  id: string;                    // ULID 唯一标识
  name: string;                  // 显示名称，如 "myproject-assistant-claude-1"
  role: string;                  // 自定义角色描述
  roleKey?: string;              // i18n 翻译键
  roleType: MemberRole;          // 角色类型：owner | admin | assistant | member
  avatar: string;                // 头像，如 "css:orbit"、"css:ember"
  status: MemberStatus;          // 当前状态
  terminalStatus?: string;       // 终端连接状态
  terminalType?: TerminalType;   // 终端类型（决定使用哪个 CLI）
  terminalCommand?: string;      // 终端命令（如 "claude"、"gemini"）
  terminalPath?: string;         // 可执行文件路径
  autoStartTerminal?: boolean;   // 邀请后自动启动终端
  unlimitedAccess?: boolean;     // 无限制模式（绕过使用限制）
  sandboxed?: boolean;           // 沙盒模式
};
```

#### ProjectData（项目数据）结构

每个工作区对应一份 `project-data.json`，保存在 `~/.golutra/` 下：

```json
{
  "projectId": "workspace-id",
  "version": 1,
  "members": [
    { "id": "...", "roleType": "owner", "name": "Owner", ... },
    { "id": "...", "roleType": "assistant", "terminalType": "claude", ... }
  ],
  "memberSequence": {
    "myproject-assistant-claude": 3
  },
  "terminal": { "recentClosedTabs": [] },
  "roadmap": { "objective": "", "tasks": [] },
  "skills": { "current": [] }
}
```

### 1.3 消息分发与编排引擎

这是 golutra 的核心价值所在。当你在聊天界面发送消息时：

```
用户输入 → 前端 ChatInput → Tauri IPC → Rust 编排层 → PTY 终端 → CLI 工具
                                            │
                                    ┌───────┴───────┐
                                    ▼               ▼
                              目标解析          消息批处理
                         (dispatch.rs)    (chat_dispatch_batcher.rs)
```

**目标解析逻辑**（`orchestration/dispatch.rs`）：

| 会话类型 | @提及 | 分发规则 |
|---------|-------|---------|
| 私聊 (DM) | — | 直接发给对方 |
| 群聊 | @All | 发给所有成员（除自己） |
| 群聊 | @具体成员 | 只发给被提及的成员 |

**消息批处理器**（`chat_dispatch_batcher.rs`）：

解决关键问题：同一个 CLI 终端不能同时接收多条输入。

```
消息1 ──┐                    ┌── 合并后发送 ──→ CLI
消息2 ──┤ 同一发送者+同一终端 ──┤
消息3 ──┘                    └── 去重 + 幂等检查
```

核心规则：
- **同发送者同会话** → 合并为一条批处理
- **不同发送者** → 排队等待，前一条完成后再发
- **幂等保护** → 消息 ID 冲突时自动跳过，防止重复发送

### 1.4 终端会话生命周期

```
Pending → Connecting → Connected → Working → Connected (循环)
                                      ↓
                                   Exited
```

**关键时间参数**（来自源码）：
- `STATUS_WORKING_SILENCE_TIMEOUT_MS = 4500ms`：Working 状态下静默 4.5 秒 → 回到 Online
- `CHAT_SILENCE_TIMEOUT_MS = 3000ms`：输出稳定 3 秒 → 认为回复完成
- `COMMAND_CONFIRM_DELAY_MS = 100ms`：发送命令后 100ms 追加回车确认

**完整分发序列**：
1. 消息入队到 Batcher
2. 如终端不存在 → 创建 PTY 会话
3. 文本写入 PTY
4. 等 100ms 后发送回车 (CR)
5. 终端输出响应
6. 语义检测器 (semantic_worker) 判断完成
7. 输出写入聊天数据库
8. 前端通过 Tauri 事件接收并渲染

### 1.5 数据持久化层

```
~/.golutra/
├── workspace-registry.json        # 工作区 ID ↔ 路径映射
├── workspace-registry.lock        # 跨进程文件锁
├── recent-workspaces.json         # 最近打开的工作区
├── avatar-library.json            # 头像资源索引
├── avatars/                       # 头像图片存储
├── chat-db/                       # ReDB 数据库（消息、会话）
└── [workspace-data]/
    └── project-data.json          # 每个工作区的成员/路线图/技能
```

**存储分层**：
| 层 | 技术 | 内容 |
|----|------|------|
| L1 文件系统 | JSON | 工作区注册表、项目配置、头像 |
| L2 嵌入式数据库 | ReDB | 聊天消息、会话元数据 |
| L3 内存 | Rust HashMap | 活跃终端会话、输出缓冲（2000 行回滚） |

### 1.6 成员邀请流程全链路

这是你配置 "监工/成员/助手" 的核心路径：

```
InviteAssistantModal.vue
    │
    ├── 选择模型（Claude / Gemini / Codex / ...）
    ├── 设置实例数量（1-20）
    ├── 开关：无限制模式 (unlimitedAccess)
    ├── 开关：沙盒模式 (sandboxed)
    │
    ▼
useFriendInvites.ts → inviteProjectMembers()
    │
    ▼  Tauri IPC
project_members.rs (Rust)
    │
    ├── 生成基础名称: "{workspace}-{roleType}-{terminalLabel}"
    ├── 追加序号: "-1", "-2", "-3" ...
    ├── 哈希生成头像: css:orbit / css:ember / css:mint / css:canyon / css:storm
    ├── 写入 project-data.json
    │
    ▼
返回前端 → 更新 Pinia Store → 自动创建终端会话 → CLI 就绪
```

**命名规则示例**：
- 工作区名 `my-vue-app`，邀请 3 个 Claude 助手：
  - `my-vue-app-assistant-claude-1`
  - `my-vue-app-assistant-claude-2`
  - `my-vue-app-assistant-claude-3`

---

## 第二部分：角色体系与配置参数详解

### 2.1 四大角色类型

| 角色 | roleType | 中文名 | 说明 | 终端支持 | 多实例 |
|------|----------|--------|------|---------|--------|
| **群主** | `owner` | 群主 | 项目创建者，自动生成 | ❌ | ❌ |
| **管理员** | `admin` | 管理员 | 有服务器管理权限 | ❌ | ❌ |
| **AI 助手** | `assistant` | AI 助手 | 绑定 CLI 工具的 AI 代理 | ✅ | ✅ 1-20 |
| **成员** | `member` | 成员 | 普通项目成员 | ✅（可选）| ❌ |

> **前端开发者最常用的是 `assistant` 角色**——它就是你的 AI 军团主力。

### 2.2 终端类型全表

| 终端类型 | terminalType | 默认命令 | 适用场景 |
|---------|-------------|---------|---------|
| Claude Code | `claude` | `claude` | 代码编写、重构、CR |
| Gemini CLI | `gemini` | `gemini` | 代码分析、文档生成 |
| Codex CLI | `codex` | `codex` | 代码补全、快速原型 |
| OpenCode | `opencode` | `opencode` | 开源模型推理 |
| Qwen Code | `qwen` | `qwen` | 中文场景优化 |
| 系统终端 | `shell` | `bash`/`zsh` | 脚本执行、构建命令 |

> **自定义 CLI**：任何命令行工具都可以通过 `shell` 类型 + 自定义 `terminalCommand` 接入。

### 2.3 成员配置字段完整说明

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `terminalType` | TerminalType | 自动推断 | 终端类型，决定图标和行为 |
| `terminalCommand` | string | 模型名 | CLI 启动命令 |
| `terminalPath` | string | — | 可执行文件完整路径（可选） |
| `autoStartTerminal` | boolean | `true`（有终端时） | 邀请后自动启动 |
| `unlimitedAccess` | boolean | `true`（助手默认） | 无限制模式 |
| `sandboxed` | boolean | `false` | 沙盒隔离 |
| `instances` | number | `1` | 实例数量（1-20） |

**unlimitedAccess（无限制模式）**：
- 开启后绕过使用频率限制
- 建议：主力 AI 助手开启，辅助角色可关闭以节约资源

**sandboxed（沙盒模式）**：
- 开启后 AI 在隔离环境运行
- 建议：处理未验证代码、安全审查时开启

---

## 第三部分：前端开发者多套落地配置方案

> 以下所有方案均基于 golutra 桌面端成果物操作，**不需要修改源码**。
> 操作路径：打开 golutra → 创建/选择工作区 → 点击邀请按钮 → 按配置邀请成员。

---

### 方案一：个人前端全栈工作流（推荐入门）

> **适用场景**：日常前端开发，一个人搞定编码 + 测试 + 文档  
> **难度**：⭐  
> **所需 CLI**：Claude Code + 系统终端

#### 成员配置表

| 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 职责 |
|---------|----------|-------------|-----------------|-----------|----------------|-----------|------|
| 👑 你自己 | `owner` | — | — | 1 | — | — | 监工 + 需求下发 |
| 🤖 编码助手 | `assistant` | `claude` | `claude` | 2 | ✅ | ❌ | Vue/React 组件开发、CSS 编写 |
| 🧪 测试助手 | `assistant` | `claude` | `claude` | 1 | ✅ | ❌ | 单元测试 + E2E 测试编写 |
| 📝 Shell 执行器 | `assistant` | `shell` | `bash` | 1 | ✅ | ❌ | 运行构建/lint/测试命令 |

#### 使用流程

```
1. 创建工作区，指向你的前端项目目录
2. 邀请 2 个 Claude 助手（编码用）
3. 邀请 1 个 Claude 助手（测试用）
4. 邀请 1 个 Shell 成员（执行构建命令）

日常使用：
  你（群聊）: "@编码助手-1 帮我用 Vue 3 Composition API 实现一个分页组件"
  你（群聊）: "@编码助手-2 同时帮我写 UserProfile 页面的响应式布局"
  （两个助手并行工作）
  你（私聊测试助手）: "给分页组件写 Vitest 单元测试"
  你（私聊 Shell）: "pnpm test && pnpm lint"
```

#### 监工策略

- **点击头像**查看每个助手的实时终端输出
- **注入提示词**：在终端流中直接追加修改指令，不需要等回复完
- 观察 `Working` → `Online` 状态变化判断任务完成

---

### 方案二：前端 Code Review 监工流水线

> **适用场景**：需要多角度审查代码质量的团队 / 个人  
> **难度**：⭐⭐  
> **所需 CLI**：Claude Code + Gemini CLI

#### 成员配置表

| 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 职责 |
|---------|----------|-------------|-----------------|-----------|----------------|-----------|------|
| 👑 监工（你） | `owner` | — | — | 1 | — | — | 分发代码、汇总意见 |
| 🔍 安全审查员 | `assistant` | `claude` | `claude` | 1 | ✅ | ✅ | XSS/CSRF/注入漏洞检查 |
| 📐 架构审查员 | `assistant` | `gemini` | `gemini` | 1 | ✅ | ❌ | 组件设计、状态管理合理性 |
| 🎨 UI/UX 审查员 | `assistant` | `claude` | `claude` | 1 | ✅ | ❌ | 可访问性、响应式、一致性 |
| ⚡ 性能审查员 | `assistant` | `gemini` | `gemini` | 1 | ✅ | ❌ | Bundle 分析、渲染性能 |

#### 使用流程

```
你（群聊 @All）:
  "以下是本次 PR 的改动文件，请各自从自己的角度审查：
   - src/components/DataTable.vue
   - src/composables/useVirtualScroll.ts
   - src/utils/sanitize.ts
   
   安全审查员重点看 sanitize.ts 的 XSS 防护
   架构审查员看 composable 的设计
   UI 审查员看 DataTable 的可访问性
   性能审查员看虚拟滚动实现"

（4个助手并行审查，各自输出意见）

你：汇总意见 → 修改代码 → 再次 @All 复审
```

#### 监工策略

- 安全审查员开启 **sandboxed** 模式，防止恶意代码执行
- 用 **群聊 @All** 一次性下发，4 个 AI 并行审查
- 汇总时对比不同 AI 的意见，交叉验证

---

### 方案三：前端组件工厂（批量生产模式）

> **适用场景**：UI 组件库建设、设计系统落地、大量页面开发  
> **难度**：⭐⭐  
> **所需 CLI**：Claude Code

#### 成员配置表

| 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 职责 |
|---------|----------|-------------|-----------------|-----------|----------------|-----------|------|
| 👑 产品监工（你） | `owner` | — | — | 1 | — | — | 组件规格定义、验收 |
| 🏭 组件工人 A-E | `assistant` | `claude` | `claude` | 5 | ✅ | ❌ | 并行编写不同组件 |
| 📋 类型工人 | `assistant` | `claude` | `claude` | 1 | ✅ | ❌ | TypeScript 类型定义 |
| 📖 文档工人 | `assistant` | `claude` | `claude` | 1 | ✅ | ❌ | Storybook / 组件文档 |

#### 使用流程

```
你分别私聊 5 个组件工人，同时下发任务：

@工人-1: "用 Vue 3 + TypeScript 写 Button 组件，支持 primary/secondary/ghost 变体"
@工人-2: "写 Modal 组件，支持 teleport、ESC 关闭、焦点陷阱"
@工人-3: "写 Tabs 组件，支持路由模式和受控模式"
@工人-4: "写 Select 组件，支持搜索、多选、虚拟滚动"
@工人-5: "写 DatePicker 组件，支持范围选择"

（5 个助手并行工作，你轮流查看进度）

完成后：
@类型工人: "为以上 5 个组件生成统一的 TypeScript 类型导出文件"
@文档工人: "为所有组件生成 Storybook stories 和 README"
```

#### 监工策略

- **5 个并行 = 5 倍速度**，关键是给每个工人清晰的规格
- 提前准备**组件规格模板**（见第四部分 Prompt 模板）
- 通过点击头像查看实时进度，发现问题立即在终端注入修正

---

### 方案四：前端测试自动化军团

> **适用场景**：提高测试覆盖率、补全历史遗留代码的测试  
> **难度**：⭐⭐⭐  
> **所需 CLI**：Claude Code + Shell

#### 成员配置表

| 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 职责 |
|---------|----------|-------------|-----------------|-----------|----------------|-----------|------|
| 👑 QA 监工（你） | `owner` | — | — | 1 | — | — | 分配测试任务、验收 |
| 🧪 单测专家 | `assistant` | `claude` | `claude` | 3 | ✅ | ❌ | Vitest / Jest 单元测试 |
| 🌐 E2E 专家 | `assistant` | `claude` | `claude` | 2 | ✅ | ❌ | Playwright / Cypress E2E |
| ⚙️ 执行器 | `assistant` | `shell` | `bash` | 2 | ✅ | ❌ | 运行测试 + 收集覆盖率 |

#### 使用流程

```
第一轮：分配测试任务
@单测专家-1: "为 src/utils/ 下所有工具函数写单元测试"
@单测专家-2: "为 src/composables/ 下所有 composable 写单元测试"
@单测专家-3: "为 src/stores/ 下所有 Pinia store 写单元测试"
@E2E专家-1: "为登录流程写 Playwright E2E 测试"
@E2E专家-2: "为购物车流程写 Playwright E2E 测试"

第二轮：运行验证
@执行器-1: "cd /project && pnpm test -- --coverage"
@执行器-2: "cd /project && npx playwright test"

第三轮：根据结果修复
@单测专家-1: "执行器报告以下测试失败，请修复：..."
```

#### 监工策略

- 3 个单测 + 2 个 E2E 并行 = 全面覆盖
- 先写测试，后运行，再修复 → **三轮流水线**
- 关注执行器的终端输出，覆盖率不达标就追加任务

---

### 方案五：全栈独立开发者（一人公司模式）

> **适用场景**：独立开发者需要全栈交付，前后端 + DevOps 全包  
> **难度**：⭐⭐⭐  
> **所需 CLI**：Claude Code + Gemini CLI + Shell

#### 成员配置表

| 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 职责 |
|---------|----------|-------------|-----------------|-----------|----------------|-----------|------|
| 👑 CEO（你） | `owner` | — | — | 1 | — | — | 总指挥 |
| 🎨 前端开发 | `assistant` | `claude` | `claude` | 2 | ✅ | ❌ | Vue/React 页面与组件 |
| ⚙️ 后端开发 | `assistant` | `claude` | `claude` | 1 | ✅ | ❌ | Node.js / Python API |
| 📊 数据库设计 | `assistant` | `gemini` | `gemini` | 1 | ✅ | ❌ | Schema 设计、SQL |
| 🚀 DevOps | `assistant` | `shell` | `bash` | 1 | ✅ | ❌ | Docker、CI/CD、部署 |
| 📝 产品文档 | `assistant` | `gemini` | `gemini` | 1 | ✅ | ❌ | API 文档、README |
| 🔍 代码审查 | `assistant` | `claude` | `claude` | 1 | ✅ | ✅ | 安全审查 + 质量把关 |

#### 使用流程

```
Sprint 启动（群聊 @All）:
"本周 Sprint 目标：完成用户管理模块
 - 前端开发：用户列表页 + 用户详情页
 - 后端开发：/api/users CRUD
 - 数据库：users 表设计
 - DevOps：准备 Docker 开发环境"

并行执行：
@数据库: "设计 users 表，包含 id, email, name, avatar, role, created_at"
@后端: "用 Express.js 实现 /api/users CRUD，连接 PostgreSQL"
@前端-1: "用 Vue 3 实现用户列表页，对接 /api/users"
@前端-2: "用 Vue 3 实现用户详情页，对接 /api/users/:id"
@DevOps: "写 docker-compose.yml，包含 Node + PostgreSQL"

代码完成后：
@代码审查: "审查所有新增文件的安全性和代码质量"
@文档: "为 users API 生成 OpenAPI 文档"
```

---

### 方案六：前端技术调研 & 文档军团

> **适用场景**：技术选型、方案调研、知识库建设  
> **难度**：⭐  
> **所需 CLI**：Claude Code + Gemini CLI

#### 成员配置表

| 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 职责 |
|---------|----------|-------------|-----------------|-----------|----------------|-----------|------|
| 👑 技术总监（你） | `owner` | — | — | 1 | — | — | 发起调研、汇总决策 |
| 🔬 Claude 研究员 | `assistant` | `claude` | `claude` | 2 | ✅ | ❌ | 深度技术分析 |
| 🔬 Gemini 研究员 | `assistant` | `gemini` | `gemini` | 2 | ✅ | ❌ | 对比分析、数据搜集 |

#### 使用流程

```
你（群聊 @All）:
"调研前端状态管理方案，每人从不同角度分析：
 @Claude研究员-1: Pinia vs Zustand 架构对比
 @Claude研究员-2: Pinia vs Jotai 性能基准测试方案
 @Gemini研究员-1: 各方案的社区活跃度、Stars、维护状况
 @Gemini研究员-2: 各方案在 SSR/SSG 场景的兼容性"

（4 个研究员并行调研）

你：汇总 4 份报告 → 做出选型决策 → 输出技术选型文档
```

#### 监工策略

- 不同 AI 模型有不同的知识偏好，**交叉验证**结果更可靠
- Claude 擅长深度分析，Gemini 擅长信息检索
- 汇总后用 @All 让所有人 Review 最终方案

---

## 第四部分：工作流集成最佳实践

### 4.1 日常开发接入流程

```
每日流程：

07:00  打开 golutra → 选择工作区（指向项目目录）
       ├── 所有配置好的 AI 助手自动上线（autoStartTerminal: true）
       └── 查看昨天的对话记录（ReDB 持久化）

08:00  规划今日任务 → 写入 Roadmap
       ├── 在 golutra 的 Roadmap 面板添加任务
       └── @All 告知今日计划

09:00-12:00  并行开发
       ├── 不同助手处理不同任务
       ├── 你作为监工轮流查看进度
       └── 发现问题 → 终端注入修正指令

14:00-17:00  测试 + 审查
       ├── 测试助手写测试
       ├── Shell 执行器跑测试
       └── 审查助手 Code Review

17:00  收尾
       ├── 汇总当天成果
       ├── 更新 Roadmap 状态
       └── 关闭工作区（数据自动保存）
```

### 4.2 Prompt 模板库（前端专用）

以下模板可直接在 golutra 聊天中使用：

#### 组件开发模板

```
请用 Vue 3 Composition API + TypeScript 实现 {组件名} 组件：

技术栈：Vue 3.5+、TypeScript 5.x、Tailwind CSS 3.x
文件结构：
  - {组件名}.vue（SFC 单文件组件）
  - {组件名}.test.ts（Vitest 单元测试）
  - types.ts（TypeScript 类型导出）

功能需求：
  1. {功能点1}
  2. {功能点2}
  3. {功能点3}

非功能需求：
  - 支持键盘导航（a11y）
  - 响应式适配（mobile-first）
  - 暗色模式支持
  - Props 使用 defineProps + withDefaults
  - 事件使用 defineEmits 类型声明
```

#### Code Review 模板

```
请审查以下代码，重点关注：

1. 安全性：XSS、innerHTML 使用、v-html 防护
2. 性能：不必要的 re-render、computed 缓存
3. TypeScript：类型安全、any 使用
4. 可访问性：ARIA 属性、语义化 HTML
5. 最佳实践：组合式 API 使用、响应式数据管理

文件：{文件路径}
```

#### Bug 修复模板

```
Bug 描述：{描述}
复现步骤：
  1. {步骤1}
  2. {步骤2}
期望行为：{期望}
实际行为：{实际}

请分析根因并修复。修复后请确保：
  - 不引入新的 TypeScript 错误
  - 补充回归测试
  - 解释修复思路
```

### 4.3 多 Agent 协作模式

#### 模式一：流水线模式

```
Agent A（设计） → Agent B（编码） → Agent C（测试） → Agent D（审查）
```

适用于：功能开发全流程

#### 模式二：并行模式

```
Agent A ─┐
Agent B ──┤──→ 你（汇总）
Agent C ──┤
Agent D ─┘
```

适用于：Code Review、技术调研

#### 模式三：主从模式

```
你 ──→ Agent A（主力）
         │
         ├──→ 你 review
         │
         ▼
       Agent B（补充/修正）
```

适用于：复杂功能迭代

#### 模式四：竞赛模式

```
Agent A ──→ 方案 1 ─┐
Agent B ──→ 方案 2 ──┤──→ 你（选最优）
Agent C ──→ 方案 3 ─┘
```

适用于：技术选型、方案对比

### 4.4 常见陷阱与排障

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 助手一直是 `Working` 状态 | CLI 等待确认输入 | 在终端注入 `y` 或 `yes` |
| 消息没有发到指定助手 | @提及语法错误 | 确保使用正确的成员名称 |
| 终端显示 `Disconnected` | CLI 工具未安装/崩溃 | 检查 CLI 是否在系统 PATH 中 |
| 多条消息合并了 | Batcher 合并了同发送者消息 | 等前一条回复后再发下一条 |
| 助手回复被截断 | 终端缓冲区满 | 查看终端 scrollback（2000 行） |
| 邀请后助手没上线 | `autoStartTerminal` 为 false | 手动点击启动终端 |

**运行日志调试**：
- 在环境变量中设置 `GOLUTRA_TERMINAL_TRACE=1` 可开启终端追踪
- `GOLUTRA_TERMINAL_TRACE_DETAIL=1` 开启详细追踪
- `VITE_TERMINAL_TRACE=1` 开启前端终端追踪

---

## 第五部分：进阶建议与演进方案

### 5.1 golutra-mcp 集成

golutra 有配套的 MCP（Model Context Protocol）仓库 [`golutra-mcp`](https://github.com/golutra/golutra-mcp)，提供更稳定的 CLI 连接方式：

```
golutra 桌面端 ←──→ golutra-cli ←──→ golutra-mcp
```

**建议**：
- 先用桌面端直接使用，熟悉后再接入 MCP
- MCP 适合需要跨工具集成的高级场景
- 关注 `golutra-mcp` 仓库的更新

### 5.2 自定义工作流模板

golutra 支持 **工作流模板一键导入导出**，这是前端开发者的杀手级功能：

**建议创建以下模板**：
1. **Vue 组件开发模板**：预配置 2 编码 + 1 测试 + 1 Shell
2. **React 项目模板**：预配置 3 编码 + 2 测试 + 1 审查
3. **Bug Fix 冲刺模板**：预配置 3 调试专家 + 1 测试
4. **技术文档模板**：预配置 2 研究员 + 1 文档编写

**导出分享**：制作好的模板可以导出分享给团队成员

### 5.3 团队推广策略

如果你想在前端团队中推广 golutra：

**第一阶段：个人试点（1-2 周）**
- 自己先用方案一跑起来
- 记录效率提升数据（对比有/无 golutra）
- 收集典型使用场景的截图

**第二阶段：小组试验（2-4 周）**
- 选 2-3 个同事一起用
- 共享你的工作流模板
- 建立内部 Prompt 模板库

**第三阶段：团队标准化（1-2 月）**
- 制定团队标准工作流模板
- 将模板纳入团队 Wiki
- 定期分享最佳实践

### 5.4 额外建议

#### CLI 工具安装优先级

作为前端开发者，建议按以下顺序安装 CLI 工具：

1. **Claude Code**（首选）—— 代码质量最高，对前端生态理解最深
2. **Gemini CLI** —— 信息检索能力强，适合调研和文档
3. **Codex CLI** —— 快速原型和代码补全
4. 其他根据需要添加

#### 资源优化建议

- 不要一次性开太多实例（建议总数 ≤ 8）
- 不用的助手设为 `offline` 状态节省资源
- 长时间不用的工作区可以关闭
- `unlimitedAccess` 只在主力助手上开启

#### 安全注意事项

- 处理敏感代码时开启 `sandboxed` 模式
- 定期检查 AI 生成的代码中是否有硬编码密钥
- 避免将生产环境凭证暴露在聊天记录中
- 聊天数据库存储在本地（`~/.golutra/chat-db/`），注意备份

#### 版本与更新

- 关注 [GitHub Releases](https://github.com/golutra/golutra/releases) 获取最新版本
- 项目遵循 BSL 1.1 许可证，使用 golutra 作为工具构建商业软件是允许的
- 你通过 golutra 产出的代码归你所有

---

## 速查卡片

### 快速配置速查

```
最小配置（入门）:
  2x Claude 编码助手 + 1x Shell 执行器

标准配置（日常）:
  3x Claude 编码助手 + 1x Claude 测试助手 + 1x Shell 执行器

完整配置（大型任务）:
  5x Claude 编码助手 + 2x Claude 测试助手 + 2x Gemini 审查员 + 2x Shell 执行器
```

### 状态含义速查

```
🟢 Online    = 就绪，等待指令
🔵 Working   = 正在执行任务
🔴 DND       = 请勿打扰
⚫ Offline   = 离线 / 未启动
```

### 聊天命令速查

```
@成员名    = 私聊或在群聊中指定对象
@All      = 群聊中通知所有成员
点击头像   = 查看该成员的终端实时输出
终端注入   = 在终端流中直接追加指令
```

---

> 💡 **最后建议**：先从方案一开始，用最少配置跑通流程，感受 golutra 的并行能力。然后根据实际需求逐步升级到更复杂的方案。记住核心理念：**你是监工，AI 是员工，golutra 是管理工具。**

---

*文档基于 golutra v0.1.0 源码分析，最后更新：2026-04*
