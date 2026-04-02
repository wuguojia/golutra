# golutra 完整配置与使用指南

> **本指南面向**：只使用 golutra 成品（不改源码）的用户  
> **目标**：从零开始，通过页面操作配置出你的 AI 团队，并高效使用  
> **前置要求**：已安装 golutra 桌面端 + 至少一个 CLI 工具（Claude Code / Gemini CLI / Codex CLI 等）

---

## 📖 目录

- [一、核心概念（3 分钟理解）](#一核心概念3-分钟理解)
- [二、页面操作全流程](#二页面操作全流程)
- [三、"邀请助手"弹窗 —— 你只需要配 4 个东西](#三邀请助手弹窗--你只需要配-4-个东西)
- [四、15 个 Worker 完整配置速查表](#四15-个-worker-完整配置速查表)
- [五、每个 Worker 详细说明](#五每个-worker-详细说明)
- [六、7 个方案选择指南](#六7-个方案选择指南)
- [七、技术栈如何"智能化"—— 不改代码的实现方式](#七技术栈如何智能化--不改代码的实现方式)
- [八、实战操作示例](#八实战操作示例)
- [九、常见问题 FAQ](#九常见问题-faq)

---

## 一、核心概念（3 分钟理解）

### 1.1 golutra 是什么？

golutra 是一个**本地桌面应用**，它通过伪终端（PTY）驱动你电脑上已安装的 CLI 工具（Claude Code、Gemini CLI 等），让多个 AI 同时并行工作。

**类比**：你是老板（监工），golutra 是办公室，AI 助手是你的员工。你在聊天界面下指令，员工在各自的终端里干活。

### 1.2 关键术语

| 术语 | 含义 |
|------|------|
| **Owner（群主/监工）** | 就是你，负责下指令和做决策 |
| **Worker（AI 员工）** | 一个 AI 助手实例，绑定到某个 CLI 工具 |
| **Terminal Type（终端类型）** | Worker 使用的 CLI 工具类型 |
| **Workspace（工作区）** | 一个项目目录，关联一组 Worker |
| **Scheme（方案）** | 预定义的 Worker 团队组合 |

### 1.3 支持的 CLI 工具

| CLI 工具 | terminalType | 启动命令 | 无限制访问标志 | 擅长什么 |
|---------|-------------|---------|--------------|---------|
| Claude Code | `claude` | `claude` | `--dangerously-skip-permissions` | 深度代码编写、安全分析、细粒度建议 |
| Gemini CLI | `gemini` | `gemini` | `--yolo` | 架构思考、性能分析、信息检索 |
| Codex CLI | `codex` | `codex` | `--dangerously-bypass-approvals-and-sandbox` | 代码生成、会话恢复 |
| OpenCode | `opencode` | `opencode` | 无 | 开源模型代码生成 |
| Qwen Code | `qwen` | `qwen` | `--yolo` | 中文理解、代码编写 |
| Terminal (Shell) | `shell` | `bash` | 无 | 纯命令执行（不是 AI） |

> **重要**：golutra 本身不提供 AI 模型，你需要自己安装上述 CLI 工具并配置好 API Key。

---

## 二、页面操作全流程

### 2.1 创建工作区

1. 打开 golutra 桌面端
2. 点击**创建工作区**（或导入已有项目目录）
3. 选择你的项目文件夹路径
4. golutra 会自动创建 `.golutra/workspace.json` 存储配置

### 2.2 邀请 AI 成员（核心操作）

1. 在工作区的聊天界面，找到**成员列表**区域
2. 点击 **"邀请助手"** 按钮（或 `+` 号）
3. 在弹出的**邀请助手弹窗**中配置（详见[第三章](#三邀请助手弹窗--你只需要配-4-个东西)）
4. 点击**发送邀请**
5. Worker 自动创建并启动终端

### 2.3 与 AI 成员对话

- **群聊**：直接输入消息，所有成员都能看到
- **@某人**：输入 `@成员名` 指定某个 Worker 执行任务
- **@All**：所有 AI 成员同时收到并并行处理
- **私聊**：点击成员头像进入 1v1 对话

### 2.4 监控工作状态

- **头像状态**：
  - 🟢 Online = 空闲等待指令
  - 🔵 Working = 正在执行任务
  - ⚫ Offline = 未启动
- 点击头像可查看该 Worker 的**实时终端输出**
- 可在终端中**直接注入提示词**修正方向

---

## 三、"邀请助手"弹窗 —— 你只需要配 4 个东西

打开**邀请助手**弹窗后，你会看到以下配置项。**就这 4 个，没有更多的了**：

### 3.1 配置项速查

| # | 配置项 | 界面位置 | 选什么 | 默认值 |
|---|-------|---------|-------|-------|
| ① | **模型选择** | 弹窗上方列表 | 选一个 CLI 工具 | 按设置中的默认索引 |
| ② | **实例数量** | 中部数字输入 | 1-20 | 1 |
| ③ | **无限制访问** | 开关按钮 | 开/关 | ✅ 开启 |
| ④ | **沙盒模式** | 开关按钮 | 开/关 | ❌ 关闭 |

### 3.2 每个配置项详细说明

#### ① 模型选择（选 CLI 工具）

弹窗顶部显示 6 个内置选项 + 你自定义的终端：

| 显示名 | 对应 CLI | 图标颜色 | 什么时候选它 |
|-------|---------|---------|------------|
| Gemini CLI | `gemini` | 天蓝色 | 架构审查、性能分析、技术研究、数据库设计、文档编写 |
| Codex | `codex` | 翠绿色 | 代码生成（支持会话恢复） |
| Claude Code | `claude` | 橙色 | 编码、安全审查、测试编写、需求分析 |
| OpenCode | `opencode` | 靛蓝色 | 使用开源模型的代码生成 |
| Qwen Code | `qwen` | 青绿色 | 中文场景的代码编写 |
| Terminal | `shell` | 灰色 | 纯命令执行（构建/测试/部署，不调用 AI） |

> **注意**：邀请助手（assistant 角色）时，Terminal 选项默认隐藏。如果需要 Shell 执行器，请从**成员（member）**角色入口邀请，或切换邀请模式。

**选择建议**：
- 写代码 → Claude Code（编码能力最强）
- 做审查/研究 → Gemini CLI（架构级思维好）
- 跑命令 → Terminal（纯 Shell，不消耗 AI 额度）
- 不确定 → 先选 Claude Code，后面可以再加其他的

#### ② 实例数量（开几个并行）

| 设置值 | 含义 | 建议场景 |
|-------|------|---------|
| 1 | 1 个 Worker | 审查类、研究类、文档类（1 个就够） |
| 2 | 2 个并行 | 编码类（2 个写不同页面/模块） |
| 3-5 | 多个并行 | 大型项目的组件批量开发 |
| 5-20 | 大量并行 | 批量测试编写、组件工厂 |

> **注意**：每个实例都会启动一个独立终端进程。设过高会占用较多系统资源和 API 额度。

#### ③ 无限制访问（Unlimited Access）

| 状态 | 含义 | 效果 |
|------|------|------|
| ✅ 开启 | AI 可以自动执行命令 | Claude 自动加 `--dangerously-skip-permissions` |
| | | Gemini 自动加 `--yolo` |
| | | Codex 自动加 `--dangerously-bypass-approvals-and-sandbox` |
| | | Qwen 自动加 `--yolo` |
| ❌ 关闭 | AI 每次执行命令前需要确认 | 更安全但更慢 |

**建议**：
- 编码助手、测试专家 → **开启**（频繁执行命令，关闭的话每步都要确认很烦）
- 安全审查员 → **开启**（但搭配沙盒使用）
- 不信任的场景 → **关闭**

#### ④ 沙盒模式（Sandboxed）

| 状态 | 含义 |
|------|------|
| ❌ 关闭（默认） | Worker 可以正常读写项目文件、执行命令 |
| ✅ 开启 | Worker 在隔离环境运行，限制对系统的访问 |

**建议**：
- **只有安全审查员需要开启沙盒** —— 因为它可能分析恶意代码片段
- **其他所有角色全部关闭** —— 否则无法正常读写代码

### 3.3 配置决策流程图

```
你想让它干什么？
│
├── 写代码 / 改 Bug / 重构
│   → 模型: Claude Code
│   → 实例: 2-3
│   → 无限制: ✅
│   → 沙盒: ❌
│
├── 审查代码安全性
│   → 模型: Claude Code
│   → 实例: 1
│   → 无限制: ✅
│   → 沙盒: ✅  ← 唯一需要沙盒的
│
├── 审查架构 / 性能
│   → 模型: Gemini CLI
│   → 实例: 1
│   → 无限制: ✅
│   → 沙盒: ❌
│
├── 写测试
│   → 模型: Claude Code
│   → 实例: 1-3
│   → 无限制: ✅
│   → 沙盒: ❌
│
├── 跑命令（构建/测试/部署）
│   → 模型: Terminal (Shell)
│   → 实例: 1-2
│   → 无限制: ✅
│   → 沙盒: ❌
│
├── 写文档 / 做技术研究
│   → 模型: Gemini CLI
│   → 实例: 1-2
│   → 无限制: ✅
│   → 沙盒: ❌
│
└── 需求分析（AI 项目经理）
    → 模型: Claude Code
    → 实例: 1
    → 无限制: ✅
    → 沙盒: ❌
```

---

## 四、15 个 Worker 完整配置速查表

以下是所有预定义 Worker 角色在**邀请助手弹窗**中应该怎么配：

| # | 角色名 | 选什么模型 | 实例数 | 无限制 | 沙盒 | 一句话说明 |
|---|--------|-----------|--------|--------|------|-----------|
| 1 | 编码助手 | Claude Code | 2-5 | ✅ | ❌ | 前端/后端代码开发主力 |
| 2 | 后端开发 | Claude Code | 1-2 | ✅ | ❌ | API、数据库操作、认证系统 |
| 3 | TypeScript 类型工程师 | Claude Code | 1 | ✅ | ❌ | 类型系统统一、泛型维护 |
| 4 | 安全审查员 | Claude Code | 1 | ✅ | ✅ | XSS/CSRF/注入安全检查 |
| 5 | 架构审查员 | Gemini CLI | 1 | ✅ | ❌ | 组件设计、状态管理评审 |
| 6 | UI/UX 审查员 | Claude Code | 1 | ✅ | ❌ | 可访问性、响应式、用户体验 |
| 7 | 性能审查员 | Gemini CLI | 1 | ✅ | ❌ | Bundle 优化、渲染性能、内存 |
| 8 | 测试专家（单元测试） | Claude Code | 1-3 | ✅ | ❌ | Vitest 单元测试 + 组件测试 |
| 9 | E2E 测试专家 | Claude Code | 1-2 | ✅ | ❌ | Playwright 端到端测试 |
| 10 | Shell 执行器 | Terminal | 1-2 | ✅ | ❌ | 构建/测试/Git/环境检查 |
| 11 | DevOps 工程师 | Terminal | 1 | ✅ | ❌ | Docker/CI/CD/Nginx/部署 |
| 12 | 数据库设计师 | Gemini CLI | 1 | ✅ | ❌ | Schema 设计、SQL、迁移脚本 |
| 13 | 文档工程师 | Gemini CLI | 1 | ✅ | ❌ | API 文档、README、Storybook |
| 14 | 技术研究员 | Claude + Gemini 混合 | 2+2 | ✅ | ❌ | 技术选型、方案对比、调研 |
| 15 | 需求分析师 | Claude Code | 1 | ✅ | ❌ | AI 项目经理，任务分解与调度 |

> **记住**：在弹窗里你只能配 ①模型 ②数量 ③无限制 ④沙盒。角色的"名字"和"职责"是通过你在聊天中发的 Prompt 来定义的。

---

## 五、每个 Worker 详细说明

### 5.1 编码助手（Coding Assistant）

**邀请配置**：
- 模型：Claude Code
- 实例数：2-5（按项目大小调整）
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 前端页面/组件开发（Vue 3、React 等）
- Bug 修复和代码重构
- TypeScript 类型编写

**怎么给它下指令**：
```
@编码助手-1 用 Vue 3 + TypeScript 实现一个用户列表页面：

要求：
- <script setup lang="ts">
- 使用 Pinia 管理状态
- Tailwind CSS 样式
- 支持分页、搜索、排序
- 暗色模式适配
```

**并行用法**：
```
@编码助手-1 负责 UserList 页面
@编码助手-2 负责 UserDetail 页面
```

---

### 5.2 后端开发（Backend Developer）

**邀请配置**：
- 模型：Claude Code
- 实例数：1-2
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- RESTful API 开发（Express / Fastify / Koa）
- JWT 认证、OAuth2
- 数据库操作（Prisma / TypeORM）
- 输入验证（Zod / Joi）

**怎么给它下指令**：
```
@后端开发-1 用 Express + TypeScript 实现用户管理 API：

路由设计：
  GET    /api/users      - 用户列表（分页）
  POST   /api/users      - 创建用户
  GET    /api/users/:id   - 用户详情
  PUT    /api/users/:id   - 更新用户
  DELETE /api/users/:id   - 删除用户

要求：
- TypeScript 严格模式
- Zod 输入验证
- JWT 认证中间件
- 分层架构：Router → Controller → Service → Repository
```

---

### 5.3 安全审查员（Security Reviewer）⚠️ 唯一需要沙盒的角色

**邀请配置**：
- 模型：Claude Code
- 实例数：1
- 无限制访问：✅
- 沙盒：**✅ 开启**

**它能干什么**：
- XSS 漏洞检测（v-html、innerHTML、eval）
- CSRF 防护检查
- 代码注入风险分析
- JWT/Token 存储安全评估
- npm 依赖漏洞扫描

**为什么开沙盒**：安全审查可能涉及分析恶意代码片段，沙盒隔离防止意外执行危险代码。

**怎么给它下指令**：
```
@安全审查员-1 请从安全角度审查以下文件：

文件列表：
  - src/utils/sanitize.ts
  - src/components/CommentRender.vue

重点检查：
1. XSS 风险：v-html、innerHTML、document.write、eval
2. CSRF 防护：API 调用是否携带 token
3. 敏感信息：是否有硬编码密钥、localStorage 存敏感数据
4. 依赖风险：已知漏洞的 npm 包

输出格式：
  🔴 严重 → 必须修复
  🟡 建议 → 推荐修复
  🟢 优化 → 可选
```

---

### 5.4 架构审查员（Architecture Reviewer）

**邀请配置**：
- 模型：Gemini CLI
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 组件设计评审（SRP 原则、粒度控制）
- 状态管理架构评审（Pinia Store 结构）
- Composable 复用性评估
- 循环依赖检测
- 可测试性评估

**怎么给它下指令**：
```
@架构审查员-1 请从架构角度评审以下代码：

文件：src/composables/useVirtualScroll.ts

评审维度：
1. 组件设计：SRP 原则、组件粒度、Props 设计
2. 状态管理：数据流向、副作用隔离
3. Composable 设计：复用性、入参设计、返回值类型
4. 依赖关系：模块间依赖方向、循环依赖
5. 可测试性：纯函数比例、依赖注入点
```

---

### 5.5 UI/UX 审查员（UI/UX Reviewer）

**邀请配置**：
- 模型：Claude Code
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 语义化 HTML 检查
- ARIA 属性审查
- 键盘导航/焦点管理
- 响应式设计（mobile-first）
- 色彩对比度（WCAG AA 标准）
- 暗色模式兼容
- 国际化（RTL 支持）

**怎么给它下指令**：
```
@UIUX审查员-1 请从可访问性角度审查以下组件：

文件：src/components/DataTable.vue

审查清单：
1. 所有交互元素是否可键盘操作？
2. ARIA 角色和属性是否正确？
3. 焦点顺序是否逻辑合理？
4. 色彩对比度是否满足 WCAG AA？
5. 动画是否尊重 prefers-reduced-motion？
```

---

### 5.6 性能审查员（Performance Reviewer）

**邀请配置**：
- 模型：Gemini CLI
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 渲染性能分析（不必要的 re-render）
- Bundle 大小优化（Tree-shaking、Code Splitting）
- 列表性能（虚拟滚动）
- 内存管理（事件监听清理、闭包泄露）
- 图片优化（懒加载、CDN）

**怎么给它下指令**：
```
@性能审查员-1 请从性能角度分析以下代码：

文件：src/components/DataTable.vue

分析维度：
1. 渲染性能：re-render 频率、v-if vs v-show
2. 计算优化：computed 缓存使用
3. 列表优化：是否需要虚拟滚动
4. Bundle 大小：动态导入、Tree-shaking
5. 内存管理：事件监听清理
```

---

### 5.7 测试专家（Test Expert - 单元测试）

**邀请配置**：
- 模型：Claude Code
- 实例数：1-3
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 工具函数单元测试（Vitest）
- Vue Composable 测试
- Pinia Store 测试
- 组件渲染测试

**怎么给它下指令**：
```
@测试专家-1 为 useAuth composable 编写测试：

文件位置：src/composables/useAuth.ts

测试维度：
1. 初始状态验证
2. 登录成功/失败
3. Token 刷新逻辑
4. 登出后状态清理
5. 错误处理分支

使用 Vitest + @vue/test-utils
```

---

### 5.8 E2E 测试专家（End-to-End Test Expert）

**邀请配置**：
- 模型：Claude Code
- 实例数：1-2
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 核心业务流程 E2E 测试（Playwright）
- 异常场景测试（网络断开、Token 过期）
- 视觉回归测试（截图对比）
- 跨浏览器测试

**怎么给它下指令**：
```
@E2E专家-1 用 Playwright 编写用户登录流程的 E2E 测试：

测试文件：tests/e2e/login.spec.ts

用户操作流程：
1. 打开登录页
2. 输入用户名/密码
3. 点击登录
4. 验证跳转到首页
5. 验证用户信息展示

额外场景：
- 密码错误 → 显示错误提示
- 网络断开 → 显示离线提示
```

---

### 5.9 Shell 执行器（Shell Executor）—— 不是 AI

**邀请配置**：
- 模型：Terminal
- 实例数：1-2
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- `pnpm install / build / test / lint`
- `npx playwright test`
- `git status / diff / log`
- 环境检查
- Docker Compose 操作

**⚠️ 重要**：Shell 执行器**不是 AI**，它只是一个系统终端。你需要给它**精确的命令**，不能用自然语言。

**怎么给它下指令**：
```
@Shell执行器-1 pnpm run build && pnpm run test
```

```
@Shell执行器-1 npx playwright test --project=chromium
```

---

### 5.10 DevOps 工程师（DevOps Engineer）

**邀请配置**：
- 模型：Terminal（Shell）
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

> **注意**：DevOps 工程师和 Shell 执行器配置一样（都用 Terminal），区别在于你给它分配的任务不同。如果你需要 AI 帮你写 Dockerfile 而不是执行命令，应该用 Claude Code 或 Gemini CLI。

**它的典型任务**：
- Docker Compose 启动/停止服务
- 运行 CI/CD 脚本
- Nginx 配置验证
- 健康检查

---

### 5.11 数据库设计师（Database Designer）

**邀请配置**：
- 模型：Gemini CLI
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- PostgreSQL/MySQL Schema 设计
- 索引优化和查询分析
- 迁移脚本生成（up/down）
- 种子数据生成
- ER 图和 TypeScript 类型

**怎么给它下指令**：
```
@数据库设计师-1 设计用户管理模块的数据库 Schema：

业务需求：用户注册、登录、个人资料管理

要求：
1. PostgreSQL DDL
2. 主键 UUID
3. 必要索引
4. 软删除（deleted_at）
5. 审计字段（created_at, updated_at）

同时输出：
- Prisma schema 格式
- TypeScript 类型定义
- 种子数据 SQL
```

---

### 5.12 文档工程师（Docs Writer）

**邀请配置**：
- 模型：Gemini CLI
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- OpenAPI 3.0 格式 API 文档
- 组件文档（Props/Events/Slots 表格）
- Storybook Stories
- README 编写
- CHANGELOG 维护

---

### 5.13 技术研究员（Technical Researcher）

**邀请配置**：建议用 **4 个实例**，混合两种 CLI

| 实例 | 模型 | 角色 |
|------|------|------|
| 研究员-Claude-1 | Claude Code ×1 | 原理分析 |
| 研究员-Claude-2 | Claude Code ×1 | 实战对比 |
| 研究员-Gemini-1 | Gemini CLI ×1 | 数据搜集 |
| 研究员-Gemini-2 | Gemini CLI ×1 | 风险评估 |

**操作步骤**：
1. 先邀请 2 个 Claude Code 实例（无限制 ✅，沙盒 ❌）
2. 再邀请 2 个 Gemini CLI 实例（无限制 ✅，沙盒 ❌）
3. 给每个实例分配不同的研究角度

**怎么给它们下指令**：
```
@Claude研究员-1 深度分析 React Server Components 的内部工作原理
@Claude研究员-2 用代码对比 Next.js vs Nuxt 3 实现相同功能
@Gemini研究员-1 搜集 Next.js 和 Nuxt 3 的生态数据（Stars、下载量、采用率）
@Gemini研究员-2 评估从 Nuxt 2 迁移到 Nuxt 3 的风险和成本
```

---

### 5.14 需求分析师（Demand Analyst）—— AI 项目经理

**邀请配置**：
- 模型：Claude Code
- 实例数：1（全局唯一调度者）
- 无限制访问：✅
- 沙盒：❌

**它的特殊角色**：不写代码，而是帮你分析需求、分解任务、协调其他 Worker。

**怎么给它下指令**：
```
@需求分析师-1 你现在是这个工作区的 AI 项目经理。团队成员有：
  - 编码助手-1、编码助手-2（前端开发）
  - 测试专家-1（单元测试）
  - Shell执行器-1（命令执行）

现在接到需求：
"实现一个带搜索、分页、排序的数据表格组件"

请完成：
1. 需求分析
2. 任务分解（每个子任务 15-30 分钟）
3. 分配给对应的团队成员
4. 列出任务依赖关系和并行化方案
5. 提交给我审批
```

---

### 5.15 TypeScript 类型工程师（Type Engineer）

**邀请配置**：
- 模型：Claude Code
- 实例数：1
- 无限制访问：✅
- 沙盒：❌

**它能干什么**：
- 检查所有组件的类型一致性
- 生成统一的 `index.ts` 导出文件
- 编写泛型约束和类型守卫
- 创建项目级工具类型

---

## 六、7 个方案选择指南

### 6.1 方案对照总表

| 方案 | 适合谁 | 团队规模 | 需要什么 CLI | 效率提升 |
|------|--------|---------|-------------|---------|
| **方案一**：个人前端全栈 | 前端开发者日常 | 4 人 | Claude + Shell | 300%-500% |
| **方案二**：Code Review 监工 | 想多角度审查代码 | 4 人 | Claude + Gemini | 400%-600% |
| **方案三**：组件工厂 | 批量造组件 | 7 人 | Claude + Gemini | 500%-800% |
| **方案四**：测试自动化军团 | 补充测试覆盖 | 7 人 | Claude + Shell | 600%-1000% |
| **方案五**：全栈独立开发 | 一人公司/全栈 | 7 人 | Claude + Gemini + Shell | 800%-1300% |
| **方案六**：技术调研文档 | 技术选型 | 4-5 人 | Claude + Gemini | 400%-600% |
| **方案七**：需求驱动智能 | 复杂项目 | 5-10 人 | Claude（核心）+ 按需 | 1000%-1500% |

### 6.2 选择决策树

```
你的场景是？
│
├── 刚入门 golutra，想先试试
│   └── → 方案一（最简单，4 个人）
│
├── 已有代码，想提高质量
│   └── → 方案二（4 个审查员并行）
│
├── 需要批量造 UI 组件
│   └── → 方案三（组件工厂模式）
│
├── 测试覆盖率不够
│   └── → 方案四（测试自动化军团）
│
├── 一个人做全栈项目
│   └── → 方案五（7 个角色全覆盖）
│
├── 需要做技术选型/调研
│   └── → 方案六（交叉验证模式）
│
└── 想要最省心，说需求就行
    └── → 方案七（AI 项目经理调度一切）
```

### 6.3 方案一详细操作步骤（推荐入门）

> **目标**：创建 4 人团队 = 2 编码助手 + 1 测试专家 + 1 Shell 执行器

**步骤 1**：邀请 2 个编码助手
1. 点击"邀请助手"
2. 选择 **Claude Code**
3. 实例数量设为 **2**
4. 无限制访问：**✅ 开启**
5. 沙盒模式：**❌ 关闭**
6. 点击"发送邀请"
7. → 自动创建 `你的项目-assistant-claude-1` 和 `你的项目-assistant-claude-2`

**步骤 2**：邀请 1 个测试专家
1. 点击"邀请助手"
2. 选择 **Claude Code**
3. 实例数量设为 **1**
4. 无限制访问：**✅ 开启**
5. 沙盒模式：**❌ 关闭**
6. 点击"发送邀请"
7. → 自动创建 `你的项目-assistant-claude-3`

**步骤 3**：邀请 1 个 Shell 执行器
1. 点击"邀请助手"（选择 member 类型入口）
2. 选择 **Terminal**
3. 实例数量设为 **1**
4. 无限制访问：**✅ 开启**
5. 沙盒模式：**❌ 关闭**
6. 点击"发送邀请"
7. → 自动创建 `你的项目-assistant-terminal-1`

**步骤 4**：给成员改名（可选但推荐）
1. 点击成员头像 → 管理成员
2. 把 `你的项目-assistant-claude-1` 改名为 `编码助手-1`
3. 把 `你的项目-assistant-claude-2` 改名为 `编码助手-2`
4. 把 `你的项目-assistant-claude-3` 改名为 `测试专家`
5. 把 `你的项目-assistant-terminal-1` 改名为 `Shell执行器`

**步骤 5**：开始使用
```
# 给两个编码助手分配不同页面
@编码助手-1 用 Vue 3 实现用户列表页面 src/pages/UserList.vue
@编码助手-2 用 Vue 3 实现用户详情页面 src/pages/UserDetail.vue

# 等两个都完成后，让测试专家写测试
@测试专家 为 UserList.vue 和 UserDetail.vue 编写单元测试

# 最后用 Shell 跑一下构建验证
@Shell执行器 pnpm run build && pnpm run test
```

### 6.4 方案五详细操作步骤（全栈独立开发）

> **目标**：创建 7 人团队 = 前端×2 + 后端×1 + 数据库×1 + DevOps×1 + 文档×1 + 安全审查×1

| 步骤 | 邀请操作 | 模型 | 数量 | 无限制 | 沙盒 |
|------|---------|------|------|--------|------|
| 1 | 前端开发 | Claude Code | 2 | ✅ | ❌ |
| 2 | 后端开发 | Claude Code | 1 | ✅ | ❌ |
| 3 | 数据库设计师 | Gemini CLI | 1 | ✅ | ❌ |
| 4 | DevOps | Terminal | 1 | ✅ | ❌ |
| 5 | 文档工程师 | Gemini CLI | 1 | ✅ | ❌ |
| 6 | 安全审查 | Claude Code | 1 | ✅ | **✅** |

**Sprint 工作流**：
```
阶段 1 - 基础设施（并行）：
  @数据库设计师 设计数据库 Schema
  @DevOps Docker 环境搭建 + CI/CD 配置

阶段 2 - 核心开发（并行）：
  @后端开发 基于 Schema 实现 API
  @前端开发-1 首页和列表页
  @前端开发-2 详情页和表单页

阶段 3 - 质量保障：
  @安全审查 审查所有代码

阶段 4 - 交付：
  @文档工程师 生成 API 文档和 README
  @DevOps 部署到生产环境
```

---

## 七、技术栈如何"智能化"—— 不改代码的实现方式

### 7.1 当前状态

golutra **目前没有自动检测项目技术栈**的功能。但你可以通过以下方式实现类似效果。

### 7.2 方法一：在第一条消息中注入技术栈上下文（推荐）

当 AI Worker 启动后，它会收到一个默认的 onboarding 提示（告诉它自己的名字）。你可以**在第一条指令中补充技术栈信息**：

```
@All 本项目的技术栈信息，请所有人记住：

项目名：my-awesome-app
技术栈：
  - 前端：Vue 3 + TypeScript + Pinia + Vue Router + Tailwind CSS
  - 构建：Vite 5
  - 测试：Vitest + Playwright
  - 后端：Express + Prisma + PostgreSQL
  - 部署：Docker + Nginx + GitHub Actions

编码规范：
  - 使用 <script setup lang="ts">
  - Props 使用 defineProps + withDefaults
  - 状态管理用 Pinia（不用 Vuex）
  - 样式用 Tailwind（不写 CSS 文件）
  - 暗色模式必须支持

请在后续所有任务中遵守以上规范。
```

> **效果**：所有 AI Worker 在一条消息内获得完整技术栈上下文，后续工作自动遵循。

### 7.3 方法二：让 Shell 执行器先检测再广播

```
# 第一步：让 Shell 执行器收集项目信息
@Shell执行器 cat package.json | head -50

# 第二步：把信息广播给所有 AI
@All 根据 package.json，本项目的依赖如下：
  vue: 3.4.x
  pinia: 2.x
  vite: 5.x
  vitest: 2.x
  tailwindcss: 3.x
  typescript: 5.x

请按此技术栈工作。
```

### 7.4 方法三：使用需求分析师自动推断

如果你用**方案七（需求驱动）**，可以让需求分析师帮你做：

```
@需求分析师 请先分析本项目的技术栈：
1. 读取 package.json、tsconfig.json、vite.config.ts
2. 总结使用的框架、库、工具
3. 根据技术栈为团队中每个成员生成工作规范
4. 把规范广播给所有成员
```

### 7.5 方法四：保存为工作流模板

golutra 支持**工作流模板的导入导出**。你可以：

1. 把"技术栈注入 + 成员配置"整套流程保存为模板
2. 下次新项目直接导入模板
3. 只需修改技术栈描述部分

---

## 八、实战操作示例

### 8.1 场景：我有一个 Vue 3 项目，想让 AI 帮我开发新功能

**第一步**：创建团队（方案一简化版）
1. 邀请 2 个 Claude Code（编码助手）
2. 邀请 1 个 Terminal（Shell 执行器）

**第二步**：注入技术栈
```
@All 技术栈：Vue 3 + TypeScript + Pinia + Vite + Tailwind CSS
编码规范：<script setup lang="ts">，使用 Composition API，支持暗色模式。
```

**第三步**：分配任务
```
@编码助手-1 实现商品列表组件 src/components/ProductList.vue
@编码助手-2 实现购物车组件 src/components/ShoppingCart.vue
```

**第四步**：构建验证
```
@Shell执行器 pnpm run build
```

**第五步**：有问题时修复
```
@编码助手-1 构建报错 TypeScript 类型错误，请修复：[粘贴错误信息]
```

### 8.2 场景：代码写完了，想做全面审查

**第一步**：创建审查团队（方案二）
1. 邀请 1 个 Claude Code（沙盒 ✅）→ 安全审查员
2. 邀请 1 个 Gemini CLI → 架构审查员
3. 邀请 1 个 Claude Code → UI/UX 审查员
4. 邀请 1 个 Gemini CLI → 性能审查员

**第二步**：下发审查任务
```
@All 请审查以下 PR 文件：
  - src/components/DataTable.vue
  - src/composables/useVirtualScroll.ts
  - src/stores/useUserStore.ts

各自从自己的角度分析，输出格式：
  🔴 严重问题
  🟡 建议改进
  🟢 可选优化
```

**第三步**：汇总 4 份报告，做决策

### 8.3 场景：新项目从零开始

**第一步**：创建全栈团队（方案五）
- 按 [6.4 节](#64-方案五详细操作步骤全栈独立开发) 操作

**第二步**：注入需求
```
@All 项目需求：开发一个任务管理系统（类似 Trello）

技术栈：
  前端：Vue 3 + TypeScript + Tailwind
  后端：Express + Prisma + PostgreSQL
  部署：Docker

核心功能：
1. 用户注册/登录
2. 看板视图（拖拽）
3. 任务 CRUD
4. 成员协作
5. 实时通知
```

**第三步**：分阶段执行（参考方案五的 Sprint 流程）

---

## 九、常见问题 FAQ

### Q1：Worker 创建后的名字看不懂，能改吗？

**可以**。点击成员头像 → 管理成员 → 修改"显示名称"。建议改成有意义的名字，如"编码助手-1"、"安全审查员"等。

### Q2：我选错了模型/配置，怎么办？

在成员列表中，点击成员头像 → 管理成员 → **移出群组**，然后重新邀请。

### Q3：Worker 显示 Offline 怎么办？

- 检查对应的 CLI 工具是否已安装（比如 `claude` 命令是否可用）
- 检查 API Key 是否已配置
- 尝试点击成员 → 重启终端

### Q4：什么是 "规则"？文档里的审查维度、技术栈要求需要我配吗？

**不需要**。docs/workers/ 中那些"审查维度"、"技术栈"、"Prompt 模板"是**写给你参考的**，告诉你可以怎么给 AI 下指令。它们不是你在 UI 上需要配置的东西。

**你在 UI 上只需要配 4 个东西**：模型、数量、无限制、沙盒。

### Q5：不同 Worker 用同一个 CLI 工具有区别吗？

**在 golutra 层面没有区别**。比如你邀请了 3 个 Claude Code 实例，它们的配置完全一样。区别在于**你给它们发的指令不同** —— 你让第一个写代码，第二个写测试，第三个做审查，它们就变成了不同角色。

### Q6：技术栈检测什么时候能自动化？

目前版本暂不支持自动检测。但你可以用[第七章](#七技术栈如何智能化--不改代码的实现方式)的方法手动实现类似效果。作者在 README 中提到后续会开发 CEO Agent 功能，届时可能包含自动技术栈检测。

### Q7：方案七的"需求分析师"是怎么调度其他人的？

需求分析师本身不会"控制"其他 Worker 的终端。它的工作方式是：
1. 你告诉它需求
2. 它分析后给你一个**任务分配计划**
3. 你审批通过
4. 它把计划转化为**具体的 Prompt**，告诉你该发给哪个 Worker
5. 你把 Prompt 发给对应的 Worker（或用 @All 广播）

本质上是**AI 帮你思考和规划，执行仍由你操作**。

### Q8：能同时用 Claude Code 和 Gemini CLI 吗？

**可以**，这正是 golutra 的核心价值。你可以混合使用任意多种 CLI 工具。比如方案二就是 2 个 Claude + 2 个 Gemini 并行审查。

### Q9：我的项目不是前端项目，文档里的例子都是前端的，怎么办？

Worker 的"角色"是你通过 Prompt 定义的，不是固定的。同一个 Claude Code 实例，你可以让它写 Python、Go、Java 等任何语言。docs 里的例子偏前端，但工具本身**不限技术栈**。

关键是**在第一条消息中注入你的技术栈**（参考[第七章](#七技术栈如何智能化--不改代码的实现方式)），AI 就会按你的技术栈工作。

### Q10：实例数量设多少合适？

| 项目规模 | 编码类 | 审查类 | 测试类 | Shell |
|---------|--------|--------|--------|-------|
| 小项目 | 1-2 | 1 | 1 | 1 |
| 中项目 | 2-3 | 1-2 | 1-2 | 1 |
| 大项目 | 3-5 | 2-4 | 2-3 | 2 |

> **注意**：更多实例 = 更多并行 = 更多 API 费用。按需配置即可。

---

## 附录 A：全部 CLI 后端参数对照

| CLI 工具 | terminal_type | 默认命令 | 无限制模式标志 | 会话恢复 |
|---------|--------------|---------|--------------|---------|
| Claude Code | `claude` | `claude` | `--dangerously-skip-permissions` | ❌ |
| Gemini CLI | `gemini` | `gemini` | `--yolo` | ❌ |
| Codex CLI | `codex` | `codex` | `--dangerously-bypass-approvals-and-sandbox` | ✅ `resume {session_id}` |
| OpenCode | `opencode` | `opencode` | 无 | ❌ |
| Qwen Code | `qwen` | `qwen` | `--yolo` | ❌ |
| Terminal | `shell` | `bash`（系统默认） | 无 | ❌ |

## 附录 B：Worker 启动后的自动行为

当你邀请一个 AI Worker 后，golutra 会自动：

1. **启动终端**：在后台打开一个 PTY（伪终端），执行对应的 CLI 命令
2. **加无限制标志**：如果开启了无限制访问，自动追加对应的命令行参数
3. **发送 Onboarding**：自动告诉 AI 它的名字和角色
   - 中文环境：`"{名字}，这是你的名字，现在正在和团队解决问题"`
   - 英文环境：`"{name}, this is your name. You are working with the team to solve problems."`
4. **等待就绪**：检测到 CLI 工具加载完成后，状态变为 Online

你不需要手动执行这些步骤，golutra 全部自动完成。

## 附录 C：数据存储位置

| 数据 | 存储位置 |
|------|---------|
| 工作区注册表 | `~/.golutra/workspace-registry.json` |
| 项目成员配置 | `{项目}/.golutra/workspace.json` 或 `~/.golutra/{workspaceId}/project.json` |
| 聊天记录 | `~/.golutra/chat-db/`（ReDB 嵌入式数据库） |
| 头像图片 | `~/.golutra/avatars/` |

---

*本指南基于 golutra 源码分析编写，与 UI 实际行为一致。如有更新请参考官方文档。*
