## WSL

### Host (Windows) - `~/.wslconfig`

```toml
[wsl2]
firewall=false
autoProxy=false
networkingMode=mirrored
```

### Guest (WSL) - `/etc/wsl.conf`

```toml
# Refer to https://learn.microsoft.com/en-us/windows/wsl/wsl-config#wslconf
# for the full set of configuration options.
[boot]
systemd=false               # this will override resolv.conf

# [user]
# default=sketch

[network]
hostname=fedora
# generateHosts=false       # '/etc/hosts'
# generateResolvConf=false  # '/etc/resolv.conf'

[interop]
appendWindowsPath=false
```

**Note:** You might have to modify `/etc/host.conf` and `/etc/hosts`. Usually commenting out unwanted hostname should suffice.

#### Examples

1. `/etc/hosts`

```
127.0.0.1       localhost
127.0.0.1       fedora

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

2. `/etc/resolv.conf`

```
nameserver 8.8.8.8
# search .
```

**Note:** If `systemd` is enabled, it will overwrite this. It could be symlinked, the you would have to remove it first.
