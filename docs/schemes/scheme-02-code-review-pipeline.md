# 方案二：前端 Code Review 监工流水线

> **方案代号**：Code Review Pipeline  
> **适用场景**：需要多角度审查代码质量的团队 / 个人  
> **推荐指数**：⭐⭐⭐⭐  
> **所需 CLI**：Claude Code + Gemini CLI  
> **AI 员工总数**：4 个专业审查员  
> **预计效率提升**：400%–600%（多维度并行审查）

---

## 一、方案概述

传统 Code Review 通常由 1-2 个人完成，视角有限且耗时。本方案用 4 个 AI 审查员从安全、架构、UI/UX、性能四个维度并行审查代码，你作为监工汇总意见做最终决策。

### 核心理念

```
你（监工）── @All 下发审查需求
  │
  ├── 🔍 安全审查员 (Claude, sandboxed)  → XSS / CSRF / 注入检查
  ├── 📐 架构审查员 (Gemini)             → 组件设计 / 状态管理评审
  ├── 🎨 UI/UX 审查员 (Claude)           → 可访问性 / 响应式 / 一致性
  └── ⚡ 性能审查员 (Gemini)             → Bundle / 渲染 / 内存分析
  │
  ▼
你 ← 汇总 4 份报告 → 决策 → 修改 → 再审
```

### 为什么用不同 AI 模型？

- **Claude**：擅长深度代码分析、安全漏洞识别、细粒度建议
- **Gemini**：擅长架构级思考、性能数据分析、最佳实践对比
- 不同模型 **交叉验证**，避免单模型盲区

---

## 二、完整成员配置

### 2.1 成员配置表

| # | 角色定位 | roleType | terminalType | terminalCommand | instances | unlimitedAccess | sandboxed | 命名示例 |
|---|---------|----------|-------------|-----------------|-----------|----------------|-----------|---------|
| 0 | 👑 监工（你） | `owner` | — | — | 1 | — | — | Owner |
| 1 | 🔍 安全审查员 | `assistant` | `claude` | `claude` | 1 | ✅ | ✅ **开启** | myapp-assistant-claude-1 |
| 2 | 📐 架构审查员 | `assistant` | `gemini` | `gemini` | 1 | ✅ | ❌ | myapp-assistant-gemini-1 |
| 3 | 🎨 UI/UX 审查员 | `assistant` | `claude` | `claude` | 1 | ✅ | ❌ | myapp-assistant-claude-2 |
| 4 | ⚡ 性能审查员 | `assistant` | `gemini` | `gemini` | 1 | ✅ | ❌ | myapp-assistant-gemini-2 |

### 2.2 关键配置差异

#### 安全审查员 —— 唯一开启沙盒的角色

```json
{
  "roleType": "assistant",
  "terminalType": "claude",
  "terminalCommand": "claude",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": true
}
```

**为什么开启沙盒**：
- 安全审查可能涉及分析恶意代码片段
- 沙盒隔离防止审查过程中意外执行危险代码
- 审查员只需要 **读取** 代码，不需要 **执行** 代码

#### 其他审查员 —— 标准配置

```json
{
  "roleType": "assistant",
  "terminalType": "claude|gemini",
  "terminalCommand": "claude|gemini",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": false
}
```

---

## 三、架构设计

### 3.1 审查流水线架构

```
┌──────────────────────────────────────────────────────┐
│                  Code Review Pipeline                 │
│                                                      │
│  ┌──────┐    @All 下发 PR 文件列表                     │
│  │ 你    │────────────────────────────┐               │
│  │ 监工  │                           ▼               │
│  └──────┘    ┌───────────────────────────────┐       │
│              │ dispatch.rs: mention_all=true  │       │
│              └──┬──────┬──────┬──────┬───────┘       │
│                 ▼      ▼      ▼      ▼               │
│              ┌────┐ ┌────┐ ┌────┐ ┌────┐             │
│              │安全 │ │架构 │ │UIUX│ │性能 │             │
│              │Claude│ │Gemini│ │Claude│ │Gemini│        │
│              │🔒沙盒│ │     │ │     │ │     │          │
│              └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘             │
│                 │      │      │      │               │
│                 ▼      ▼      ▼      ▼               │
│              ┌──────────────────────────────┐        │
│              │    4 份审查报告回流到聊天界面    │        │
│              └──────────────────────────────┘        │
│                         │                            │
│                         ▼                            │
│              ┌──────────────────────┐                │
│              │  你：汇总 → 决策 → 修改 │                │
│              └──────────────────────┘                │
└──────────────────────────────────────────────────────┘
```

### 3.2 审查维度矩阵

| 维度 | 审查员 | 关注重点 | 检查项示例 |
|------|--------|---------|-----------|
| 安全 | 🔍 安全审查员 | 漏洞、风险 | XSS、v-html、innerHTML、eval、CSRF token |
| 架构 | 📐 架构审查员 | 设计合理性 | 组件拆分、状态管理、Props drilling、依赖方向 |
| UI/UX | 🎨 UI/UX 审查员 | 用户体验 | ARIA 属性、焦点管理、响应式、对比度 |
| 性能 | ⚡ 性能审查员 | 运行效率 | 不必要渲染、computed 缓存、懒加载、Tree-shaking |

---

## 四、使用流程详解

### 4.1 标准审查流程

#### 第一轮：下发审查任务

```
你（群聊 @All）:
"以下是本次 PR #42 的改动文件，请各自从自己的角度审查：

改动文件：
  - src/components/DataTable.vue （新增）
  - src/composables/useVirtualScroll.ts （新增）
  - src/utils/sanitize.ts （修改）
  - src/stores/useUserStore.ts （修改）
  - src/pages/UserList.vue （修改）

审查分工：
  🔍 安全审查员：重点审查 sanitize.ts 的 XSS 防护逻辑，
     检查 DataTable 中是否有 v-html 或 innerHTML 使用
  📐 架构审查员：评审 useVirtualScroll 的 composable 设计，
     检查 useUserStore 的状态结构是否合理
  🎨 UI/UX 审查员：检查 DataTable 的可访问性（键盘导航、ARIA），
     检查 UserList 的响应式断点
  ⚡ 性能审查员：分析 useVirtualScroll 的虚拟化策略，
     检查 DataTable 的渲染性能"
```

#### 第二轮：审查员并行工作

4 个审查员同时收到消息，各自开始分析。

**监工操作**：
- 轮流点击头像查看各审查员的终端输出
- 观察 Working → Online 状态变化
- 如发现方向偏差，终端注入修正

#### 第三轮：汇总报告

```
你查看 4 份报告后，私聊各审查员获取补充信息：

你 → @安全审查员:
"你提到 sanitize.ts 的 DOMPurify 配置可能遗漏 SVG 攻击向量，
 请给出具体的配置建议代码"

你 → @性能审查员:
"你建议用 IntersectionObserver 替代 scroll 事件监听，
 请给出性能对比数据的估算"
```

#### 第四轮：修复后复审

```
你（群聊 @All）:
"已根据各位的审查意见完成修改。以下是改动点，请复审：

1. sanitize.ts: 添加了 SVG 过滤规则
2. useVirtualScroll.ts: 改为 IntersectionObserver 实现
3. DataTable.vue: 添加了 ARIA 角色和键盘导航
4. UserList.vue: 优化了 computed 缓存策略

请确认是否还有遗留问题。"
```

### 4.2 紧急安全审查

```
你 → 私聊 @安全审查员:
"紧急：线上报告了一个 XSS 漏洞，疑似在用户评论渲染处。
 请立即审查以下文件：
 - src/components/CommentRender.vue
 - src/utils/markdown.ts
 - src/directives/v-safe-html.ts
 
 重点：
 1. 所有用户输入的渲染路径
 2. 是否存在绕过 sanitize 的路径
 3. 给出紧急修复方案"
```

### 4.3 新人代码审查

```
你（群聊 @All）:
"新同事提交了第一个 PR，请从教学角度审查：
 - 不只是指出问题，还要解释为什么
 - 给出最佳实践的代码示例
 - 语气友好，鼓励为主
 
 文件：src/components/NewFeature.vue"
```

---

## 五、审查 Prompt 模板库

### 5.1 通用审查模板

```
请审查以下代码文件，从 {审查维度} 角度分析：

文件列表：
{文件路径列表}

输出格式：
1. 严重问题 (🔴 必须修复)
2. 建议改进 (🟡 推荐修复)
3. 最佳实践 (🟢 可选优化)

每个问题请包含：
- 文件名和行号
- 问题描述
- 风险等级
- 建议修复方案（含代码示例）
```

### 5.2 安全审查专用模板

```
请从前端安全角度审查以下代码：

重点检查：
1. XSS 风险：v-html、innerHTML、document.write、eval
2. CSRF 防护：API 调用是否携带 token
3. 敏感信息暴露：hardcoded credentials、localStorage 存储
4. 依赖风险：已知漏洞的 npm 包
5. CSP 兼容性：内联脚本/样式是否兼容 CSP
6. 原型链污染：Object.assign 等操作的安全性

文件：{文件路径}
```

### 5.3 架构审查专用模板

```
请从前端架构角度评审以下代码：

评审维度：
1. 组件设计：SRP 原则、组件粒度、Props 设计
2. 状态管理：Pinia store 结构、数据流向、副作用隔离
3. Composable 设计：复用性、入参设计、返回值类型
4. 依赖关系：模块间依赖方向、循环依赖检测
5. 可测试性：纯函数比例、依赖注入点
6. 可扩展性：对未来需求变更的适应能力

文件：{文件路径}
```

### 5.4 UI/UX 审查专用模板

```
请从 UI/UX 和可访问性角度审查以下代码：

审查维度：
1. 语义化 HTML：标签选择是否合理
2. ARIA 属性：role、aria-label、aria-describedby
3. 焦点管理：Tab 顺序、焦点陷阱、focus-visible
4. 键盘导航：所有交互是否可键盘操作
5. 响应式设计：断点设置、mobile-first、触摸适配
6. 色彩对比：WCAG AA 标准、暗色模式支持
7. 动画：prefers-reduced-motion 支持
8. 国际化：文本是否支持 RTL、长文本溢出

文件：{文件路径}
```

### 5.5 性能审查专用模板

```
请从前端性能角度分析以下代码：

分析维度：
1. 渲染性能：不必要的 re-render、v-if vs v-show 选择
2. 计算优化：computed 缓存、memo 使用
3. 数据获取：请求去重、缓存策略、分页加载
4. Bundle 大小：Tree-shaking 友好性、动态导入
5. 列表优化：虚拟滚动、key 使用
6. 内存管理：事件监听清理、大对象释放
7. 图片优化：懒加载、格式选择、尺寸适配

文件：{文件路径}
```

---

## 六、监工策略详解

### 6.1 审查质量评估

审查完成后，评估每份报告的质量：

| 评估维度 | 高质量 | 低质量 |
|---------|--------|--------|
| 具体性 | 指出具体行号和代码 | 泛泛而谈 |
| 可操作 | 给出具体修复代码 | 只说"需要改进" |
| 优先级 | 区分严重/建议/可选 | 不分轻重 |
| 上下文 | 理解业务场景 | 脱离实际 |

**质量不够时的干预**：
```
你 → @架构审查员:
"你的审查太笼统了。请针对 useVirtualScroll.ts 给出：
 1. 具体哪个函数/行有问题
 2. 为什么这是问题
 3. 建议的替代方案代码
 4. 修改后的预期收益"
```

### 6.2 冲突意见处理

当不同审查员意见冲突时：

```
场景：安全审查员建议用 CSP 禁止 inline styles
      UI/UX 审查员建议用 inline styles 实现动态主题

处理方式：
你（群聊 @安全审查员 @UI/UX审查员）:
"两位对 inline styles 的使用有不同意见：
 - 安全侧建议禁止 inline styles
 - UI 侧需要 inline styles 做动态主题
 
 请讨论折中方案，例如：
 - 使用 CSS Custom Properties 替代
 - 或使用 style nonce
 
 请各自给出对折中方案的评估。"
```

---

## 七、Skill 配置

```json
{
  "skills": {
    "current": [
      {
        "nameKey": "skill.security-review",
        "icon": "🔒",
        "color": "#ef4444",
        "bg": "rgba(239, 68, 68, 0.1)",
        "ring": "rgba(239, 68, 68, 0.3)",
        "ver": "1.0"
      },
      {
        "nameKey": "skill.code-review",
        "icon": "🔍",
        "color": "#8b5cf6",
        "bg": "rgba(139, 92, 246, 0.1)",
        "ring": "rgba(139, 92, 246, 0.3)",
        "ver": "1.0"
      }
    ]
  }
}
```

---

## 八、扩缩容建议

| 场景 | 调整方案 | 审查员总数 |
|------|---------|-----------|
| 快速审查 | 只保留安全+性能 | 2 |
| 完整审查 | 4 个全部保留 | 4 |
| 加入修复 | 增加 1 个编码助手 | 5 |
| 重大重构 | 每个维度 2 个实例 | 8 |

---

*关联文档：[安全审查员](../workers/worker-security-reviewer.md) | [架构审查员](../workers/worker-architecture-reviewer.md) | [UI/UX 审查员](../workers/worker-uiux-reviewer.md) | [性能审查员](../workers/worker-performance-reviewer.md)*
