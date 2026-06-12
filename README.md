# mihomo-client

基于 mihomo (Clash Meta) 的 Docker 化 Hysteria 2 客户端，内置国内流量分流。

## 特性

- 单容器，mihomo 原生支持 Hysteria 2
- 混合端口：一个端口同时支持 HTTP 和 SOCKS5
- 内置分流规则：国内流量直连，其余走代理
- DNS 分流 + fake-ip 模式，防 DNS 污染
- GeoIP/GeoSite 规则每 24 小时自动更新
- 支持端口跳跃 (port hopping) + brutal 拥塞控制
- 构建时自动拉取最新 mihomo 版本
- API 认证保护

## 快速开始 (docker-compose)

```yaml
services:
  mihomo:
    image: ghcr.io/shangui999/mihomo-client:latest
    container_name: mihomo-client
    restart: always
    environment:
      - HY2_URI=hysteria2://YOUR_PASSWORD@YOUR_SERVER:PORT?sni=example.com&insecure=1&mport=50000-60000#my-proxy
    ports:
      - "10808:10808"   # 混合端口 (HTTP + SOCKS5)
      - "9090:9090"     # 管理面板 API
```

## 快速开始 (docker run)

```bash
docker run -d --name mihomo-client --restart always \
  -e HY2_URI="hysteria2://password@server:port?sni=example.com&insecure=1" \
  -p 10808:10808 -p 9090:9090 \
  ghcr.io/shangui999/mihomo-client:latest
```

## 环境变量

| 变量 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `HY2_URI` | 是 | - | 完整的 hysteria2:// URI |
| `MIXED_PORT` | 否 | `10808` | 混合端口，同时支持 HTTP 和 SOCKS5 |
| `MIHOMO_SECRET` | 否 | 随机 UUID | 管理面板 API 认证密钥 |

> 需要 VLESS / SS 多协议支持？使用 `dev` 标签：`ghcr.io/shangui999/mihomo-client:dev`

## URI 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `sni` | TLS SNI 域名 | `sni=bing.com` |
| `insecure` | 跳过证书验证 (0/1) | `insecure=1` |
| `allowInsecure` | 同 insecure | `allowInsecure=1` |
| `mport` | 端口跳跃范围 | `mport=50000-60000` |
| `hop-interval` | 端口跳跃间隔(秒) | `hop-interval=30` |
| `up` | 上行带宽 (brutal) | `up=100mbps` |
| `down` | 下行带宽 (brutal) | `down=100mbps` |
| `obfs` | 混淆类型 | `obfs=salamander` |
| `obfs-password` | 混淆密码 | `obfs-password=xxx` |
| `cwnd` | 拥塞窗口 | `cwnd=100` |
| `alpn` | ALPN 协议 | `alpn=h3` |
| `fingerprint` | TLS 指纹 | `fingerprint=chrome` |
| `#name` | 代理名称 (fragment) | `#my-proxy` |

## 使用方式

### 作为 HTTP 代理

```bash
curl -x http://YOUR_HOST_IP:10808 https://www.google.com
export http_proxy=http://YOUR_HOST_IP:10808
export https_proxy=http://YOUR_HOST_IP:10808
```

### 作为 SOCKS5 代理

```bash
curl -x socks5h://YOUR_HOST_IP:10808 https://www.google.com
```

## 分流规则

| 顺序 | 匹配条件 | 动作 |
|------|----------|------|
| 1 | 私有 IP | 直连 |
| 2 | 私有域名 | 直连 |
| 3 | 国内域名 (geosite:cn) | 直连 |
| 4 | 国内 IP (geoip:cn) | 直连 |
| 5 | 其余所有流量 | 走 hy2 代理 |

## DNS 配置

| 查询类型 | DNS 服务器 | 路径 |
|----------|-----------|------|
| 默认 | 223.5.5.5 / 114.114.114.114 | 直连 UDP |
| 国内/私有域名 | DoH (doh.pub, alidns) | 直连 |
| 国外域名 | DoH (cloudflare, google) | 经 hy2 代理 |

## 管理面板

容器启动日志会打印 API Secret。

- [Yacd](http://yacd.haishan.me/?hostname=YOUR_HOST_IP&port=9090)
- [MetaCubeX Dashboard](http://d.metacubex.one/?hostname=YOUR_HOST_IP&port=9090)

## 镜像标签

| 标签 | 说明 |
|------|------|
| `latest` | 稳定版，仅支持 Hysteria 2 |
| `dev` | 开发版，支持 HY2 + VLESS + SS 多协议 + 多节点 |

## 许可证

MIT
