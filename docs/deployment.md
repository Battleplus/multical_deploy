# 部署步骤

本文使用以下示例，请替换为真实值：

```text
Web 域名：app.example.com
API 域名：api.example.com
Mac A：控制平面 + Runtime A
Mac B：Runtime B
```

## 一、部署前准备

### Mac A

- 设置固定局域网地址或 DHCP 地址保留；
- 安装 Docker Desktop；
- 保证磁盘有足够空间保存镜像、数据库和日志；
- 关闭自动睡眠；
- 准备公网 IP + 端口转发，或准备公网隧道账户。

### Mac B

- 保证能访问 `api.example.com`；
- 安装 Git 和所需开发工具；
- 准备专用 Runtime 用户；
- 准备需要运行的 Agent CLI 登录凭证。

### DNS

创建：

```text
app.example.com
api.example.com
```

使用公网 IP 时，两条记录均指向公网入口；使用隧道时，根据隧道服务商要求创建记录。

## 二、Mac A 部署 Multica

```bash
docker info
docker compose version

git clone --depth 1 https://github.com/multica-ai/multica.git
cd multica
make selfhost
```

检查初始状态：

```bash
docker compose -f docker-compose.selfhost.yml ps
docker compose -f docker-compose.selfhost.yml logs --tail=100 backend
curl -fsS http://localhost:8080/readyz
```

预期健康检查类似：

```json
{"status":"ok","checks":{"db":"ok","migrations":"ok"}}
```

## 三、修改生产配置

编辑 Multica 项目目录中的 `.env`，至少确认：

```dotenv
APP_ENV=production
FRONTEND_ORIGIN=https://app.example.com
MULTICA_APP_URL=https://app.example.com
MULTICA_PUBLIC_URL=https://api.example.com
ALLOW_SIGNUP=false
```

保留安装程序随机生成的数据库密码与认证密钥，不要替换成示例值，也不要提交 `.env`。

应用修改：

```bash
docker compose -f docker-compose.selfhost.yml up -d
```

## 四、配置公网入口

### 方案 A：公网 IP + Caddy

1. 将路由器 TCP `80` 和 `443` 转发至 Mac A；
2. 安装 Caddy；
3. 使用本仓库的 `deploy/Caddyfile.example`；
4. 替换域名并启动 Caddy；
5. 确认防火墙仅允许必要端口。

验证：

```bash
curl -fsS https://api.example.com/readyz
curl -fsS https://app.example.com/api/config
```

### 方案 B：公网隧道

当没有公网 IP 或处于 CGNAT 后面时，可使用公网隧道。入口必须保持以下规则：

```text
app.example.com/ws/*  -> http://127.0.0.1:8080
app.example.com/*     -> http://127.0.0.1:3000
api.example.com/*     -> http://127.0.0.1:8080
```

隧道和反向代理必须支持 WebSocket。不能只把 `app.example.com` 全部指向 `3000`，否则实时功能会失败。

## 五、创建管理员和用户

1. 按 Multica 首次启动页面或容器日志提供的引导创建首位管理员；
2. 登录后创建团队 Workspace；
3. 完成邮件验证码服务配置后再邀请普通用户；
4. 保持 `ALLOW_SIGNUP=false`，避免不受控注册；
5. 不要把首次启动 Token、验证码或认证 Secret 发到群聊。

## 六、注册 Runtime A

在 Mac A 上执行：

```bash
curl -fsSL https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.sh | bash

multica setup self-host \
  --server-url https://api.example.com \
  --app-url https://app.example.com

multica daemon status
```

安装至少一个 Agent CLI，并完成登录：

```bash
codex --version
# 或
claude --version
```

## 七、注册 Runtime B

在 Mac B 上使用同一 Workspace 的授权执行：

```bash
curl -fsSL https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.sh | bash

multica setup self-host \
  --server-url https://api.example.com \
  --app-url https://app.example.com

multica daemon status
```

在 Web UI 的 Settings → Runtimes 中，将两台机器重命名为容易识别的名称，例如：

```text
mac-a-control-runtime
mac-b-worker-runtime
```

## 八、创建 Agent

建议先创建两个测试 Agent：

| Agent | Runtime | 用途 |
| --- | --- | --- |
| `codex-mac-a` | Runtime A | 验证 Mac A 本地执行 |
| `codex-mac-b` | Runtime B | 主要任务执行 |

先让每个 Agent 执行只读测试，再允许其修改仓库：

```text
输出当前目录、Git 分支和 README 标题，不修改文件。
```

## 九、上线验收

### 服务检查

```bash
curl -fsS https://api.example.com/readyz
docker compose -f docker-compose.selfhost.yml ps
```

### 功能检查

- 登录和退出正常；
- 邀请用户正常；
- `/ws` WebSocket 建立成功；
- Runtime A、Runtime B 均在线；
- 两个测试 Agent 均能运行；
- 任务日志可以实时返回；
- Runtime B 离线时，控制平面仍可访问。

### 重启检查

依次重启 Mac B、Mac A，确认：

- daemon 自动恢复或有明确的启动流程；
- Docker Compose 服务恢复；
- Caddy或公网隧道恢复；
- 数据库数据没有丢失。
