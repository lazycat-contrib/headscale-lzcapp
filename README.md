# Headscale for LazyCat

This repository packages Headscale with the Headplane Web UI as a LazyCat LPK v2 application.

本仓库将 Headscale 和 Headplane Web 管理界面打包为懒猫微服 LPK v2 应用。默认入口打开 Headplane，Headscale 控制服务保留在根路径供 Tailscale 客户端连接。

## Images

- `headscale/headscale:v0.29.3`
- `ghcr.io/tale/headplane:0.7.0`

The application version follows the Headscale image version.

## Runtime Layout

Headscale keeps the official directory-style layout:

- `/lzcapp/var/headscale/config` -> `/etc/headscale`
- `/lzcapp/var/headscale/lib` -> `/var/lib/headscale`
- `/var/run/headscale` uses tmpfs, matching the upstream container guidance.

Headplane shares the Headscale config directory so it can read and write `config.yaml` and `dns_records.json`.
Docker socket access and the Headscale discovery label are configured through `lzc-build.yml` `compose_override`.

The Headscale container is distroless, so a small `config-init` service initializes writable configuration directories before Headscale starts.

## First Use

The default launcher entry opens Headplane at `/admin`. The secondary Headscale entry keeps the control server path available at `/`.

默认启动器入口会打开 Headplane 管理界面 `/admin`。第二个 Headscale 入口保留根路径 `/`，用于客户端连接控制服务。

Create a Headscale API key from the Headscale service and use it to log in:

```bash
headscale apikeys create --expiration 90d
```

首次使用时，需要在 `headscale` 服务中创建 API Key，然后填入 Headplane 登录界面。

## Custom Domain

Set the optional `Public URL` install parameter to use a custom public Headscale URL, for example `https://hs.example.com`. Leave it empty to use the LazyCat app domain.

自定义域名时，在安装参数 `公网访问地址` 中填写完整 URL，例如 `https://hs.example.com`。留空则使用懒猫应用域名。

The custom domain must reverse proxy to this LazyCat app and support Headscale WebSocket POST upgrades. Do not set the MagicDNS tailnet domain to the same domain as the public Headscale URL.

自定义域名必须正确反代到本应用，并支持 Headscale 所需的 WebSocket POST upgrade。MagicDNS 的 Tailnet 域名不能和 Headscale 公网访问域名相同。

## Headscale Notes

The default configuration follows the Headscale 0.29.3 documentation:

- Headscale runs behind the LazyCat reverse proxy. `config-init` renders the final `server_url` from the LazyCat public application URL before Headscale starts.
- TLS is terminated by LazyCat, so `tls_cert_path` and `tls_key_path` are empty.
- `/var/run/headscale` is tmpfs, while `/etc/headscale` and `/var/lib/headscale` are persistent directories.
- The official Headscale container is distroless, so configuration bootstrap is handled by the `config-init` service.
- Cloudflare Proxy/Tunnel is not recommended for the Headscale control endpoint because the Tailscale control protocol requires WebSocket POST upgrade handling.

## Publishing

`.github/lazycat-action.yml` enables:

- LazyCat Registry delivery for both runtime images.
- Application version discovery from the `headscale` image.
- Official store publishing.
- Private store publishing.

Required GitHub Secrets:

- `LZC_API_TOKEN`
- `APPSTORE_URL`
- `APPSTORE_TOKEN`

Optional GitHub Secrets:

- `LZC_API_HOST`
- `APP_ID`
- `PRIVATE_STORE_GROUP_CODES`

## Upstream Links

- Headscale website: https://headscale.net/stable/
- Headscale source: https://github.com/juanfont/headscale
- Headscale container docs: https://headscale.net/stable/setup/install/container/
- Headscale reverse proxy docs: https://headscale.net/stable/ref/integration/reverse-proxy/
- Headplane website: https://headplane.net/
- Headplane source: https://github.com/tale/headplane
- Headplane configuration docs: https://headplane.net/configuration/
- Headplane Docker docs: https://headplane.net/install/docker
