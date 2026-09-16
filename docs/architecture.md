# 双 Mac 架构说明

## 设计目标

该方案优先满足小型团队部署中的可维护性和安全性，而不是追求复杂的高可用：

- 控制平面集中部署，降低数据库和服务间网络故障；
- 执行节点横向扩展，每增加一台机器只需安装 daemon；
- 公网仅开放 HTTPS；
- Agent 工作负载与个人数据隔离；
- 控制平面故障时能够通过备份恢复。

## 为什么不拆成“前端 Mac”和“后端 Mac”

将 Frontend 放在一台 Mac、Backend/PostgreSQL 放在另一台 Mac，收益较小，但会增加以下问题：

- 数据库请求和内部 API 都依赖局域网；
- 任一 Mac 或局域网故障都会使整套服务不可用；
- Compose、升级、日志检查和备份流程更复杂；
- 仍然没有解决 PostgreSQL 单点问题；
- 空闲的前端主机不能有效承担 Agent 任务。

因此推荐将 Frontend、Backend 和 PostgreSQL 放在 Mac A，将第二台 Mac 用作独立 Runtime。若 Mac A 资源充足，也可同时运行 Runtime A。

## 服务边界

### 控制平面

控制平面负责：

- 用户登录、Workspace 和权限；
- Issue、聊天、通知和任务状态；
- Agent 与 Runtime 配置；
- daemon 连接和事件转发；
- PostgreSQL 持久化。

### Runtime

Runtime 负责：

- 执行 Codex、Claude Code 等本地 CLI；
- 访问节点本地 Git 仓库；
- 保存任务执行产生的本地文件；
- 将输出和状态通过安全连接返回 Backend。

Runtime 不应直接接受公网入站连接。正常情况下由 daemon 主动连接 `api.example.com`。

## 网络流向

| 来源 | 目标 | 协议 | 说明 |
| --- | --- | --- | --- |
| 用户浏览器 | `app.example.com` | HTTPS | Web UI |
| 用户浏览器 | `app.example.com/ws` | WSS | 实时聊天、通知和状态 |
| Multica CLI/daemon | `api.example.com` | HTTPS/WSS | 登录、Runtime 注册和任务通信 |
| Backend | PostgreSQL | Docker 内部网络 | 数据持久化 |
| Runtime | Agent Provider | HTTPS | 模型 API，由对应 CLI 发起 |
| Runtime | Git Provider | SSH/HTTPS | 拉取和推送仓库代码 |

## 故障边界

### Mac B 故障

- Mac A 和 Web UI 继续工作；
- Runtime B 离线；
- 绑定 Runtime B 的 Agent 暂时无法执行；
- 可以将任务切换到 Runtime A。

### Mac A 故障

- Web、API、数据库和调度全部停止；
- Mac B 上已经启动的本地进程可能继续运行，但无法可靠同步状态；
- 需要恢复 Mac A，或在备用主机恢复数据库和 Compose 服务。

### 公网入口故障

- 局域网内服务可能仍正常；
- 外部用户和 Runtime 无法连接；
- 应分别检查 DNS、TLS、路由器端口转发或公网隧道状态。

## 扩展路线

### 增加计算能力

继续增加 Runtime 节点即可，不需要迁移 PostgreSQL：

```text
Mac C / Linux Server / Developer Laptop
        └── multica daemon + Agent CLI
```

### 增加可靠性

当使用规模扩大后，可以按以下顺序升级：

1. 将控制平面迁移到稳定的 Linux 服务器或云主机；
2. 使用外部托管 PostgreSQL；
3. 增加自动备份和异地备份；
4. 增加服务监控和告警；
5. 最后再考虑多实例 Backend 和高可用数据库。
