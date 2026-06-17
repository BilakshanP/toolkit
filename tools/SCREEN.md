## Screen

Terminal multiplexer — persistent sessions that survive disconnects.

```sh
# Start a named session
screen -S mysession

# Detach
Ctrl+A, D

# List sessions
screen -ls

# Reattach
screen -r mysession

# Reattach (force detach from elsewhere)
screen -dr mysession

# Kill a session
screen -X -S mysession quit
```

### Windows (inside screen)

```
Ctrl+A, C          # new window
Ctrl+A, N          # next window
Ctrl+A, P          # previous window
Ctrl+A, 0-9        # jump to window by number
Ctrl+A, "          # list windows
Ctrl+A, A          # rename window
```

### Splits

```
Ctrl+A, |          # vertical split
Ctrl+A, S          # horizontal split
Ctrl+A, Tab        # move between splits
Ctrl+A, X          # close current split
```

### Misc

```sh
# Scrollback mode
Ctrl+A, [          # enter copy mode, use arrows/PgUp to scroll, q to exit

# Log session to file
Ctrl+A, H

# Run a command in a detached session
screen -dmS backup rsync -avz src/ host:/dest/
```

**Note:** Prefer tmux for new setups. Use screen when it's the only option on a minimal server.
