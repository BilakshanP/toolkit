## Netcat (nc)

### File Transfer

```sh
# Receiver (run first)
nc -lvp 9001 > output_file

# Sender
nc -N <receiver-ip> 9001 < input_file
```

- `-l`: listen mode
- `-v`: verbose
- `-p`: port
- `-N`: close connection after EOF (so you don't have to Ctrl+D)

**Note:** nc transfers in plaintext. Always encrypt sensitive files with GPG first before sending over nc.

### Chat

```sh
# Side A
nc -lvp 9001

# Side B
nc <ip> 9001
```

Type and press Enter to send. Ctrl+C to quit.

### Port Scanning

```sh
nc -zv <ip> 80          # single port
nc -zv <ip> 20-100      # port range
```

- `-z`: scan only, don't send data

### Check if Port is Open

```sh
nc -zv <ip> <port>
# Connection succeeded = open
# Connection refused   = closed
```

### Netcat as a Proxy (Port Forwarding)

```sh
nc -lvp 9001 | nc <destination-ip> 9002
```
