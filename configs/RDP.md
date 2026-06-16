## RDP (Remote Desktop)

```sh
# Connect from Windows
mstsc /v:192.168.1.250
mstsc /v:192.168.1.250:3389           # explicit port
mstsc /v:hostname /w:1920 /h:1080     # custom resolution
mstsc /v:hostname /f                   # fullscreen

# Connect from Linux (install: sudo dnf install freerdp)
xfreerdp /v:192.168.1.250 /u:username /p:password
xfreerdp /v:192.168.1.250 /u:username /size:1920x1080 /dynamic-resolution
xfreerdp /v:192.168.1.250 /u:username /f   # fullscreen
```

- Default port: `3389`
- Target machine needs Remote Desktop enabled (Windows: Settings → System → Remote Desktop)

**Credentials via command line (Windows):**

```cmd
cmdkey /generic:192.168.1.250 /user:username /pass:password
cmdkey /generic:192.168.1.250 /user:username /pass   # prompts for password
cmdkey /delete:192.168.1.250           # remove saved credentials
```

**RDP over SSH tunnel (access remote desktop through SSH):**

```sh
ssh -fNL 3389:localhost:3389 user@server-ip
mstsc /v:localhost
```
