# 客户端分流参考

以用户提供的配置为基线，按客户端实际内核版本、系统能力和用户后续要求调整。以下是生成配置的要求，不是可直接导入的完整 JSON；不要将服务器地址、UUID、Reality 密钥或用户设备路径写入公开仓库。

## 不同设备和版本

已知使用场景：iOS 用户报告 sing-box VT 1.11.4，macOS 用户报告 SFM / sing-box VT 1.13.13，服务端用户报告 sing-box 1.13.13。这些是适配参考，不是对其他设备的版本要求；应用版本与实际内核版本可能不同，生成前确认各自的实际版本。服务端版本用 `sing-box version` 确认，iOS 与 macOS 在应用信息或日志中确认内核版本；无法确认时标注待验证。

- 若 iOS 的内核确为 1.11.4，DNS 服务器使用该版本的 `address` 格式；不要直接套用 1.12 起的新 DNS 服务器格式。
- 若 macOS 的内核确为 1.13.13，旧 DNS `address` 格式虽已弃用，按官方文档在 1.13 系列仍可作为共用配置的兼容选择；旧 `block` 特殊出站已移除，使用支持的 `reject` 规则动作。具体字段以目标版本检查结果为准。
- 用户当前手机和电脑共用一份配置：优先保留这种用法，并分别在两端检查和验证。需要生成新配置时，优先选取两端支持的字段；只有存在无法解决的版本或平台差异时才分成两份。服务端和客户端可以使用不同版本，更新应用后重新核对版本和配置。
- 兼容 1.11 和 1.13 的旧 DNS 写法不适用于移除了该格式的 1.14 及以后版本；升级后重新检查，必要时迁移或分别维护配置。

## 路由顺序

1. 在版本支持时，先按需要嗅探协议，再处理 DNS 劫持与 QUIC 拒绝；基线的 QUIC 拒绝适用于所有目标，包括国内，可能影响不支持回退的应用。用户可选择关闭或缩小其范围。
2. 私有地址和局域网走 `direct`。
3. 下列明确指定的域名及其子域名走 `proxy`，放在中国规则之前，避免被中国 IP 或域名规则抢先直连：

   ```text
   chatgpt.com openai.com oaistatic.com oaiusercontent.com auth0.com
   claude.ai anthropic.com gemini.google.com google.com googleapis.com
   gstatic.com youtube.com ytimg.com gmail.com github.com facebook.com
   instagram.com tiktok.com tiktokcdn.com twitter.com x.com telegram.org
   whatsapp.com linkedin.com paypal.com stripe.com airwallex.com shopify.com
   amazon.com amazonaws.com ebay.com etsy.com giffgaff.com mercury.com
   wise.com lili.co apple.com muse.ai
   ```

   `apple.com` 代理规则与 Reality 握手目标 `www.apple.com` 是不同用途，不要把后者当作 VPS 连接地址。实际仅对能识别域名的连接生效，不能据此保证相应服务的所有连接都匹配。
4. 中国域名规则集 `geosite-cn` 或中国 IP 规则集 `geoip-cn` 匹配时走 `direct`。规则集采用兼容当前版本的官方支持格式；参考来源分别为 [sing-geosite](https://github.com/SagerNet/sing-geosite) 和 [sing-geoip](https://github.com/SagerNet/sing-geoip)。默认通过代理下载，先确认首启能下载并检查来源仍可用。
5. `route.final` 设为 `proxy`；未命中直连或明确代理规则的流量默认代理，不因代理出错自动切换直连。

域名规则是否匹配取决于客户端实际拿到的域名；IP 规则能匹配实际 IP 或已解析的结果。按内核版本确认是否需要 `resolve` 等处理，避免把规则集存在误认为所有国内连接都必然直连。分流和规则集存在误判，使用真实流量与日志核验。

## DNS 和客户端参数

- 默认国内 DNS 使用 `https://223.5.5.5/dns-query` 并经 `direct`；远程 DNS 使用 `https://1.1.1.1/dns-query` 并经 `proxy`。明确代理域名的 DNS 规则先于 `geosite-cn`，使用远程 DNS；其他命中国内域名规则使用国内 DNS，余下默认远程 DNS。不要把 Markdown 链接语法复制成 JSON 的 URL 值。
- 按兼容版本使用 `ipv4_only` DNS 策略；它不等于禁止所有 IPv6 流量。另行检查 IPv6 是否按预期路由，不为了此设置擅自全局禁用 IPv6。
- 支持 TUN 时，基线地址 `172.19.0.1/30`、MTU `1350`、`auto_route: true`、`strict_route: false`。按平台和冲突情况调整；Apple 平台的实现限制见[官方说明](https://sing-box.sagernet.org/clients/apple/features/)。这不是 VPN 断线保护，不能保证 VPN 关闭或故障时阻止直连。
- 可用时启用仅监听 `127.0.0.1` 的 mixed 入站，基线端口 `2345`；端口已占用时选择其他端口并告知用户。确保实际 TUN 或代理客户端已启用，才能按这些规则处理流量。
- VLESS 出站使用 `xtls-rprx-vision`、Reality、`www.apple.com` SNI、Chrome uTLS 指纹、`xudp`；可按支持情况启用 TCP Fast Open，出现兼容问题再关闭。SNI 是 `www.apple.com`，点号前不要添加反斜杠。
- 优先按照[官方迁移说明](https://sing-box.sagernet.org/migration/)生成内核支持的格式：旧 `block` 特殊出站在 1.13.0 移除；旧 DNS `address` 服务器格式在 1.12.0 弃用、1.14.0 移除。不要机械复制粘贴的旧字段。

## 验证

生成后用目标客户端同版本的 `sing-box check -c <配置路径>` 或该客户端的配置检查功能验证；导入前备份既有配置。启用代理后，通过客户端日志和独立出口检查，分别验证明确代理域名、国内站点、局域网、未列出的海外站点，以及 DNS 与 IPv6 路径。无法进行某项实测就标明待验证，不根据配置文本宣称已经生效。
