## PAC (Proxy Auto-Configuration)

A PAC file is a JavaScript file that tells the browser how to route each request — through a proxy, direct, or a fallback chain.

### Basic structure

```js
function FindProxyForURL(url, host) {
    // rules here
    return "DIRECT";
}
```

The function is called for every request. `url` is the full URL, `host` is just the hostname.

### Return values

| Value | Meaning |
|-------|---------|
| `"DIRECT"` | Connect directly, no proxy |
| `"PROXY host:port"` | Use HTTP proxy |
| `"SOCKS5 host:port"` | Use SOCKS5 proxy |
| `"SOCKS host:port"` | Use SOCKS4 proxy |

#### Fallback chains

Separate with `;` — browser tries left to right:

```js
return "SOCKS5 127.0.0.1:1080; DIRECT";
// Try SOCKS5 first, fall back to direct if proxy is down
```

```js
return "PROXY proxy1:8080; PROXY proxy2:8080; DIRECT";
// Try proxy1, then proxy2, then direct
```

### Available functions

| Function | Description |
|----------|-------------|
| `isPlainHostName(host)` | True if no dots in hostname (e.g., `localhost`, `intranet`) |
| `shExpMatch(host, pattern)` | Shell glob match (`*`, `?`) |
| `isInNet(host, network, mask)` | True if host IP is in subnet (triggers DNS lookup!) |
| `dnsResolve(host)` | Resolve hostname to IP (synchronous, blocks) |
| `myIpAddress()` | Returns client's IP address |
| `dnsDomainIs(host, domain)` | True if host ends with domain |
| `localHostOrDomainIs(host, fqdn)` | True if exact match or host is prefix of fqdn |
| `isResolvable(host)` | True if hostname resolves (triggers DNS lookup!) |
| `dnsDomainLevels(host)` | Number of dots in hostname |

### Examples

#### Bypass local, proxy everything else

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

#### Route specific domains through proxy

```js
function FindProxyForURL(url, host) {
    if (
        shExpMatch(host, "*.example.com") ||
        shExpMatch(host, "*.internal.corp")
    ) {
        return "PROXY proxy.corp:8080";
    }

    return "DIRECT";
}
```

#### Different proxies for different domains

```js
function FindProxyForURL(url, host) {
    if (shExpMatch(host, "*.us.example.com")) {
        return "SOCKS5 us-proxy:1080";
    }
    if (shExpMatch(host, "*.eu.example.com")) {
        return "SOCKS5 eu-proxy:1080";
    }
    return "DIRECT";
}
```

### Usage in browsers

**Firefox:** Settings → Network Settings → Automatic proxy configuration URL:

```
file:///path/to/proxy.pac
```

Or hosted:

```
http://server/proxy.pac
```

**Chrome/Edge:** Uses system proxy settings, or launch with:

```sh
chromium --proxy-pac-url="file:///path/to/proxy.pac"
```

### Gotchas

- **`dnsResolve()` and `isInNet()` leak DNS** — they do synchronous system DNS lookups for every request. Your ISP sees these queries. Avoid them if privacy matters.
- **Firefox caches PAC file contents** — for remote PAC URLs, it caches aggressively. Local `file://` PACs are re-read more frequently but not guaranteed per-request.
- **Firefox DoH + SOCKS5 bug** — TRR/DoH connections go through the SOCKS proxy despite PAC exclusions. Workaround: add the DoH provider to Firefox's "No proxy for" field. See [Bug 1230803](https://bugzilla.mozilla.org/show_bug.cgi?id=1230803).
- **`; DIRECT` fallback timeout** — when the proxy is dead, there's a ~1-2 sec delay on the first request while the browser detects the failure. Subsequent requests fall back immediately.
- **SOCKS5 vs SOCKS** — `SOCKS5` proxies DNS through the proxy too (if "Proxy DNS when using SOCKS v5" is enabled). `SOCKS` (v4) doesn't.
