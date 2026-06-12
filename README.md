# sing-box-client

Dockerized sing-box with Hysteria 2 outbound and CN split routing.

## Features

- Single container, sing-box natively supports Hysteria 2
- Built-in split routing: CN traffic direct, everything else via hy2
- Pre-loaded rule-sets from [DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata)

## Quick Start

```bash
docker run -d --name sing-box-client --restart always \
  -e HY2_URI="hysteria2://password@server:port?sni=example.com&insecure=0" \
  -p 10808:10808 -p 10809:10809 \
  ghcr.io/shangui999/sing-box-client:latest
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `HY2_URI` | Yes | - | Full hysteria2:// URI |
| `HY2_SOCKS_PORT` | No | `10808` | SOCKS5 proxy listen port |
| `HY2_HTTP_PORT` | No | `10809` | HTTP proxy listen port |

## Routing Rules

| Traffic | Action |
|---------|--------|
| CN domains/IPs | Direct |
| Private domains/IPs | Direct |
| Everything else | hy2 proxy |

## Upgrade Rule-sets

Rule-sets are baked into the image. Rebuild to get latest:

```bash
docker build --no-cache -t ghcr.io/shangui999/sing-box-client:latest .
docker push ghcr.io/shangui999/sing-box-client:latest
```

## License

MIT
