## Rsync

```sh
# Local to remote
rsync -avz src/ user@host:/dest/

# Remote to local
rsync -avz user@host:/src/ dest/

# Dry run (preview without changes)
rsync -avzn src/ user@host:/dest/

# With delete (mirror — removes files on dest not in src)
rsync -avz --delete src/ user@host:/dest/

# Exclude patterns
rsync -avz --exclude '.git' --exclude 'node_modules' src/ user@host:/dest/

# With progress
rsync -avz --progress src/ user@host:/dest/

# Resume partial transfers
rsync -avz --partial --progress src/ user@host:/dest/

# Custom SSH port
rsync -avz -e 'ssh -p 2222' src/ user@host:/dest/

# Custom identity file
rsync -avz -e 'ssh -i ~/.ssh/id_ed25519' src/ user@host:/dest/

# Local only (no SSH)
rsync -av /src/folder/ /backup/folder/
```

- `-a`: archive (preserves permissions, symlinks, timestamps)
- `-v`: verbose
- `-z`: compress during transfer
- `-n`: dry run
- `--delete`: delete extraneous files on dest
- `--partial`: keep partially transferred files (resumable)

**Note:** Trailing slash on `src/` matters — `src/` copies contents, `src` copies the directory itself.

### Continuous Sync

```sh
# Poll every 5 seconds
watch -n 5 rsync -avz --delete src/ user@host:/dest/

# Event-driven (Linux, needs inotify-tools)
while inotifywait -r -e modify,create,delete src/; do
    rsync -avz --delete src/ user@host:/dest/
done
```
