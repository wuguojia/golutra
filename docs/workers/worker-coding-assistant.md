# AI 员工档案：编码助手

> **角色代号**：Coding Assistant  
> **推荐 CLI**：Claude Code  
> **角色类型**：`assistant`  
> **使用频率**：⭐⭐⭐⭐⭐（最高频角色）  
> **出现方案**：方案一、三、五

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | 前端/全栈代码编写 |
| 推荐 CLI | Claude Code（代码质量最高） |
| 替代 CLI | Codex、Gemini、Qwen |
| 推荐实例数 | 2-5 个（按任务并行度） |
| 沙盒模式 | ❌ 关闭（需要读写文件） |
| 无限制模式 | ✅ 开启（主力角色，高频使用） |
| 自动启动 | ✅ 开启 |

---

## 二、golutra 配置

### 2.1 邀请参数

```
终端类型：Claude Code
实例数量：2（标准）/ 5（组件工厂模式）
无限制模式：开启
沙盒模式：关闭
```

### 2.2 project-data.json 配置

```json
{
  "id": "{ULID}",
  "name": "{workspace}-assistant-claude-{N}",
  "roleKey": "members.roles.aiAssistant",
  "roleType": "assistant",
  "avatar": "css:ember",
  "status": "online",
  "terminalType": "claude",
  "terminalCommand": "claude",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": false
}
```

---

## 三、职责定义

### 3.1 核心职责

| 职责 | 说明 | 优先级 |
|------|------|--------|
| 组件开发 | Vue/React SFC 组件编写 | 🔴 高 |
| 页面开发 | 完整页面实现 | 🔴 高 |
| 功能实现 | 业务逻辑编码 | 🔴 高 |
| Bug 修复 | 定位和修复缺陷 | 🔴 高 |
| 代码重构 | 优化代码结构 | 🟡 中 |
| 样式编写 | CSS/Tailwind 样式 | 🟡 中 |

### 3.2 职责边界

**✅ 该做的**：
- 编写符合规范的代码
- 生成 TypeScript 类型
- 添加必要的注释
- 输出简要的改动说明

**❌ 不该做的**：
- 执行构建/测试命令（交给 Shell 执行器）
- 编写正式文档（交给文档工程师）
- 做技术选型决策（交给研究员）
- 安全审查（交给安全审查员）

---

## 四、核心 Skill

### 4.1 技术栈 Skill

```json
[
  { "nameKey": "skill.vue3", "icon": "🟢", "ver": "3.5+" },
  { "nameKey": "skill.typescript", "icon": "🔷", "ver": "5.x" },
  { "nameKey": "skill.tailwindcss", "icon": "🎨", "ver": "3.x" },
  { "nameKey": "skill.composition-api", "icon": "⚡", "ver": "1.0" },
  { "nameKey": "skill.pinia", "icon": "🍍", "ver": "3.x" },
  { "nameKey": "skill.vite", "icon": "⚡", "ver": "6.x" }
]
```

### 4.2 编码规范 Skill

| 规范 | 要求 |
|------|------|
| Script 语法 | `<script setup lang="ts">` |
| Props 定义 | `defineProps` + `withDefaults` |
| Events 定义 | `defineEmits` 类型声明 |
| 响应式 | `ref`, `computed`, `watch` |
| 命名 | composable: `use*`，组件: PascalCase |
| 样式 | Tailwind 优先，`dark:` 暗色支持 |
| a11y | 语义化标签 + ARIA 属性 |

---

## 五、Prompt 模板库

### 5.1 组件开发

```
用 Vue 3 Composition API + TypeScript + Tailwind CSS 实现 {组件名} 组件。

技术要求：
- <script setup lang="ts">
- Props: defineProps + withDefaults
- Events: defineEmits 类型声明
- 支持暗色模式 (dark: prefix)
- 响应式 (mobile-first)
- 键盘可操作 (a11y)

Props 定义：
{Props 列表}

Events 定义：
{Events 列表}

功能需求：
{功能列表}

输出文件：
- {组件名}.vue
- types.ts (导出类型)
```

### 5.2 页面开发

```
用 Vue 3 + TypeScript 实现 {页面名} 页面。

路由：{路由路径}
布局：{布局要求}

功能需求：
{功能列表}

API 对接：
{API 列表}

状态管理：
- 使用 Pinia store: {store 名}
- 数据流：{描述}

要求：
- 加载状态处理 (skeleton)
- 错误状态处理 (error boundary)
- 空状态处理 (empty state)
```

### 5.3 Bug 修复

```
修复以下 Bug：

描述：{描述}
文件：{文件路径}
复现步骤：{步骤}
期望行为：{期望}
实际行为：{实际}

要求：
- 分析根因
- 最小改动修复
- 不引入新的 TS 错误
- 补充回归测试
- 输出修复说明
```

### 5.4 代码重构

```
重构 {文件/目录}：

重构目标：
{目标描述}

约束条件：
- 不改变外部 API（Props/Events/Expose）
- 不引入新依赖
- 所有现有测试继续通过

重构方向：
{方向描述}

输出：
- 重构后的代码
- 改动说明
- 前后对比
```

---

## 六、协作关系

```
上游：
  你（监工）→ 下发需求 → 编码助手

下游：
  编码助手 → 代码完成 → 测试专家（写测试）
  编码助手 → 代码完成 → 安全审查员（审查安全）
  编码助手 → 代码完成 → Shell 执行器（构建验证）
  编码助手 → 代码完成 → 文档工程师（生成文档）

平行：
  编码助手-1 ←→ 编码助手-2（不同任务，互不依赖）
```

---

## 七、微调建议

### 7.1 按框架微调

| 框架 | 微调点 |
|------|--------|
| Vue 3 | 强调 Composition API、SFC、Pinia |
| React | 强调 Hooks、JSX、Zustand/Redux |
| Next.js | 强调 SSR/SSG、App Router、Server Components |
| 小程序 | 强调平台 API、组件约束 |

### 7.2 按项目规模微调

| 规模 | 微调点 |
|------|--------|
| 小项目 | 简洁为主，减少抽象层 |
| 中项目 | 引入 composable 复用，模块化 |
| 大项目 | 强调架构分层、依赖注入、测试覆盖 |

### 7.3 按团队规范微调

在首条消息中注入项目规范：

```
你 → @编码助手-1:
"在开始工作前，请遵守以下项目编码规范：
 1. 组件命名使用 PascalCase
 2. composable 命名使用 use + PascalCase
 3. 所有 API 调用封装在 src/api/ 目录
 4. 公共类型定义在 src/types/ 目录
 5. 使用 @/ 作为 src 的路径别名
 6. CSS 使用 Tailwind，不写自定义 CSS
 
 后续所有任务请遵守以上规范。"
```

---

## 八、常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 输出使用 Options API | 未明确要求 Composition API | Prompt 中明确 `<script setup>` |
| 缺少 TypeScript 类型 | 未强调类型要求 | Prompt 中加 "TypeScript 严格模式" |
| CSS 写法不一致 | 未统一 CSS 方案 | Prompt 中指定 "仅使用 Tailwind" |
| 代码量过大 | 一次下发太多需求 | 拆分为小任务分批下发 |
| 方向偏离 | 理解有误 | 终端注入修正指令 |

---

*关联方案：[方案一](../schemes/scheme-01-personal-fullstack.md) | [方案三](../schemes/scheme-03-component-factory.md) | [方案五](../schemes/scheme-05-solo-fullstack.md)*
