# Headscale 懒猫应用

本仓库将 Headscale 和 Headplane Web 管理界面打包为懒猫微服 LPK v2 应用。

## 镜像

- `headscale/headscale:v0.29.3`
- `ghcr.io/tale/headplane:0.7.0`

应用版本跟随 Headscale 镜像版本。

## 运行目录

Headscale 使用官方推荐的目录结构：

- `/lzcapp/var/headscale/config` -> `/etc/headscale`
- `/lzcapp/var/headscale/lib` -> `/var/lib/headscale`
- `/var/run/headscale` 使用 tmpfs，符合官方容器部署建议。

Headplane 共享 Headscale 配置目录，用于读取 `config.yaml` 和 `dns_records.json`。默认关闭 Headplane 的 Docker 集成，避免启动依赖 Docker API。Playground Docker socket 仍通过 `compose_override` 按兼容方式挂载。

Headscale 官方镜像是 distroless，配置初始化由独立的 `config-init` 服务完成。

## 首次使用

默认启动器入口打开 Headplane：`/admin/`。Headscale 控制服务和 API 保留在根路径 `/`，供 Tailscale 客户端连接。

创建 Headscale API Key 后填入 Headplane 登录界面：

```bash
lzc-cli docker exec headscale /ko-app/headscale apikeys create --expiration 90d
```

## 公网访问与域名

`public_path` 设置为 `/`，所以 Headscale 控制端点、API 和 Headplane Web UI 都不会被懒猫登录拦截。Tailscale 客户端需要这样才能访问控制服务器。

正常使用时，对外暴露懒猫应用的 HTTPS 域名，并把这个 URL 作为 Headscale 服务地址。如果使用自定义域名，在安装参数 `公网访问地址` 中填写完整 URL，例如 `https://hs.example.com`。

Headscale 官方反代文档要求：反向代理必须支持 Tailscale 控制协议的 WebSocket upgrade。Tailscale 客户端使用 `POST` 做 WebSocket upgrade，`Upgrade` header 的值是 `tailscale-control-protocol`。

MagicDNS 的 Tailnet 域名不要和 Headscale 公网访问域名相同。它是 tailnet 内部 DNS 后缀，不需要公网 DNS 解析。例如公网地址使用 `https://hs.example.com`，MagicDNS 基础域名可使用 `tailnet.example.com` 或 `headscale.lan`。

## Tailscale 客户端连接

客户端连接到这台 Headscale：

```bash
tailscale up --login-server=https://你的Headscale域名
```

如果客户端已经登录到其它控制服务器，先退出再连接：

```bash
tailscale logout
tailscale up --login-server=https://你的Headscale域名
```

## 子网路由

如果要把某个局域网网段转发进 tailnet，在能访问该网段的节点上执行：

```bash
tailscale up --login-server=https://你的Headscale域名 --advertise-routes=192.168.1.0/24
```

然后在 Headplane 的路由页面启用该路由。其它客户端需要时开启接收路由：

```bash
tailscale up --login-server=https://你的Headscale域名 --accept-routes
```

## 出口节点

如果要让某台节点为其它客户端转发公网流量：

```bash
tailscale up --login-server=https://你的Headscale域名 --advertise-exit-node
```

在 Headplane 启用对应路由后，客户端这样使用出口节点：

```bash
tailscale up --login-server=https://你的Headscale域名 --exit-node=<节点名或100.x地址>
```

## DERP

应用提供安装参数 `启用内置 DERP`，默认关闭。

只有需要让当前 Headscale 实例提供内置 DERP/STUN 中继时才启用。启用后 manifest 会开放 UDP `3478` 到 Headscale 服务，并在生成的 Headscale 配置中设置 `derp.server.enabled: true`。

要求：

- `server_url` 必须是 HTTPS。
- UDP `3478` 必须能从公网访问。
- HTTP/HTTPS 流量仍走懒猫应用域名。

如果不启用内置 DERP，Headscale 会使用默认的 Tailscale 外部 DERP map。

## 注意事项

默认配置遵循 Headscale 0.29.3 官方文档：

- TLS 由懒猫反向代理终止，所以 `tls_cert_path` 和 `tls_key_path` 为空。
- 启动日志中的 `listening without TLS but ServerURL does not start with http://` 是预期行为，表示 TLS 由反向代理处理。
- `/etc/headscale` 和 `/var/lib/headscale` 是持久目录。
- Cloudflare Proxy/Tunnel 不推荐用于 Headscale 控制端点，除非它能正确支持所需的 WebSocket POST upgrade。

## 发布

`.github/lazycat-action.yml` 已配置：

- 两个运行镜像交付到懒猫镜像仓库。
- 应用版本从 `headscale` 镜像发现。
- 发布到官方商店。
- 发布到私有商店。

必需 GitHub Secrets：

- `LZC_API_TOKEN`
- `APPSTORE_URL`
- `APPSTORE_TOKEN`

可选 GitHub Secrets：

- `LZC_API_HOST`
- `APP_ID`
- `PRIVATE_STORE_GROUP_CODES`

## 上游链接

- Headscale 官网：https://headscale.net/stable/
- Headscale 源码：https://github.com/juanfont/headscale
- Headscale 容器文档：https://headscale.net/stable/setup/install/container/
- Headscale 反代文档：https://headscale.net/stable/ref/integration/reverse-proxy/
- Headscale DERP 文档：https://headscale.net/stable/ref/derp/
- Headscale 路由文档：https://headscale.net/stable/ref/routes/
- Headplane 官网：https://headplane.net/
- Headplane 源码：https://github.com/tale/headplane
- Headplane 配置文档：https://headplane.net/configuration/
- Headplane Docker 文档：https://headplane.net/install/docker
