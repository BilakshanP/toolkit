## Hash

Shell builtin with two modes: command cache and named directories (zsh).

### Command hash table (bash/zsh)

The shell caches command-to-path lookups to avoid searching `$PATH` every time.

```sh
hash              # show cached command paths
hash -r           # clear the cache (useful after installing new binaries)
hash ls           # re-hash a specific command
```

### Named directories (zsh only, `-d` flag)

Create path bookmarks that work anywhere you'd use a path.

```zsh
# Create
hash -d ws=~/Workspace
hash -d proj=~/Workspace/myproject

# Use
cd ~ws
ls ~proj/src
vim ~ws/toolkit/README.md

# Prompt integration (%~ shows named dirs)
# ~/Workspace/toolkit → ~ws/toolkit

# List all
hash -d

# Remove
unhash -d ws
```

Add to `.zshrc` to persist across sessions.
