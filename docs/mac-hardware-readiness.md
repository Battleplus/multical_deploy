# 当前 Mac 配置与部署可行性评估

评估日期：2026 年 9 月 16 日。

本文记录当前 Mac 对 Multica 官方单机自托管方案的支持情况。文档不包含设备序列号、Hardware UUID、Provisioning UDID 等隐私信息。

## 检查结果

| 项目 | 当前配置 | 结论 |
| --- | --- | --- |
| 机型 | Mac mini | 适合长期固定部署 |
| 芯片 | Apple M4，10 核 CPU | 满足控制平面和少量 Agent 任务需求 |
| 内存 | 16 GB | 可运行单机方案，建议限制并发 |
| 磁盘 | 约 228 GB，总可用约 151 GB | 可以开始部署，需要持续监控 |
| 系统 | macOS 15.6 | 满足当前部署基础要求 |
| Docker | 尚未安装 | 部署前必须安装 Docker Desktop |
| 自动睡眠 | 已开启 | 正式提供服务前需要关闭 |
| 断电自动启动 | 未开启 | 建议正式上线前开启 |

## 总体结论

当前 M4 Mac mini 可以运行 Multica 官方单机自托管结构：

```text
Mac mini
├── Multica Frontend
├── Multica Backend
├── PostgreSQL
├── Multica daemon
├── Codex / Claude CLI
└── 项目工作目录
```

适合以下使用规模：

- 少量受邀用户；
- 同时运行 1～2 个普通 Agent 任务；
- 中小型代码仓库；
- 初期验证、内部使用或受控开放测试。

当前主要限制不是 M4 处理器，而是 16 GB 内存和约 151 GB 可用磁盘。不建议一开始就开放大量用户或启用高并发任务。

## 建议并发

建议初始配置为：

```text
同时执行 1 个 Agent 任务
```

稳定运行并观察内存后，可以尝试：

```text
同时执行 2 个 Agent 任务
```

如果任务包含大型 Docker 构建、前端构建、模型工具、多服务开发环境或大型测试，应继续保持单任务并发。

## 部署前必须处理

### 1. 安装 Docker Desktop

当前系统尚未安装 `docker`。安装 Apple Silicon 版本的 Docker Desktop 后，检查：

```bash
docker --version
docker compose version
docker info
```

Docker Desktop 初始资源建议：

```text
CPU：4～6 核
内存：6～8 GB
磁盘上限：60～100 GB
```

不要将全部 16 GB 内存分配给 Docker。macOS、Multica daemon 和 Codex/Claude CLI 同样需要宿主机内存。

### 2. 关闭接通电源时的自动睡眠

当前系统的接通电源睡眠设置为开启。无人操作时进入睡眠会造成：

- 外部用户无法访问；
- daemon 与 Backend 断开；
- 正在运行的 Agent 任务可能中断；
- WebSocket 实时连接断开。

建议执行：

```bash
sudo pmset -c sleep 0
sudo pmset -c disksleep 0
```

显示器仍然可以自动关闭：

```bash
sudo pmset -c displaysleep 10
```

应用后检查：

```bash
pmset -g custom
```

### 3. 开启断电恢复后自动启动

当前 `autorestart` 未开启，建议执行：

```bash
sudo pmset -a autorestart 1
```

该设置只负责恢复供电后重新启动 Mac。Docker Desktop、Multica daemon 和公网入口还需要各自配置为登录或开机后自动启动。

### 4. 保留足够磁盘空间

Docker 镜像、PostgreSQL、Git 仓库、依赖目录和任务日志都会持续使用磁盘空间。建议设置以下警戒线：

| 可用空间 | 建议操作 |
| --- | --- |
| 50 GB 以上 | 正常运行并定期检查 |
| 30～50 GB | 清理不用的项目缓存和确认 Docker 占用 |
| 30 GB 以下 | 暂停新任务并立即处理磁盘空间 |

检查 Docker 占用：

```bash
docker system df
docker image ls
docker volume ls
```

不要未经确认就定时执行：

```bash
docker system prune -a
```

该命令可能删除后续仍需要的镜像和构建缓存。

## 正式开放前的额外要求

硬件满足要求不代表已经具备正式公网服务条件。向外部用户开放前还需要完成：

- 稳定的 HTTPS 公网入口；
- 用户注册、邀请和权限策略；
- 真实邮件验证码服务；
- PostgreSQL 定期备份和恢复测试；
- Multica daemon 专用 macOS 标准用户；
- Agent 工作目录和个人文件隔离；
- 任务并发、超时和磁盘容量限制；
- 日志检查和异常任务处理流程；
- 确认 Multica 第三方托管的许可证要求。

## 建议实施顺序

1. 安装并启动 Docker Desktop；
2. 按官方方式部署 Multica；
3. 在本机完成管理员初始化；
4. 安装并登录一个 Agent CLI；
5. 完成一次端到端测试任务；
6. 关闭自动睡眠并设置断电自动启动；
7. 配置备份；
8. 配置 HTTPS 公网入口；
9. 先以邀请制和单任务并发开放；
10. 根据实际资源占用决定是否增加并发或第二台 Runtime。

## 最终判断

当前机器硬件合格，可以先使用一台 Mac 完成 Multica 官方原版部署。当前阻塞项是 Docker Desktop 尚未安装，正式上线前还需要调整电源设置并完成公网、安全和备份配置。
