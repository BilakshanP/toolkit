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

### Local config (`~/.config/ss-local.json`)

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
ss-local -c ~/.config/ss-local.json
```

### Run as background service (LaunchAgent)

Create `~/Library/LaunchAgents/com.shadowsocks.local.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.shadowsocks.local</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/ss-local</string>
        <string>-c</string>
        <string>/Users/bipurohi2601/.config/ss-local.json</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

Load/unload:

```sh
# Start (and enable on login)
launchctl load ~/Library/LaunchAgents/com.shadowsocks.local.plist

# Stop
launchctl unload ~/Library/LaunchAgents/com.shadowsocks.local.plist
```

Auto-starts on login, restarts if it crashes.

### Shell toggle (`sox`)

Add to your shell rc (e.g., `~/.zshrc.d/proxy.sh`):

```sh
sox() {
    local plist="$HOME/Library/LaunchAgents/com.shadowsocks.local.plist"
    local label="com.shadowsocks.local"

    is_loaded() {
        launchctl print "gui/$(id -u)/$label" >/dev/null 2>&1
    }

    case "$1" in
        on)
            if is_loaded; then
                echo "[sox] Already enabled."
            else
                launchctl load "$plist" &&
                    echo "[sox] Enabled." ||
                    echo "[sox] Failed to enable."
            fi
            ;;
        off)
            if is_loaded; then
                launchctl unload "$plist" &&
                    echo "[sox] Disabled." ||
                    echo "[sox] Failed to disable."
            else
                echo "[sox] Already disabled."
            fi
            ;;
        "")
            if is_loaded; then
                echo "[sox] Active."
            else
                echo "[sox] Inactive."
            fi
            ;;
        *)
            echo "Usage: sox [on|off]"
            return 1
            ;;
    esac
}
```

Usage:

```sh
sox        # show status
sox on     # enable
sox off    # disable
```

### Firefox Configuration (PAC file + DoH)

#### PAC file (`~/.config/proxy.pac`)

Single static file — never changes. Uses SOCKS5 with DIRECT fallback so Firefox automatically bypasses the proxy when ss-local is off:

```js
function FindProxyForURL(url, host) {
    if (
        isPlainHostName(host) ||
        host === "127.0.0.1" ||
        host === "::1" ||
        shExpMatch(host, "*.local")
    ) {
        return "DIRECT";
    }

    return "SOCKS5 127.0.0.1:1080; DIRECT";
}
```

In Firefox: Settings → Network Settings → **Automatic proxy configuration URL**:

```
file:///Users/<username>/.config/proxy.pac
```

#### Firefox proxy exceptions ("No proxy for")

Add your DoH provider so it never routes through SOCKS:

```
cloudflare-dns.com
```

#### DNS over HTTPS (about:config)

```
network.trr.mode = 3
network.trr.uri = https://cloudflare-dns.com/dns-query
network.trr.bootstrapAddr = 1.1.1.1
```

Disable the UI DoH setting (Privacy & Security → DNS over HTTPS → Off) — use the `about:config` settings above instead.

#### DoH exceptions

Add local domains that should resolve via system DNS (not DoH):

```
stream.trym.in
```

> **Gotcha (Firefox bug):** Firefox routes TRR/DoH connections through the SOCKS proxy and doesn't recover when the proxy dies, even if the PAC file specifies `; DIRECT` fallback. The fix is adding the DoH provider hostname to Firefox's "No proxy for" exceptions so TRR always connects DIRECT. See [Bug 1230803](https://bugzilla.mozilla.org/show_bug.cgi?id=1230803).

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
- ss-local not running → restart with `ss-local -c ~/.config/ss-local.json`

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
