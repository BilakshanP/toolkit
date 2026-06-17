## Tee

Reads stdin and writes to both stdout and file(s) simultaneously.

```sh
# Write to file and stdout
echo "hello" | tee output.txt

# Append instead of overwrite
echo "hello" | tee -a output.txt

# Write to multiple files
echo "hello" | tee file1.txt file2.txt

# Pipe through tee to another command
ls -la | tee listing.txt | grep ".md"

# Capture command output while still seeing it
make 2>&1 | tee build.log

# Write to file as root (can't just redirect with sudo)
echo "line" | sudo tee /etc/somefile
echo "line" | sudo tee -a /etc/somefile   # append

# Discard stdout, only write to file
echo "hello" | tee output.txt > /dev/null
```

- `-a`: append to file instead of overwriting
- `-i`: ignore interrupt signals

### Splitting stdout and stderr

```sh
# stdout to one file, stderr to another
command > stdout.log 2> stderr.log

# stdout to file + screen, stderr to separate file
command 2> stderr.log | tee stdout.log

# stderr to file + screen, stdout to separate file
command > stdout.log 2>&1 1>/dev/tty | tee stderr.log
# simpler with process substitution:
command 2> >(tee stderr.log >&2) > stdout.log
```

### Redirect combos

```sh
# stdout only to file
command > out.log

# stderr only to file
command 2> err.log

# stdout + stderr to same file
command > all.log 2>&1
command &> all.log                         # bash shorthand

# stdout + stderr to file + screen
command 2>&1 | tee all.log

# stdout to file, stderr to file, both to screen
command > >(tee stdout.log) 2> >(tee stderr.log >&2)
```
