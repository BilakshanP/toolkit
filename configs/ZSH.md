## Zsh

### Prompt

```zsh
setopt PROMPT_SUBST

git_info() {
  local hash=$(git rev-parse --short HEAD 2>/dev/null)
  [[ -z $hash ]] && return
  local branch=$(git branch --show-current 2>/dev/null)
  local dirty=""
  [[ -n $(git status --porcelain --untracked-files=no 2>/dev/null) ]] && dirty="*"
  local arrows=""
  if [[ -n $branch ]]; then
    local ahead behind
    ahead=$(git rev-list --count @{u}..HEAD 2>/dev/null)
    behind=$(git rev-list --count HEAD..@{u} 2>/dev/null)
    [[ $ahead -gt 0 ]] && arrows+="↑${ahead}"
    [[ $behind -gt 0 ]] && arrows+="↓${behind}"
  fi
  local stash=$(git stash list 2>/dev/null | wc -l | tr -d ' ')
  local stash_info=""
  [[ $stash -gt 0 ]] && stash_info=" {${stash}}"
  if [[ -n $branch ]]; then
    echo " ($branch) ${hash}${dirty}${arrows:+ $arrows}${stash_info}"
  else
    echo " (detached) ${hash}${dirty}${stash_info}"
  fi
}

venv_info() {
  [[ -n $VIRTUAL_ENV ]] && echo " (${VIRTUAL_ENV:t})"
}

PROMPT='%F{yellow}%* %D{%d-%m-%Y}%f %F{blue}%B%~%b%f%F{green}$(git_info)%f%F{magenta}$(venv_info)%f%(?.. %F{red}[%?]%f)
%(?.%B%F{green}.%B%F{red})%# %f%b'
```

Output: `18:04:34 29-06-2026 ~/project (main) a1b2c3d* ↑2↓1 {3} (venv) [1]`

- Yellow — time + date
- Blue bold — path
- Green — branch + short hash, `*` if dirty tracked files, `↑↓` ahead/behind, `{n}` stash count
- Magenta — virtualenv (only when active)
- Red `[N]` — exit code (only on failure)
- `%` green on success, red on failure
- Detached HEAD shows `(detached) hash`

### Terminal title

Use iTerm2's built-in title settings: Settings → Profiles → General → Title → "Session Name + Job (optionally with parameters)". No shell config needed.

### History

```zsh
HISTSIZE=10000
SAVEHIST=10000
HISTFILE=~/.zsh_history
setopt SHARE_HISTORY       # share history across all open terminals in real-time
setopt HIST_IGNORE_DUPS    # skip consecutive duplicate entries
setopt HIST_IGNORE_SPACE   # commands starting with space are not saved
```

### Options

```zsh
setopt AUTO_CD             # type a directory name to cd into it
```

### Completion

```zsh
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}' 'r:|/=*'
autoload -Uz compinit && compinit
```

- Case-insensitive matching (`m:{a-z}={A-Z}`)
- Recursive path expansion (`r:|/=*`) — `/u/lo/b` → Tab → `/usr/local/bin`

### Spelling correction

```zsh
setopt CORRECT         # correct mistyped commands
setopt CORRECT_ALL     # correct all arguments too
```

Prompts with `zsh: correct 'X' to 'Y' [nyae]?` on typos. Drop `CORRECT_ALL` if argument correction is too aggressive.

### Plugins (via Homebrew)

```sh
brew install zsh-autosuggestions zsh-syntax-highlighting
```

```zsh
source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh
source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

- Autosuggestions — gray predictions as you type, press → to accept
- Syntax highlighting — valid commands green, invalid red

### .zshrc.d

```zsh
if [ -d ~/.zshrc.d ]; then
    for rc in ~/.zshrc.d/*(N); do
        [ -f "$rc" ] && source "$rc"
    done
    unset rc
fi
```

Drop individual files into `~/.zshrc.d/` (e.g., `aliases.zsh`, `functions.zsh`) and they'll be sourced automatically.

### Bookmarks

```zsh
hash -d ws=~/Workspace
# then: cd ~ws
```

### ls

```zsh
alias l='ls -GFA'
```

- `-G` — colorized output (directories blue, executables red, symlinks magenta)
- `-F` — append type indicator suffix to entries
- `-A` — show hidden files (except `.` and `..`)

#### `-F` suffix indicators

| Suffix | Type |
|--------|------|
| `/` | Directory |
| `*` | Executable |
| `=` | Unix socket |
| `\|` | Named pipe (FIFO) |
| `@` | Symlink |
| (none) | Regular file |

#### `ls -l` first-character type codes

| Char | Type |
|------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symlink |
| `s` | Socket |
| `p` | Pipe (FIFO) |
| `b` | Block device |
| `c` | Character device |

### Misc

```sh
zsh -f    # start zsh without reading any config (.zshrc, .zprofile, .zshenv)
```

### Using prompt formatting in scripts

The `%`-sequences in `PROMPT` are expanded by zsh's built-in prompt engine. Use `print -P` to access the same formatting in scripts:

```zsh
print -P '%F{red}Error:%f something went wrong'
print -P '%F{green}✓%f Done in %B3.2s%b'
print -P '%F{cyan}%~%f'   # current dir with ~ abbreviation
```

Helper pattern:

```zsh
info()  { print -P "%F{blue}[INFO]%f  $1" }
warn()  { print -P "%F{yellow}[WARN]%f  $1" }
err()   { print -P "%F{red}[ERROR]%f $1" }
ok()    { print -P "%F{green}[OK]%f    $1" }
```

Notes:
- `print -P` is zsh-only — for portable scripts use ANSI escapes (`\033[31m`)
- `$(...)` in `PROMPT` works because of `setopt PROMPT_SUBST` — it re-evaluates on every render
- Docs: `man zshmisc` → "EXPANSION OF PROMPT SEQUENCES", or https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html

### Prompt escapes reference

```
%n      username
%m      hostname (short)
%~      current dir (~ abbreviated, respects hash -d)
%*      time HH:MM:SS
%D{fmt} date with strftime format
%#      % for user, # for root
%?      last exit code
%F{c}   start color
%f      end color
%B %b   bold on/off
%(?.T.F) ternary — T if exit 0, F otherwise
```
