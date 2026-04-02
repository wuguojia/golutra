# AI 员工档案总索引

> 本目录包含 golutra 中所有 AI 员工角色的详细档案。  
> 每个档案定义了该角色的配置、职责、技能、Prompt 模板和微调建议。  
> 用于后续扩充、定制和微调 AI 员工的行为。

---

## 员工分类

### 🔧 开发类

| 档案 | 角色 | 推荐 CLI | 典型方案 |
|------|------|----------|---------|
| [编码助手](worker-coding-assistant.md) | 前端/全栈编码 | Claude Code | 方案一/三/五 |
| [后端开发](worker-backend-dev.md) | 后端 API 开发 | Claude Code | 方案五 |
| [TypeScript 类型工程师](worker-type-engineer.md) | 类型系统维护 | Claude Code | 方案三 |

### 🔍 审查类

| 档案 | 角色 | 推荐 CLI | 典型方案 |
|------|------|----------|---------|
| [安全审查员](worker-security-reviewer.md) | 安全漏洞检查 | Claude Code (沙盒) | 方案二/五 |
| [架构审查员](worker-architecture-reviewer.md) | 架构设计评审 | Gemini CLI | 方案二 |
| [UI/UX 审查员](worker-uiux-reviewer.md) | 可访问性/体验审查 | Claude Code | 方案二 |
| [性能审查员](worker-performance-reviewer.md) | 性能优化分析 | Gemini CLI | 方案二 |

### 🧪 测试类

| 档案 | 角色 | 推荐 CLI | 典型方案 |
|------|------|----------|---------|
| [测试专家](worker-test-expert.md) | 单元测试编写 | Claude Code | 方案一/四 |
| [E2E 测试专家](worker-e2e-expert.md) | 端到端测试 | Claude Code | 方案四 |

### ⚙️ 基础设施类

| 档案 | 角色 | 推荐 CLI | 典型方案 |
|------|------|----------|---------|
| [Shell 执行器](worker-shell-executor.md) | 命令执行 | bash/zsh | 方案一/四/五 |
| [DevOps 工程师](worker-devops-engineer.md) | 部署/CI/CD | bash/zsh | 方案五 |
| [数据库设计师](worker-database-designer.md) | Schema 设计 | Gemini CLI | 方案五 |

### 📝 文档/研究类

| 档案 | 角色 | 推荐 CLI | 典型方案 |
|------|------|----------|---------|
| [文档工程师](worker-docs-writer.md) | 文档编写 | Gemini CLI | 方案三/五/六 |
| [技术研究员](worker-researcher.md) | 技术调研 | Claude/Gemini | 方案六 |

---

## 档案结构说明

每份 AI 员工档案包含以下标准章节：

1. **基础信息** — 角色定义、推荐 CLI、配置参数
2. **golutra 配置** — project-data.json 中的完整配置
3. **职责定义** — 具体工作内容和边界
4. **核心 Skill** — 该角色需要的技能集
5. **Prompt 模板库** — 可直接使用的指令模板
6. **协作关系** — 与其他角色的上下游关系
7. **微调建议** — 如何根据项目定制该角色
8. **常见问题** — 使用中的 FAQ

---

## 快速配置速查

### 最小团队（3 人）
```
编码助手 ×2 + Shell 执行器 ×1
```

### 标准团队（5 人）
```
编码助手 ×2 + 测试专家 ×1 + Shell 执行器 ×1 + 安全审查员 ×1
```

### 完整团队（8 人）
```
编码助手 ×3 + 测试专家 ×1 + E2E专家 ×1 + Shell执行器 ×1 + 安全审查员 ×1 + 文档工程师 ×1
```

### 一人公司（7 人）
```
前端 ×2 + 后端 ×1 + 数据库 ×1 + DevOps ×1 + 文档 ×1 + 审查 ×1
```

---

## 自定义新角色

如需创建不在列表中的角色，参考以下模板：

```json
{
  "roleType": "assistant",
  "terminalType": "claude|gemini|codex|opencode|qwen|shell",
  "terminalCommand": "{CLI 命令}",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": false
}
```

关键决策：
- **terminalType**：决定使用哪个 AI 模型（或 Shell）
- **unlimitedAccess**：主力角色开启，辅助角色可关闭
- **sandboxed**：安全敏感角色开启
- **instances**：根据并行需求设置 1-20

---

*关联文档：[前端开发者指南](../frontend-developer-guide.md) | [方案文档](../schemes/)*
