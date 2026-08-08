# Headscale for LazyCat

This repository packages Headscale with the Headplane Web UI as a LazyCat LPK v2 application.

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

Create a Headscale API key from the Headscale service and use it to log in:

```bash
headscale apikeys create --expiration 90d
```

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
