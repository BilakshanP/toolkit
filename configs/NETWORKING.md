## Networking

### Static IP (non DHCP)

> 192.168.1.xyz
> subnet 255.255.255.0
> gateway 192.168.1.1

#### nmcli (NetworkManager)

```sh
# Show connections
nmcli con show

# Set static IP (wired)
nmcli con mod "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 1.1.1.1"

# Apply
nmcli con up "Wired connection 1"

# Set static IP (WiFi)
nmcli con mod "YourWiFiSSID" \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 8.8.4.4" \
  ipv4.method manual

# Reconnect to apply
nmcli con down "YourWiFiSSID" && nmcli con up "YourWiFiSSID"

# Back to DHCP
nmcli con mod "Wired connection 1" ipv4.method auto
nmcli con up "Wired connection 1"
```

#### ip (temporary, lost on reboot)

```sh
ip addr add 192.168.1.100/24 dev eth0
ip route add default via 192.168.1.1
ip link show                               # list interfaces
```

#### netsh (Windows)

```cmd
netsh interface ip set address "Wi-Fi" static 192.168.1.100 255.255.255.0 192.168.1.1
netsh interface ip set dns "Wi-Fi" static 8.8.8.8
netsh interface ip show config             # show current config
netstat -an                                # all connections and listening ports
```

### DNS

```sh
# Check current DNS
cat /etc/resolv.conf
resolvectl status

# Manual /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1

# Set via nmcli
nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8 1.1.1.1"
nmcli con up "Wired connection 1"
```

### Diagnostics

```sh
# Interfaces & IPs
ip addr                            # all interfaces
ip -br addr                        # brief
ip link                            # link state (up/down)

# Routing
ip route                           # routing table
ip route get 8.8.8.8              # which interface/gateway for a destination

# Connectivity
ping -c 4 8.8.8.8                 # ICMP
ping -c 4 google.com              # also tests DNS resolution

# DNS lookup
dig google.com                     # full query
dig +short google.com              # just the answer
nslookup google.com

# Trace path
traceroute google.com
tracepath google.com               # no root needed

# Ports & connections
ss -tlnp                           # listening TCP ports
ss -ulnp                           # listening UDP ports
ss -tunap                          # all connections with process

# Specific port check
ss -tlnp | grep :8080

# Bandwidth test
curl -o /dev/null http://speedtest.tele2.net/10MB.zip
```

### Firewall

#### firewalld (Fedora/RHEL)

```sh
# Status
systemctl status firewalld
firewall-cmd --state

# List rules
firewall-cmd --list-all

# Open port
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

# Remove port
firewall-cmd --remove-port=8080/tcp --permanent
firewall-cmd --reload

# Allow service
firewall-cmd --add-service=http --permanent
firewall-cmd --reload

# List available services
firewall-cmd --get-services
```

#### ufw (Ubuntu/Debian)

```sh
ufw status
ufw enable
ufw allow 8080/tcp
ufw deny 3000/tcp
ufw delete allow 8080/tcp
ufw allow from 192.168.1.0/24
```

#### iptables (low-level)

```sh
# List rules
iptables -L -n -v

# Allow incoming on port
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# Drop incoming on port
iptables -A INPUT -p tcp --dport 3000 -j DROP

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Save (Fedora/RHEL)
iptables-save > /etc/sysconfig/iptables
```

### Useful Checks

```sh
# What's using a port
ss -tlnp | grep :5432
lsof -i :5432

# Public IP
curl -s ifconfig.me
curl -s icanhazip.com

# Local hostname resolution
getent hosts hostname

# ARP table (who's on the LAN)
ip neigh

# Network interfaces speed/stats
ethtool eth0
cat /proc/net/dev
```
