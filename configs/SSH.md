## SSH

1. Key Gen

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-copy-id user@server # aoutomatic or manually append it to ~/.ssh/authorized_keys
```

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
