# sing-box-reality-apple

供 AI 编程助手使用的中文 Skill：在 Linux VPS 部署 **VLESS + Reality + Vision**，默认以 **www.apple.com** 为 Reality 握手目标，并按实际客户端内核版本生成配置。

这是部署工作流，不是已执行的一键脚本。它要求助手核对官方文档、保留 SSH 连接、检查配置并完成真实连接测试；不会绑定 Ubuntu 24.04 或旧教程的软件版本。具体系统是否可用，以官方支持和实际环境检测为准。

默认分流：局域网与中国大陆直连，其余流量走代理；明确指定的代理域名优先。客户端还参考 [DNS、TUN 和域名设置](references/client-profile.md)，并按实际客户端内核版本调整。该技能不会自动修改你现有的配置。

支持按设备分别适配配置，例如 iOS 内核 1.11.4、macOS 内核 1.13.13 和服务端 1.13.13 的组合；生成前仍需确认实际内核版本，不能直接把较新客户端的配置套用到旧版客户端。

## 安装与使用

将整个 `sing-box-reality-apple` 文件夹放入支持 `SKILL.md` 的助手技能目录。例如 Codex 使用 `~/.codex/skills/sing-box-reality-apple/`。其他工具按其技能目录规范安装。

调用示例：

```text
使用 $sing-box-reality-apple，帮我在已有 VPS 上部署
sing-box VLESS + Reality + Vision，Reality 目标使用 www.apple.com。
客户端是 macOS 和 iOS。先识别服务器系统、架构和服务管理方式，
再按官方支持情况安装并生成配置。国内及局域网直连，其他默认走代理。
```

也可以要求只生成配置、检查已有部署或逐步指导操作。SSH 登录信息通过本地安全方式提供，不把密码和私钥发到聊天里。

## 发布到仓库

独立仓库可把本目录内容放在仓库根目录；技能合集可放在 `skills/sing-box-reality-apple/`。运行时生成的服务器配置、客户端配置及凭据不属于本技能源码，不应上传。

## 来源与验证范围

根据[想风的原始教程](https://x.com/xaiwind/status/2067050155015340111)整理为可重复执行的工作流，修正了域名不一致及直接复制旧配置的问题。技术字段以 [sing-box 官方文档](https://sing-box.sagernet.org/)和对应版本为准。

本包的结构检查不等于实际部署验证。每次使用仍须在目标 VPS 与客户端进行连接测试。
