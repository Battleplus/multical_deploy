# 安全与运维

## Runtime 隔离

Multica daemon 和 Agent CLI 会继承启动用户的文件权限。两台 Mac 均建议创建专用标准用户，例如：

```text
multica-runner
```

该用户只应访问：

- 指定的 Git 工作目录；
- 任务所需开发工具；
- 受限的 Git 凭证；
- Agent CLI 自身凭证。

不应访问：

- 管理员个人主目录；
- 私人 SSH Key；
- 浏览器 Cookie 和密码库；
- 照片、聊天记录和云盘目录；
- 与任务无关的公司机密文件。

## 密钥规则

以下内容不得提交到 Git：

- `.env`；
- GitHub PAT；
- Agent Provider API Key；
- 数据库密码；
- OAuth Secret；
- 首次启动 Token；
- Caddy 或隧道服务的凭证文件。

如密钥曾出现在聊天、Issue、终端录屏或 Git 历史中，应立即撤销并重新生成，而不是仅删除文本。

## 网络规则

- 公网只开放 HTTPS；
- PostgreSQL 只在 Docker 内部网络使用；
- Frontend/Backend 保持绑定 `127.0.0.1`；
- Runtime 节点不开放公网入站端口；
- 路由器管理界面不得暴露到公网；
- 管理 SSH 如确有需要，应通过 VPN 并限制来源。

## 备份建议

至少保留：

- Mac A 本机每日备份；
- Mac B 或其他设备上的一份加密备份；
- 一份不与两台 Mac 同时存放的异地备份。

备份必须包含 PostgreSQL 数据和当前生产配置，但配置备份中的密钥应加密保存。

示例数据库逻辑备份思路：

```bash
docker compose -f docker-compose.selfhost.yml exec -T postgres \
  pg_dump -U postgres postgres > multica-$(date +%F).sql
```

实际数据库用户、数据库名和容器服务名应以 Multica 当前 Compose 配置为准。不要在没有验证的情况下直接把示例命令加入自动任务。

## 日常检查

### 每日

- Web 和 API 是否可访问；
- Runtime 是否在线；
- 是否有失败或长时间运行的异常任务；
- Mac A 剩余磁盘空间是否充足。

### 每周

- 检查 Docker 容器日志；
- 检查备份是否成功；
- 检查不再使用的账号和 Runtime；
- 检查 Agent CLI 登录状态。

### 每月

- 执行一次恢复演练；
- 检查 Multica 和 Agent CLI 更新；
- 审核 Workspace 成员和权限；
- 检查 TLS 证书和公网入口配置；
- 检查专用 Runtime 用户的文件权限。

## 升级流程

1. 通知用户维护时间；
2. 记录当前 Multica 镜像或 Git 版本；
3. 备份 PostgreSQL 和生产配置；
4. 阅读目标版本升级说明；
5. 拉取新版本并重新创建容器；
6. 检查数据库迁移和 `/readyz`；
7. 检查 WebSocket 和两台 Runtime；
8. 执行一次测试任务；
9. 出现异常时回退并恢复备份。

## 故障处理顺序

### Web 无法访问

1. 检查 DNS；
2. 检查 TLS 和反向代理；
3. 检查 `3000` Frontend；
4. 检查 `8080` Backend；
5. 检查 Docker 和磁盘空间。

### Runtime 离线

1. 检查节点是否休眠；
2. 检查节点能否访问 `api.example.com`；
3. 检查 daemon 状态；
4. 检查登录是否过期；
5. 检查时间、代理和防火墙设置。

### Agent 无法执行

1. 在对应 Runtime 上直接运行 Agent CLI；
2. 检查 Agent CLI 登录状态和额度；
3. 检查 Git 仓库权限；
4. 检查工作目录和开发依赖；
5. 检查任务是否绑定了正确 Runtime。
