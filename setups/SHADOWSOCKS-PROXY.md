# Shadowsocks SOCKS5 Proxy Setup

A lightweight SOCKS5 proxy using Shadowsocks on an AWS EC2 instance. Works on desktop (Firefox) and Android (per-app routing).

## AWS Setup

### Instance

- **Type:** t2.micro (free tier eligible)
- **AMI:** Ubuntu 24.04 LTS
- **Storage:** 8 GB gp3 (default)

### Security Group (Inbound)

| Port  | Protocol | Source        | Purpose     |
|-------|----------|---------------|-------------|
| 22    | TCP      | Your IP only  | SSH access  |
| 8388  | TCP      | 0.0.0.0/0    | Shadowsocks |

Outbound: leave default (all traffic allowed).

> **Tip:** Change port 8388 to something random (e.g., 43721) to avoid automated scanners. Update server and client configs to match.

## Server Setup (EC2 Instance)

```sh
sudo apt update && sudo apt install shadowsocks-libev -y
```

Edit `/etc/shadowsocks-libev/config.json`:

```json
{
    "server": "0.0.0.0",
    "mode": "tcp_and_udp",
    "server_port": 8388,
    "local_port": 1080,
    "password": "your-strong-password",
    "timeout": 86400,
    "method": "chacha20-ietf-poly1305"
}
```

> **Important:** `server` must be `"0.0.0.0"` — the default `["::1", "127.0.0.1"]` only listens on localhost and won't accept external connections.

Start and enable:

```sh
sudo systemctl restart shadowsocks-libev
sudo systemctl enable shadowsocks-libev
sudo systemctl status shadowsocks-libev
```

## Client Setup — macOS (Firefox)

### Install

```sh
brew install shadowsocks-libev
```

### Local config (`~/ss-local.json`)

```json
{
    "server": "<ec2-public-ip>",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "your-strong-password",
    "timeout": 86400,
    "method": "chacha20-ietf-poly1305"
}
```

### Run

```sh
ss-local -c ~/ss-local.json
```

### Firefox Configuration

Settings → Network Settings → Manual proxy configuration:

| Field       | Value       |
|-------------|-------------|
| HTTP Proxy  | *(empty)*   |
| HTTPS Proxy | *(empty)*   |
| FTP Proxy   | *(empty)*   |
| SOCKS Host  | 127.0.0.1   |
| Port        | 1080        |

- Select **SOCKS v5**
- Check ✓ **Proxy DNS when using SOCKS v5**

## Client Setup — Android

1. Install **Shadowsocks** from the Play Store
2. Add a profile:
   - Server: `<ec2-public-ip>` (use IP directly, not a domain — see gotcha below)
   - Remote Port: `8388`
   - Password: same as server config
   - Encrypt Method: `chacha20-ietf-poly1305`
3. For per-app routing: enable **Apps VPN** and select only the apps you want proxied (e.g., Firefox)
4. Tap connect

> **Gotcha:** Using a domain name in the server field often fails on Android. The app may try to resolve it through the proxy (circular dependency). Use the IP address directly.

> **Gotcha:** If connection fails on mobile data, your carrier may be blocking port 8388. Switch the server to port `443` (looks like HTTPS, rarely blocked). Update server config, security group, and client to match.

## Client Setup — iOS

Use **Shadowrocket** (paid, ~$3) or **Potatso Lite** (free). Same server/port/password/method settings.

## Testing

Visit [https://whatismyipaddress.com](https://whatismyipaddress.com) — should show EC2 instance IP, not your home IP.

### Troubleshooting

```sh
# Is ss-local running?
ps aux | grep ss-local

# Can you reach the server?
nc -zv <ec2-public-ip> 8388

# Is shadowsocks listening on the server?
sudo ss -tlnp | grep 8388
```

- `nc` times out → security group isn't open on port 8388
- `nc` connects but Firefox doesn't work → check Firefox proxy settings point to 127.0.0.1:1080
- ss-local not running → restart with `ss-local -c ~/ss-local.json`

## Notes

- **Stop EC2 instance** when not using it (free tier = 750 hrs/month)
- Public IP changes on stop/start — assign an **Elastic IP** for a fixed address (free while attached to a running instance)
- Shadowsocks traffic is encrypted + password-protected, so exposing port 8388 is fine

## Alternative: SSH SOCKS5 Tunnel (Desktop Only)

Quick and dirty, no extra software on the server:

```sh
ssh -D 1080 -q -C -N -i ~/.ssh/your-key.pem ubuntu@<ec2-public-ip>
```

Same Firefox proxy settings (SOCKS Host 127.0.0.1:1080). Doesn't work on phones.

## Alternative: WireGuard VPN (System-Wide)

For full system-wide VPN on all devices, use WireGuard instead. Requires opening UDP port 51820. Better for routing all traffic, but overkill if you only need Firefox proxied.
