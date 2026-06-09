## Bash

1. `.bashrc`

```sh
# User specific aliases and functions
if [ -d ~/.bashrc.d ]; then
    for rc in ~/.bashrc.d/*; do
        if [ -f "$rc" ]; then
            . "$rc"
        fi
    done
fi
unset rc
```

2. `.bashrc.d`

```sh
# aliases.sh
alias ..='cd ..'

alias c='clear'
alias q='exit'
alias e='exit'

alias py='python3'
```

```sh
# functions.sh
cd() {
    builtin cd "$@" && ls
}
```
