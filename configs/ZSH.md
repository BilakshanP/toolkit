## Zsh

### Prompt

```zsh
setopt PROMPT_SUBST

git_info() {
  local branch=$(git branch --show-current 2>/dev/null)
  [[ -z $branch ]] && return
  local hash=$(git rev-parse --short HEAD 2>/dev/null)
  local dirty=""
  [[ -n $(git status --porcelain 2>/dev/null) ]] && dirty="*"
  echo " ($branch) ${hash}${dirty}"
}

venv_info() {
  [[ -n $VIRTUAL_ENV ]] && echo " (${VIRTUAL_ENV:t})"
}

PROMPT='%F{yellow}%* %D{%d-%m-%Y}%f %F{blue}%B%~%b%f%F{green}$(git_info)%f%F{magenta}$(venv_info)%f%(?.. %F{red}[%?]%f)
%(?.%B%F{green}.%B%F{red})%# %f%b'
```

Output: `18:35:25 27-06-2026 ~/project (main) a1b2c3d* (venv) [1]`

- Yellow — time + date
- Blue bold — path
- Green — branch + short hash, `*` if dirty
- Magenta — virtualenv (only when active)
- Red `[N]` — exit code (only on failure)
- `%` green on success, red on failure

### Terminal title (iTerm2)

```zsh
_set_title_precmd() { print -Pn "\e]0;%~\a" }
_set_title_preexec() { print -Pn "\e]0;%~ — ${1%% *}\a" }
precmd_functions+=(_set_title_precmd)
preexec_functions+=(_set_title_preexec)
```

Shows `~/dir` at rest, `~/dir — cmd` while running. The `—` separator avoids ambiguity with spaces in directory names.

**iTerm2 requirement:** Settings → Profiles → General → Title — include "Session Name" and ensure "Applications in terminal may change the title" is enabled.

### History

```zsh
HISTSIZE=10000
SAVEHIST=10000
HISTFILE=~/.zsh_history
setopt SHARE_HISTORY       # share history across all open terminals in real-time
setopt HIST_IGNORE_DUPS    # skip consecutive duplicate entries
```

### Options

```zsh
setopt AUTO_CD             # type a directory name to cd into it
```

### Completion

```zsh
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}'   # case-insensitive
autoload -Uz compinit && compinit
```

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

### Bookmarks

```zsh
hash -d ws=~/Workspace
# then: cd ~ws
```

### Prompt escapes reference

```
%n      username
%m      hostname (short)
%~      current dir (~ abbreviated)
%*      time HH:MM:SS
%D{fmt} date with strftime format
%#      % for user, # for root
%?      last exit code
%F{c}   start color
%f      end color
%B %b   bold on/off
%(?.T.F) ternary — T if exit 0, F otherwise
```
