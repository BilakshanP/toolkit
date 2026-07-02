## Archive & Compression

### tar

Create and extract tape archives. Does not compress by default — combine with a compressor flag.

```sh
# Create
tar -cf archive.tar file1 dir1          # no compression
tar -czf archive.tar.gz file1 dir1      # gzip
tar -cjf archive.tar.bz2 file1 dir1     # bzip2
tar -cJf archive.tar.xz file1 dir1      # xz

# Extract
tar -xf archive.tar                     # auto-detects compression on most systems
tar -xzf archive.tar.gz                 # explicit gzip
tar -xjf archive.tar.bz2               # explicit bzip2
tar -xJf archive.tar.xz                # explicit xz

# Extract to specific directory
tar -xf archive.tar.gz -C /path/to/dir

# List contents without extracting
tar -tf archive.tar.gz

# Extract a single file
tar -xf archive.tar.gz path/to/file.txt

# Verbose (show files as they're processed)
tar -xvf archive.tar.gz
```

Flags:
- `-c` create, `-x` extract, `-t` list
- `-f` file (must be last flag before filename)
- `-v` verbose
- `-z` gzip, `-j` bzip2, `-J` xz
- `-C` change directory before extracting

### gzip / gunzip / zcat

Compress or decompress single files. Replaces the original file by default.

```sh
# Compress (file.txt → file.txt.gz, original removed)
gzip file.txt

# Keep original
gzip -k file.txt

# Decompress (file.txt.gz → file.txt, .gz removed)
gunzip file.txt.gz
gzip -d file.txt.gz          # same thing

# Keep original when decompressing
gunzip -k file.txt.gz

# Read compressed file to stdout (without decompressing on disk)
zcat file.txt.gz             # macOS/BSD
gzip -dc file.txt.gz        # portable alternative

# Compress from stdin/stdout (piping)
cat file.txt | gzip > file.txt.gz
aws s3 cp s3://bucket/file.gz - | gunzip | tail -50

# Set compression level (1=fastest, 9=smallest, default=6)
gzip -9 file.txt

# Compress multiple files (each gets its own .gz)
gzip file1.txt file2.txt
```

### zip / unzip

Archive + compress multiple files into a single `.zip`. Cross-platform friendly.

```sh
# Create
zip archive.zip file1 file2
zip -r archive.zip dir1/             # recursive (include directory)

# Extract
unzip archive.zip
unzip archive.zip -d /path/to/dir    # extract to specific directory

# List contents
unzip -l archive.zip

# Extract specific file
unzip archive.zip path/to/file.txt

# Overwrite without prompting
unzip -o archive.zip

# Exclude files
zip -r archive.zip dir1/ -x "*.log" "*.tmp"

# Password protect
zip -e archive.zip file1 file2

# Update existing zip (add new/changed files)
zip -u archive.zip newfile.txt
```

### bzip2 / bunzip2 / bzcat

Higher compression ratio than gzip, slower.

```sh
bzip2 file.txt               # → file.txt.bz2
bunzip2 file.txt.bz2         # decompress
bzcat file.txt.bz2           # read to stdout
bzip2 -k file.txt            # keep original
```

### xz / unxz / xzcat

Best compression ratio, slowest.

```sh
xz file.txt                  # → file.txt.xz
unxz file.txt.xz             # decompress
xzcat file.txt.xz            # read to stdout
xz -k file.txt               # keep original
xz -9 file.txt               # max compression
xz -T0 file.txt              # use all CPU cores
```

### Comparison

| Tool | Extension | Ratio | Speed | Use case |
|------|-----------|-------|-------|----------|
| gzip | .gz | Good | Fast | General purpose, logs, pipes |
| bzip2 | .bz2 | Better | Slower | When size matters more than time |
| xz | .xz | Best | Slowest | Distributing source tarballs |
| zip | .zip | Good | Fast | Cross-platform sharing |

### Common patterns

```sh
# Compress a directory into a single .tar.gz
tar -czf project-backup.tar.gz project/

# Stream decompress from S3
aws s3 cp s3://bucket/data.gz - | gunzip | head -100

# Parallel gzip (if pigz is installed)
tar -cf - dir/ | pigz > archive.tar.gz

# Check integrity without extracting
gzip -t file.gz
tar -tzf archive.tar.gz > /dev/null

# Find and compress old logs
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;

# Extract and strip leading directory
tar -xf archive.tar.gz --strip-components=1
```
