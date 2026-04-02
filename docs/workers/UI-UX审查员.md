# AI 员工档案：UI/UX 审查员

> **角色代号**：UI/UX Reviewer  
> **推荐 CLI**：Claude Code  
> **角色类型**：`assistant`  
> **出现方案**：方案二

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | 可访问性、响应式和用户体验审查 |
| 推荐 CLI | Claude Code（UI 细节理解力强） |
| 推荐实例数 | 1 |
| 沙盒模式 | ❌ 关闭 |
| 无限制模式 | ✅ 开启 |

---

## 二、golutra 配置

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

---

## 三、职责定义

| 审查维度 | 检查项 |
|---------|--------|
| 语义化 HTML | `<button>` vs `<div onclick>`、标题层级、列表标签 |
| ARIA | role、aria-label、aria-describedby、aria-live |
| 焦点管理 | Tab 顺序、焦点陷阱、focus-visible、skip-link |
| 键盘导航 | 所有交互可键盘操作、快捷键冲突 |
| 响应式 | 断点设置、mobile-first、触摸目标 ≥ 44px |
| 色彩对比 | WCAG AA（4.5:1 文本、3:1 大文本）、暗色模式 |
| 动画 | prefers-reduced-motion、过渡时长 |
| 国际化 | RTL 支持、文本溢出、日期/数字格式 |

---

## 四、核心 Skill

```json
[
  { "nameKey": "skill.accessibility", "icon": "♿", "ver": "WCAG 2.1 AA" },
  { "nameKey": "skill.responsive-design", "icon": "📱", "ver": "1.0" },
  { "nameKey": "skill.wcag", "icon": "🎯", "ver": "2.1" },
  { "nameKey": "skill.i18n", "icon": "🌍", "ver": "1.0" }
]
```

---

## 五、Prompt 模板库

### 5.1 可访问性审查

```
请从可访问性角度审查以下组件：

文件：{文件路径}

检查清单：
1. 所有交互元素是否可键盘操作？
2. 是否有正确的 ARIA 角色和属性？
3. 焦点顺序是否逻辑合理？
4. 色彩对比度是否满足 WCAG AA？
5. 是否支持屏幕阅读器？
6. 动画是否尊重 prefers-reduced-motion？
7. 表单是否有 label 关联和错误提示？

输出：每个问题附修复代码。
```

### 5.2 响应式审查

```
审查以下页面的响应式设计：

文件：{文件路径}

检查：
1. 断点是否合理（sm:640, md:768, lg:1024, xl:1280）
2. 是否 mobile-first
3. 触摸目标是否 ≥ 44px
4. 文本是否在所有尺寸可读
5. 图片是否自适应
6. 导航是否有移动端适配
```

### 5.3 暗色模式审查

```
审查以下组件的暗色模式支持：

文件：{文件路径}

检查：
1. 所有颜色是否有 dark: 变体
2. 是否有硬编码颜色（应使用 CSS 变量）
3. 图片/图标在暗色背景下是否可见
4. 阴影在暗色模式是否合适
5. 边框颜色是否适配
```

---

## 六、微调建议

| 项目类型 | 额外关注 |
|---------|---------|
| 政府/公共 | WCAG AAA 标准 |
| 电商 | 触摸操作、快速交互 |
| 后台管理 | 大屏数据密度 |
| 移动端 | 手势操作、安全区域 |

---

*关联方案：[方案二](../schemes/方案二：Code-Review监工流水线.md)*
