# AI 员工档案：性能审查员

> **角色代号**：Performance Reviewer  
> **推荐 CLI**：Gemini CLI  
> **角色类型**：`assistant`  
> **出现方案**：方案二

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | 前端性能分析与优化建议 |
| 推荐 CLI | Gemini CLI（数据分析能力强） |
| 替代 CLI | Claude Code |
| 推荐实例数 | 1 |
| 沙盒模式 | ❌ 关闭 |
| 无限制模式 | ✅ 开启 |

---

## 二、golutra 配置

```json
{
  "id": "{ULID}",
  "name": "{workspace}-assistant-gemini-{N}",
  "roleKey": "members.roles.aiAssistant",
  "roleType": "assistant",
  "avatar": "css:ember",
  "status": "online",
  "terminalType": "gemini",
  "terminalCommand": "gemini",
  "autoStartTerminal": true,
  "unlimitedAccess": true,
  "sandboxed": false
}
```

---

## 三、职责定义

| 审查维度 | 检查项 |
|---------|--------|
| 渲染性能 | 不必要的 re-render、v-if vs v-show、key 使用 |
| 计算优化 | computed 缓存、ref vs reactive 选择 |
| 数据获取 | 请求去重、缓存策略、分页/无限滚动 |
| Bundle 优化 | Tree-shaking、动态导入、代码分割 |
| 列表优化 | 虚拟滚动、key 唯一性 |
| 内存管理 | 事件监听清理、大对象释放、闭包泄露 |
| 图片优化 | 懒加载、格式选择、尺寸适配、CDN |

---

## 四、核心 Skill

```json
[
  { "nameKey": "skill.performance-audit", "icon": "⚡", "ver": "1.0" },
  { "nameKey": "skill.bundle-analysis", "icon": "📦", "ver": "1.0" },
  { "nameKey": "skill.rendering-optimization", "icon": "🖥️", "ver": "1.0" },
  { "nameKey": "skill.lighthouse", "icon": "🔦", "ver": "1.0" }
]
```

---

## 五、Prompt 模板库

### 5.1 全面性能审查

```
请从前端性能角度分析以下代码：

文件：{文件路径}

分析维度：
1. 渲染性能：re-render 频率、v-if/v-show 选择
2. 计算优化：computed 使用、memo 策略
3. 数据获取：请求去重、缓存、预取
4. Bundle：动态导入、Tree-shaking 友好性
5. 列表：虚拟滚动、key 使用
6. 内存：事件监听清理、大对象
7. 图片：懒加载、格式、尺寸

输出格式：
- 问题描述
- 影响程度（高/中/低）
- 优化方案（含代码）
- 预期性能提升
```

### 5.2 Bundle 分析

```
分析项目的 Bundle 大小优化空间：

检查：
1. 哪些依赖占用最大
2. 是否有重复打包
3. 动态导入是否合理
4. CSS 是否有未使用的样式
5. 图片资源是否优化
```

### 5.3 列表性能优化

```
分析 {组件名} 的列表渲染性能：

检查：
1. 是否使用虚拟滚动（数据量 > 100 条时）
2. key 是否使用唯一标识（非 index）
3. 列表项是否为独立组件
4. 是否有不必要的深度监听
5. 排序/过滤是否在 computed 中缓存
```

---

## 六、微调建议

| 场景 | 关注重点 |
|------|---------|
| SSR 项目 | Hydration 性能、首屏渲染 |
| SPA | 路由懒加载、预取策略 |
| 数据密集型 | 虚拟滚动、Web Worker |
| 动画密集型 | requestAnimationFrame、GPU 加速 |

---

*关联方案：[方案二](../schemes/scheme-02-code-review-pipeline.md)*
