# 另一台电脑接入 AI 执行话术

将下面内容发送给另一台电脑上的 AI。邮箱、密码、验证码和 Tailscale 登录必须由用户本人处理。

```text
请帮助这台电脑接入一个已部署好的 Multica 私密协作实例。

直接任务地址：
https://0000mac-mini.tail5875d4.ts.net/111/issues

服务特点：

- 仅通过 Tailscale 私有网络访问；
- 未启用 Tailscale Funnel；
- 不是公网服务；
- Multica Workspace 名称和 slug 均为 111；
- Agent 在服务端 Mac 上执行，不在当前协作者电脑执行；
- 当前任务并发上限为 1。

执行规则：

1. 先检查，不要直接安装或修改系统；
2. 安装 Tailscale 前获得用户确认；
3. Tailscale 登录由用户本人完成；
4. Multica 邮箱登录由用户本人完成；
5. 不询问、读取或输出密码、验证码、Token、Cookie 和 API Key；
6. 不启用 Funnel、Exit Node、子网路由或 Tailscale SSH；
7. 不修改当前电脑的防火墙、代理或 DNS，除非出现明确阻塞并获得确认；
8. 不需要安装 Multica daemon、Docker、Codex 或 Claude；
9. 当前电脑只是浏览器协作者，不是 Runtime 节点。

第一步：检查 Tailscale

执行：

command -v tailscale
tailscale version
tailscale status

如果没有安装 Tailscale，说明情况并请求用户确认安装官方版本。

第二步：确认 Tailscale 身份

让用户使用自己的 Tailscale 账号登录。

不要求和服务端管理员使用同一个登录账号，但该账号必须：

- 被邀请加入服务端所在的 Tailnet；或
- 获得服务端 Mac 的设备共享权限；并且
- 被 ACL/grants 允许访问该设备的 HTTPS 服务。

如果用户登录的是另一个无关 Tailnet，不要尝试绕过；报告需要管理员邀请或共享设备。

第三步：检查网络访问

确认 Tailscale 已连接后执行：

curl -I https://0000mac-mini.tail5875d4.ts.net

预期 HTTPS 返回正常状态。

如果 DNS 无法解析或连接超时，检查：

- Tailscale 是否 Connected；
- 当前账号是否加入正确 Tailnet；
- 服务端设备是否在线；
- 管理员是否共享设备；
- Tailnet ACL/grants 是否允许 HTTPS。

不要改用公网 IP，不要关闭 TLS 校验。

第四步：打开任务界面

在浏览器打开：

https://0000mac-mini.tail5875d4.ts.net/111/issues

如果跳转到登录页，让用户本人使用自己的 Multica 邮箱账号登录。

如果提示不能注册，需要服务端管理员将该邮箱加入 ALLOWED_EMAILS，并临时保持 ALLOW_SIGNUP=true。

如果登录成功但看不到 Workspace 111，需要服务端管理员邀请该 Multica 用户加入 Workspace 111。

第五步：协作验收

确认：

- 可以打开任务地址；
- 可以登录自己的 Multica 账号；
- 可以看到 Workspace 111；
- 可以查看现有 Issue；
- 权限允许时可以创建一个只读测试 Issue；
- 可以看到 Agent 状态和任务日志；
- 不要求在当前电脑安装 Agent CLI。

测试任务建议：

请报告测试仓库的当前 Git 分支和 README 标题，不要修改文件，不要提交，不要推送。

最终报告只包含：

- Tailscale 是否连接；
- 是否属于正确 Tailnet或已获得设备共享；
- HTTPS 地址是否可访问；
- Multica 是否登录；
- Workspace 111 是否可见；
- 测试 Issue 是否成功。

不要在报告中输出邮箱全称、验证码、Token、Cookie、Tailnet 成员列表或设备密钥。
```
