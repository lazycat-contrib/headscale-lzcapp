# Headscale for LazyCat

This repository packages Headscale with the Headplane Web UI as a LazyCat LPK v2 application.

Chinese documentation: [README.zh-CN.md](README.zh-CN.md)

## Images

- `headscale/headscale:v0.29.3`
- `ghcr.io/tale/headplane:0.7.0`

The application version follows the Headscale image version.

## Runtime Layout

Headscale keeps the official directory-style layout:

- `/lzcapp/var/headscale/config` -> `/etc/headscale`
- `/lzcapp/var/headscale/lib` -> `/var/lib/headscale`
- `/var/run/headscale` uses tmpfs, matching the upstream container guidance.

Headplane shares the Headscale config directory so it can read `config.yaml` and `dns_records.json`. Docker integration is disabled by default to avoid making Headplane startup depend on Docker API availability. The Playground Docker socket is still mounted through `compose_override` for compatibility.

The Headscale container is distroless, so a small `config-init` service initializes writable configuration directories before Headscale starts.

## First Use

The default launcher entry opens Headplane at `/admin/`. The Headscale control server and API remain available at `/` for Tailscale clients.

Create a Headscale API key from the Headscale service and use it to log in to Headplane:

```bash
lzc-cli docker exec headscale /ko-app/headscale apikeys create --expiration 90d
```

## Public Access And Domains

`public_path` is set to `/`, so the Headscale control endpoint, API, and Headplane web UI are not intercepted by LazyCat login. This is required for Tailscale clients to reach the control server.

For normal use, expose the LazyCat app domain over HTTPS and use that URL as the Headscale server URL. If you use a custom domain, set the `Public URL` install parameter to the full URL, for example `https://hs.example.com`.

The reverse proxy in front of Headscale must support the Tailscale control protocol WebSocket upgrade. Per the Headscale reverse proxy documentation, Tailscale clients use `POST` for the WebSocket upgrade and the `Upgrade` header value is `tailscale-control-protocol`.

Do not set the MagicDNS tailnet domain to the same domain as the public Headscale URL. The MagicDNS base domain is an internal tailnet DNS suffix and does not need public DNS records. For example, use `https://hs.example.com` as the server URL and `tailnet.example.com` or `headscale.lan` as the MagicDNS base domain.

## Tailscale Client Setup

Connect a client to this Headscale server:

```bash
tailscale up --login-server=https://your-headscale-domain
```

If the client is already logged in elsewhere, reset or switch it first:

```bash
tailscale logout
tailscale up --login-server=https://your-headscale-domain
```

## Subnet Routes

To forward a LAN subnet into the tailnet, run this on the node that can reach that subnet:

```bash
tailscale up --login-server=https://your-headscale-domain --advertise-routes=192.168.1.0/24
```

Then open Headplane, go to the routes view, and enable the advertised route. Other clients must use `--accept-routes` when needed:

```bash
tailscale up --login-server=https://your-headscale-domain --accept-routes
```

## Exit Node

To make a node forward Internet traffic for other clients:

```bash
tailscale up --login-server=https://your-headscale-domain --advertise-exit-node
```

Enable the route in Headplane, then connect a client through that exit node:

```bash
tailscale up --login-server=https://your-headscale-domain --exit-node=<node-name-or-100.x-address>
```

## DERP

The app includes an optional install parameter, `Enable Embedded DERP`. It is disabled by default.

Enable it only when you want this Headscale instance to provide embedded DERP/STUN relay service. When enabled, the manifest publishes UDP `3478` to the Headscale service, and the generated Headscale config sets `derp.server.enabled: true`.

Requirements:

- `server_url` must be HTTPS.
- UDP `3478` must be reachable from the public Internet.
- HTTP/HTTPS traffic still goes through the LazyCat app domain.

If embedded DERP is not enabled, Headscale uses the default external DERP map from Tailscale.

## Notes

The default configuration follows the Headscale 0.29.3 documentation:

- TLS is terminated by LazyCat, so `tls_cert_path` and `tls_key_path` are empty.
- The startup warning `listening without TLS but ServerURL does not start with http://` is expected when TLS is handled by the reverse proxy.
- `/etc/headscale` and `/var/lib/headscale` are persistent directories.
- Cloudflare Proxy/Tunnel is not recommended for the Headscale control endpoint unless it properly supports the required WebSocket POST upgrade.

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
- Headscale DERP docs: https://headscale.net/stable/ref/derp/
- Headscale routes docs: https://headscale.net/stable/ref/routes/
- Headplane website: https://headplane.net/
- Headplane source: https://github.com/tale/headplane
- Headplane configuration docs: https://headplane.net/configuration/
- Headplane Docker docs: https://headplane.net/install/docker
