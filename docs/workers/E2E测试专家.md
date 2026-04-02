# AI 员工档案：E2E 测试专家

> **角色代号**：E2E Test Expert  
> **推荐 CLI**：Claude Code  
> **角色类型**：`assistant`  
> **出现方案**：方案四

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | 端到端集成测试编写 |
| 推荐 CLI | Claude Code |
| 推荐实例数 | 1-2 |
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
  "avatar": "css:orbit",
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

| 测试类型 | 框架 | 覆盖范围 |
|---------|------|---------|
| 核心业务流程 | Playwright | 登录、CRUD、支付等关键路径 |
| 边界场景 | Playwright | 网络异常、权限不足、并发操作 |
| 视觉回归 | Playwright toHaveScreenshot | UI 截图对比 |
| 跨浏览器 | Playwright multi-browser | Chrome / Firefox / Safari |

---

## 四、核心 Skill

```json
[
  { "nameKey": "skill.playwright", "icon": "🎭", "ver": "1.x" },
  { "nameKey": "skill.e2e-testing", "icon": "🌐", "ver": "1.0" },
  { "nameKey": "skill.page-object-model", "icon": "📄", "ver": "1.0" },
  { "nameKey": "skill.visual-regression", "icon": "📸", "ver": "1.0" }
]
```

---

## 五、Prompt 模板库

### 5.1 核心流程 E2E

```
用 Playwright 编写 {流程名} 的 E2E 测试。

测试文件：tests/e2e/{流程名}.spec.ts
Page Object：tests/e2e/pages/{页面名}Page.ts

用户操作流程：
1. {步骤1}
2. {步骤2}
3. {步骤3}

断言：
1. {预期1}
2. {预期2}

要求：
- Page Object 模式封装选择器
- 使用 data-testid 属性定位
- waitForSelector 替代固定 sleep
- 关键步骤截图 toHaveScreenshot
- 测试后数据清理
```

### 5.2 异常场景 E2E

```
用 Playwright 编写异常场景测试：

场景列表：
1. 网络断开 → 显示离线提示
2. API 返回 500 → 显示错误页面
3. Token 过期 → 跳转登录页
4. 双击提交 → 只触发一次请求
5. 并发编辑 → 冲突提示

使用 page.route() 模拟网络异常。
```

### 5.3 视觉回归测试

```
为 {页面/组件} 编写视觉回归测试：

测试场景：
1. 默认状态截图对比
2. 暗色模式截图对比
3. 移动端截图对比（viewport: 375x667）
4. 数据为空时截图对比
5. 加载中状态截图对比

使用 toHaveScreenshot({ maxDiffPixels: 100 })
```

---

## 六、协作关系

```
上游：编码助手 → 页面完成 → E2E 专家

下游：E2E 专家 → 测试代码 → Shell 执行器（npx playwright test）
      E2E 专家 → 失败报告 → 编码助手（修复）
```

---

## 七、微调建议

| 场景 | 调整 |
|------|------|
| CI 环境 | 使用 headless 模式、设置合理超时 |
| 跨浏览器 | 配置 Playwright projects |
| 移动端 | 使用 devices 预设 |
| 高并发 | 配置 workers 数量 |

---

*关联方案：[方案四](../schemes/方案四：前端测试自动化军团.md)*
