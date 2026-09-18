# Tailscale 私密协作接入

## 当前地址

直接任务地址：

<https://0000mac-mini.tail5875d4.ts.net/111/issues>

当前 Tailscale Serve 将该 HTTPS 地址代理到：

```text
http://127.0.0.1:3000
```

Serve 为 `tailnet only`，没有启用 Funnel。Multica 的 `3000`、`8080` 和 PostgreSQL 均未直接暴露到公网。

## 是否必须登录同一个 Tailscale 账号

不需要所有电脑共用同一个 Tailscale 登录账号。

Tailscale 登录账号和 Tailnet 是两个概念：

- **账号**：某个人使用的 Google、Microsoft、GitHub 或其他身份；
- **Tailnet**：一组被授权互相访问的用户和设备组成的私有网络。

另一台电脑可以使用完全不同的 Tailscale 账号，但必须满足以下条件之一：

1. 该账号已经被 Tailnet 管理员邀请加入当前 Tailnet；
2. 管理员通过 Tailscale 设备共享功能，将当前 Mac 的访问权共享给该账号；
3. Tailnet ACL 或 grants 允许该用户访问当前 Mac 的 HTTPS 服务。

如果另一台电脑登录的是一个无关的 Tailnet，并且没有收到成员邀请或设备共享权限，则无法访问该地址。这正是 `tailnet only` 的安全边界。

不建议多人共用同一个 Tailscale 账号。每位协作者应使用自己的 Tailscale 身份，便于移除成员、撤销访问和审计。

官方参考：

- [邀请外部用户加入 Tailnet](https://tailscale.com/docs/features/sharing/how-to/invite-any-user)
- [将设备共享给其他 Tailnet 用户](https://tailscale.com/docs/features/sharing)

## 两层权限

访问任务界面必须同时通过两层认证：

```text
第一层：Tailscale
确认这台电脑有权访问当前 Mac

第二层：Multica
确认这个用户属于 Workspace 111
```

只有 Tailscale 权限，没有 Multica 账号，不能进入 Workspace。

只有 Multica 账号，但不在当前 Tailnet或未获得设备共享权限，也无法连接服务。

## 协作者接入步骤

1. 协作者在自己的电脑安装官方 Tailscale；
2. 协作者使用自己的 Tailscale 账号登录；
3. 管理员邀请该账号加入当前 Tailnet，或共享当前 Mac；
4. 协作者确认 Tailscale 状态为已连接；
5. 浏览器打开 <https://0000mac-mini.tail5875d4.ts.net/111/issues>；
6. 使用协作者自己的 Multica 邮箱账号登录；
7. 管理员将该 Multica 用户加入 Workspace `111`；
8. 协作者刷新页面并确认可以查看或创建任务。

## Multica 注册策略

协作者尚未创建 Multica 账号时，推荐配置：

```dotenv
ALLOW_SIGNUP=true
ALLOWED_EMAILS=管理员邮箱,协作者邮箱
DISABLE_WORKSPACE_CREATION=true
```

注意：

- `ALLOWED_EMAILS` 应使用明确邮箱白名单；
- 不要提交包含真实邮箱的 `.env`；
- `DISABLE_WORKSPACE_CREATION=true` 防止普通用户建立不受管理员控制的新 Workspace；
- 协作者完成注册前不能关闭 `ALLOW_SIGNUP`；
- 所有协作者注册完成后，应设置 `ALLOW_SIGNUP=false` 并重新创建 Backend。

应用配置：

```bash
cd ~/.multica/server
docker compose -f docker-compose.selfhost.yml up -d backend
curl -fsS http://localhost:8080/readyz
```

## 当前测试认证限制

当前实例使用开发环境固定验证码，仅适合受控 Tailnet 测试。不要把验证码写入本仓库，也不要向 Tailnet 之外传播。

如果未来要开放公网，必须：

- 恢复 `APP_ENV=production`；
- 清空固定开发验证码；
- 配置真实 SMTP 或邮件服务；
- 配置正式 HTTPS 入口；
- 完成权限、备份和许可证检查。

## 撤销协作者权限

需要停止某位协作者访问时，应分别处理：

1. 从 Workspace `111` 移除其 Multica 成员资格；
2. 从 Tailnet 移除其 Tailscale 身份，或撤销设备共享；
3. 检查该用户是否仍有其他 Tailnet ACL 授权；
4. 确认该用户无法再打开任务地址。

只删除 Multica 用户不能阻止网络连接，只移除 Tailscale 权限也不会删除 Multica 中的历史记录。两层权限需要分别管理。
