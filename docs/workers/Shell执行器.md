# AI 员工档案：Shell 执行器

> **角色代号**：Shell Executor  
> **推荐 CLI**：bash / zsh（系统终端）  
> **角色类型**：`assistant`  
> **出现方案**：方案一、四、五

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | 系统命令执行（构建/测试/部署） |
| 推荐 CLI | bash（不连接 AI 模型） |
| 推荐实例数 | 1-2 |
| 沙盒模式 | ❌ 关闭（需要执行系统命令） |
| 无限制模式 | ✅ 开启 |

> **与其他角色的核心区别**：Shell 执行器不使用 AI 模型，它是一个纯粹的命令执行器。你发什么命令，它就执行什么。

---

## 二、golutra 配置

```json
{
  "id": "{ULID}",
  "name": "{workspace}-assistant-terminal-{N}",
  "roleKey": "members.roles.aiAssistant",
  "roleType": "assistant",
  "avatar": "css:storm",
  "status": "online",
  "terminalType": "shell",
  "terminalCommand": "bash",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": false
}
```

> **terminalType = shell**：直接打开系统 Shell，不经过 AI 模型。

---

## 三、职责定义

| 任务类型 | 命令示例 |
|---------|---------|
| 依赖安装 | `pnpm install` |
| 代码构建 | `pnpm build` |
| Lint 检查 | `pnpm lint` |
| 格式化 | `pnpm format:check` |
| 单元测试 | `pnpm test -- --coverage` |
| E2E 测试 | `npx playwright test` |
| Git 操作 | `git status && git diff` |
| 环境检查 | `node -v && pnpm -v` |
| Docker | `docker compose up -d` |

---

## 四、核心 Skill

```json
[
  { "nameKey": "skill.shell-commands", "icon": "📟", "ver": "1.0" },
  { "nameKey": "skill.build-tools", "icon": "🔨", "ver": "1.0" },
  { "nameKey": "skill.npm-scripts", "icon": "📦", "ver": "1.0" }
]
```

---

## 五、Prompt 模板库

### 5.1 构建验证

```
cd {项目路径} && pnpm install && pnpm lint && pnpm test && pnpm build
```

### 5.2 覆盖率检查

```
cd {项目路径} && pnpm test -- --coverage --reporter=verbose
```

### 5.3 E2E 执行

```
cd {项目路径} && npx playwright install && npx playwright test --reporter=list
```

### 5.4 环境诊断

```
echo "=== Node ===" && node -v && echo "=== pnpm ===" && pnpm -v && echo "=== Git ===" && git --version && echo "=== Docker ===" && docker --version 2>/dev/null || echo "Docker not found"
```

### 5.5 Git 状态检查

```
cd {项目路径} && git status && echo "---" && git diff --stat
```

---

## 六、协作关系

```
上游：
  编码助手 → 代码完成 → Shell 执行器（构建验证）
  测试专家 → 测试编写完成 → Shell 执行器（运行测试）

下游：
  Shell 执行器 → 执行结果 → 你（监工查看）
  Shell 执行器 → 失败日志 → 编码助手/测试专家（修复）
```

---

## 七、微调建议

| 场景 | 调整 |
|------|------|
| 多项目 | 每个项目一个 Shell 执行器 |
| CI 模拟 | 链式命令 `&&` 模拟 CI Pipeline |
| 长命令 | 写成 npm script 后执行 |
| 需要交互 | 提前准备好 yes/no 输入 |

---

## 八、常见问题

| 问题 | 解决 |
|------|------|
| 命令卡住等待输入 | 在终端注入 `y` 或 `yes` |
| PATH 找不到命令 | 使用完整路径或先 `which {命令}` |
| 权限不足 | 检查文件权限，不要用 sudo |
| 目录错误 | 命令开头加 `cd {路径} &&` |

---

*关联方案：[方案一](../schemes/方案一：个人前端全栈工作流.md) | [方案四](../schemes/方案四：前端测试自动化军团.md) | [方案五](../schemes/方案五：全栈独立开发者.md)*
