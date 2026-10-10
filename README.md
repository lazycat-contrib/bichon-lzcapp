# bichon-lzcapp

[Bichon](https://github.com/rustmailer/bichon)（Rust 邮箱归档器 + WebUI）的懒猫微服打包：**只发喵喵商店，镜像模式**（`delivery.mode: mirror`）。

- **镜像**：`rustmailer/bichon:2.1.1`（与 `latest` 同一份 digest；容器内监听 **15630**，数据在 `/data`）。镜像模式只做只读校验，交付时把 manifest 的镜像行改写成 `docker.1ms.run/rustmailer/bichon:2.1.1`，不搬镜像到懒猫镜像源。
- **版本跟踪**：`channel: stable` + `sort: semver` + `tag_regex: '^[0-9]+\.[0-9]+\.[0-9]+$'`（排除 `2.0.1-alpha.1` 这类预发布与 `latest`）。
  说明：上游 compose 用的是 `latest`，而 `latest` 当前就等于 `2.1.1`（digest 相同），所以直接跟正式版更干净。
- **运行用户**：镜像默认就是 root，平台建的 `/lzcapp/var/bichon` 属主也是 root，所以直接 **`user: root`**（不用 `run_as`，也不用入口脚本 chown）。上游 compose 里的 `user: "1000:1000"` 在有属主映射的平台特性时才成立，本包不采用。
- **`BICHON_ENCRYPT_PASSWORD`**：加密已存邮箱凭据（IMAP 密码、OAuth token）的静态密钥——用户本来就不需要知道它，所以用 `stable_secret` 自动生成并固定（同 JWT_SECRET 的思路）。**注意清空 `/lzcapp/var/bichon` 会让已存凭据解不开**。
- **路由**：`/` → `bichon:15630`，`public_path: [/]`（入口放行，应用自带 REST/WebUI 鉴权）。
- **健康检查**：`curl -fsS http://127.0.0.1:15630/api/status`（镜像自带 curl，端点与镜像自己的 HEALTHCHECK 一致）。
- **收信**：默认只做出站 IMAP 归档同步；可选的 SMTP 收信端口本包没有放出（要放需要 `application.ingress`，按需再加）。
- **图标**：上游 `web/src/assets/logo.svg`。

## 商店状态

- **喵喵商店**：`cloud.lazycat.app.bichon` 由 Action 发布（`stores.private`）。
- **官方商店**：镜像模式不满足官方商店「镜像必须来自 `registry.lazycat.cloud`」的要求，只发喵喵商店。
