## Tmux

Modern terminal multiplexer. Prefix key: `Ctrl+B` (default).

```sh
# Start a named session
tmux new -s mysession

# Detach
Ctrl+B, D

# List sessions
tmux ls

# Reattach
tmux attach -t mysession
tmux a -t mysession

# Kill a session
tmux kill-session -t mysession

# Kill all sessions
tmux kill-server
```

### Windows (tabs)

```
Ctrl+B, C          # new window
Ctrl+B, N          # next window
Ctrl+B, P          # previous window
Ctrl+B, 0-9        # jump by number
Ctrl+B, ,          # rename window
Ctrl+B, &          # kill window
Ctrl+B, W          # list windows (interactive)
```

### Panes (splits)

```
Ctrl+B, %          # vertical split
Ctrl+B, "          # horizontal split
Ctrl+B, Arrow      # move between panes
Ctrl+B, X          # kill pane
Ctrl+B, Z          # zoom pane (toggle fullscreen)
Ctrl+B, {          # swap pane left
Ctrl+B, }          # swap pane right
Ctrl+B, Space      # cycle pane layouts
```

### Resize Panes

```
Ctrl+B, Ctrl+Arrow          # resize in arrow direction
```

### Copy Mode (scrollback)

```
Ctrl+B, [          # enter copy mode (navigate with arrows/PgUp)
q                  # exit copy mode
Space              # start selection (in copy mode)
Enter              # copy selection
Ctrl+B, ]          # paste
```

### Detached Commands

```sh
# Run a command in a new detached session
tmux new -d -s backup 'rsync -avz src/ host:/dest/'

# Send a command to an existing session
tmux send-keys -t mysession 'ls -la' Enter
```

### Config (`~/.tmux.conf`)

```sh
# Remap prefix to Ctrl+A
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Mouse support
set -g mouse on

# Start window numbering at 1
set -g base-index 1
setw -g pane-base-index 1

# Better splits
bind | split-window -h
bind - split-window -v
```

```sh
# Reload config
tmux source-file ~/.tmux.conf
# or inside tmux: Ctrl+B, :source-file ~/.tmux.conf
```
