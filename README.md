# mihomo-client

Dockerized mihomo (Clash Meta) with Hysteria 2 support and CN split routing.

## Features

- Single container, mihomo natively supports Hysteria 2
- Built-in split routing: CN traffic direct, everything else via proxy
- DNS split with fake-ip mode
- GeoIP/GeoSite databases for rule matching
- Auto-fetch latest mihomo at build time

## Quick Start (docker-compose)

```yaml
services:
  mihomo:
    image: ghcr.io/shangui999/mihomo-client:latest
    container_name: mihomo-client
    restart: always
    environment:
      - HY2_URI=hysteria2://YOUR_PASSWORD@YOUR_SERVER:PORT?sni=example.com&insecure=0&mport=50000-60000#my-proxy
      - HY2_SOCKS_PORT=10808
      - HY2_HTTP_PORT=10809
    ports:
      - "10808:10808"
      - "10809:10809"
      - "9090:9090"  # dashboard API
```

## Quick Start (docker run)

```bash
docker run -d --name mihomo-client --restart always \
  -e HY2_URI="hysteria2://password@server:port?sni=example.com&insecure=0" \
  -p 10808:10808 -p 10809:10809 \
  ghcr.io/shangui999/mihomo-client:latest
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
| Private IPs/domains | Direct |
| CN domains (geosite:cn) | Direct |
| CN IPs (geoip:cn) | Direct |
| Everything else | hy2 proxy |

## DNS

| Query | Server | Path |
|-------|--------|------|
| CN/private domains | DoH (doh.pub, alidns) | Direct |
| Others | DoH (cloudflare, google) | Via hy2 proxy |

## License

MIT
