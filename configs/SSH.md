## SSH

1. Key Gen

```sh
# Generate a secure ED25519 key (Recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy the key to a remote server automatically
ssh-copy-id user@server

# GitHub / GitLab Setup

## MacOS
pbcopy < ~/.ssh/id_ed25519.pub

## Windows
clip < ~/.ssh/id_ed25519.pub

## Linux
cat ~/.ssh/id_ed25519.pub # Highlight and copy the output manually, or use `xclip` if installed

# Test your connection
ssh -T git@gitlab.com # or github.com
```

**Manual Alternative**: If `ssh-copy-id` isn't available, manually append the contents of your local `~/.ssh/id_ed25519.pub` into the remote server's `~/.ssh/authorized_keys` file.
Approved keys in `authorized_keys` follow this format: `ssh-rsa pub-key` identifier or `ssh-ed25519 pub-key` identifier.

2. `~/.ssh/config`

```ssh-config
Host nickname
    HostName example.com
    User username
    IdentityFile ~/.ssh/id_ed25519
```

**Note:** Approved keys are saved in `~/.ssh/authorized_keys`, with following format: `ssh-rsa pub-key identifier`.

3. Port forwarding

```sh
ssh -vfNR Y:localhost:X hostname  # Run on A, A:X -> M:Y
ssh -vfNL Z:localhost:Y hostname  # Run on B, M:Y -> B:Z
```

- `-v`: verbose, `-f`: background, `-N`: no-shell

- `-R`: it tells *Remote* to open the port `Y` and listen
- `-L`: it tells *Local* to open the port `Z` and listen

**Note**: You can replace `localhost` with: `127.0.0.1`, `0.0.0.0` or `hostname`.

4. SSH Server

```sh
# Install (Fedora/RHEL)
sudo dnf install openssh-server

# Install (Debian/Ubuntu)
sudo apt install openssh-server

# Enable and start
sudo systemctl enable --now sshd
sudo systemctl status sshd
```

`/etc/ssh/sshd_config` (hardened):

```
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
AllowUsers sketch
```

```sh
# Validate config before restarting
sudo sshd -t

# Restart after config change
sudo systemctl restart sshd
```

**Note:** Always keep an active session open when changing sshd config remotely — if you lock yourself out, you can't get back in.

5. Connection Multiplexing

Reuses a single SSH connection for multiple sessions — avoids repeated handshakes.

`~/.ssh/config`:

```ssh-config
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
```

```sh
mkdir -p ~/.ssh/sockets
```

- `ControlMaster auto`: first connection becomes the master, subsequent ones piggyback
- `ControlPath`: socket file location (`%r` = user, `%h` = host, `%p` = port)
- `ControlPersist 600`: keep master alive 10 min after last session closes

```sh
# Check status of a master connection
ssh -O check hostname

# Kill a master connection
ssh -O exit hostname
```

6. ProxyCommand / ProxyJump

Connect to a host through an intermediary (bastion/jump host):

```ssh-config
# Via a bastion
Host internal
    HostName 10.0.0.5
    User deploy
    ProxyCommand ssh bastion -W %h:%p

# Modern alternative (OpenSSH 7.3+)
Host internal
    HostName 10.0.0.5
    User deploy
    ProxyJump bastion

# Chaining multiple jumps
Host deep-internal
    HostName 10.0.1.99
    ProxyJump bastion1,bastion2
```

One-off from command line:

```sh
ssh -J bastion user@10.0.0.5
ssh -o ProxyCommand="ssh bastion -W %h:%p" user@10.0.0.5
```

7. AWS SSM Session Manager (no open inbound ports needed)

```sh
# Start a shell session
aws ssm start-session --target i-0123456789abcdef0

# SSH over SSM (add to ~/.ssh/config)
Host i-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
    User ec2-user
    IdentityFile ~/.ssh/id_ed25519
```

```sh
# Now just SSH normally
ssh i-0123456789abcdef0

# Port forwarding over SSM
aws ssm start-session --target i-0123456789abcdef0 \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["5432"],"localPortNumber":["5432"]}'
```

**Note:** Requires the SSM agent running on the instance and appropriate IAM permissions. No security groups or SSH keys needed for basic `start-session`.
