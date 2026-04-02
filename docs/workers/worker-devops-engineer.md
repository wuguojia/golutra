# AI 员工档案：DevOps 工程师

> **角色代号**：DevOps Engineer  
> **推荐 CLI**：bash（系统终端）  
> **角色类型**：`assistant`  
> **出现方案**：方案五

---

## 一、基础信息

| 属性 | 值 |
|------|-----|
| 角色定位 | Docker、CI/CD、部署、基础设施 |
| 推荐 CLI | bash（需要直接执行系统命令） |
| 推荐实例数 | 1 |
| 沙盒模式 | ❌ 关闭 |
| 无限制模式 | ✅ 开启 |

> **注意**：DevOps 工程师使用 Shell 终端，与 Shell 执行器类似，但职责聚焦在基础设施配置。

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

---

## 三、职责定义

| 职责 | 工具/技术 |
|------|---------|
| 容器化 | Docker、docker-compose |
| CI/CD | GitHub Actions、GitLab CI |
| 反向代理 | Nginx 配置 |
| 环境管理 | .env 模板、环境变量 |
| 部署脚本 | Shell 脚本编写 |
| 健康检查 | curl、wget |

---

## 四、核心 Skill

```json
[
  { "nameKey": "skill.docker", "icon": "🐳", "ver": "24.x" },
  { "nameKey": "skill.github-actions", "icon": "🔄", "ver": "1.0" },
  { "nameKey": "skill.nginx", "icon": "🌐", "ver": "1.0" },
  { "nameKey": "skill.shell-scripting", "icon": "📟", "ver": "1.0" }
]
```

---

## 五、Prompt 模板库

### 5.1 Docker 环境搭建

```
创建 docker-compose.yml 包含：

服务列表：
- {服务1}：{镜像:版本}，端口 {端口}
- {服务2}：{镜像:版本}，端口 {端口}

要求：
- 数据持久化（volumes）
- 网络隔离（networks）
- 健康检查（healthcheck）
- .env 变量模板
- 开发/生产两套配置
```

### 5.2 CI/CD Pipeline

```
创建 GitHub Actions CI/CD Pipeline：

.github/workflows/ci.yml:
  触发：push to main, PR
  步骤：
  1. Checkout
  2. Setup Node.js
  3. Install dependencies (pnpm)
  4. Lint
  5. Test (with coverage)
  6. Build
  7. Upload coverage report

.github/workflows/deploy.yml:
  触发：tag push (v*)
  步骤：
  1. Build Docker image
  2. Push to registry
  3. Deploy to server
```

### 5.3 Nginx 配置

```
生成 Nginx 配置用于 Vue SPA 部署：

要求：
- SPA history mode 支持（try_files）
- 静态资源缓存（Cache-Control）
- Gzip 压缩
- Security headers (X-Frame-Options, CSP 等)
- API 反向代理 (/api → backend:3001)
- HTTPS（Let's Encrypt 证书）
```

### 5.4 部署验证

```
docker compose up -d && sleep 5 && \
echo "=== Services ===" && docker compose ps && \
echo "=== Health ===" && curl -s http://localhost:3001/health && \
echo "=== Frontend ===" && curl -s -o /dev/null -w "%{http_code}" http://localhost
```

---

## 六、协作关系

```
上游：你（CEO）→ 基础设施需求 → DevOps

下游：
  DevOps → 环境就绪 → 后端开发（API 部署）
  DevOps → CI/CD 就绪 → 全团队（自动化流程）
  DevOps → 部署完成 → 你（验收）
```

---

## 七、微调建议

| 场景 | 调整 |
|------|------|
| 小项目 | Docker Compose 足够 |
| 中项目 | 加入 CI/CD |
| 大项目 | Kubernetes、Terraform |
| Serverless | Vercel / Netlify 配置 |

---

*关联方案：[方案五](../schemes/scheme-05-solo-fullstack.md)*
