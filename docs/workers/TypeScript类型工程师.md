# AI 员工档案：TypeScript 类型工程师

> **角色代号**：Type Engineer  
> **推荐 CLI**：Claude Code  
> **角色类型**：`assistant`  
> **出现方案**：方案三

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | TypeScript 类型系统维护、统一导出 |
| 推荐 CLI | Claude Code（类型推理能力强） |
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

| 职责 | 说明 |
|------|------|
| 类型统一 | 检查所有组件的 Props/Events/Slots 类型一致性 |
| 导出文件 | 生成 index.ts 统一入口 |
| 泛型约束 | 确保 Select\<T\>、Table\<T\> 等泛型正确 |
| 类型守卫 | 编写 isXxx() 类型守卫函数 |
| 工具类型 | 编写项目级 utility types |

---

## 四、核心 Skill

```json
[
  { "nameKey": "skill.typescript-advanced", "icon": "🔷", "ver": "5.x" },
  { "nameKey": "skill.type-inference", "icon": "🧮", "ver": "1.0" },
  { "nameKey": "skill.generics", "icon": "📐", "ver": "1.0" },
  { "nameKey": "skill.utility-types", "icon": "🔧", "ver": "1.0" }
]
```

---

## 五、Prompt 模板库

### 5.1 类型统一

```
检查以下组件列表的 TypeScript 类型定义一致性：

组件列表：
{组件列表}

检查项：
1. Props 是否都使用 defineProps + withDefaults
2. Events 是否都使用 defineEmits 类型声明
3. 命名是否一致（{Component}Props, {Component}Emits）
4. 共享类型是否抽取到 types.ts
5. 泛型使用是否正确

输出：统一后的 src/components/types.ts 和 src/components/index.ts
```

### 5.2 工具类型

```
为项目创建通用 TypeScript 工具类型：

需要的工具类型：
1. DeepPartial<T> - 深度可选
2. DeepReadonly<T> - 深度只读
3. Nullable<T> - 可空
4. AsyncReturnType<T> - 异步函数返回值
5. PropsOf<Component> - 提取组件 Props 类型

输出到 src/types/utils.ts
```

### 5.3 API 类型生成

```
根据以下 API 接口定义，生成对应的 TypeScript 类型：

API 列表：
{OpenAPI/Swagger 或接口文档}

输出：
- src/types/api.ts (请求/响应类型)
- src/types/models.ts (业务模型类型)
```

---

## 六、协作关系

```
上游：组件工人 → 组件完成 → 类型工程师（统一类型）

下游：类型工程师 → 类型文件 → 文档工程师（生成 API 文档）
```

---

## 七、微调建议

| 场景 | 调整 |
|------|------|
| 组件库 | 强调导出类型的公共 API |
| 全栈项目 | 前后端共享类型（tRPC 风格） |
| Monorepo | 跨包类型引用、Package exports |

---

*关联方案：[方案三](../schemes/方案三：前端组件工厂.md)*
