## Diff & Patches

### Comparing Files and Folders

1. Basic Diff

```sh
# Single file
diff file1.txt file2.txt

# Recursive folder comparison
diff -r folder1/ folder2/

# Summary only (which files differ, not the content)
diff -r --brief folder1/ folder2/
```

2. Unified Format (Recommended)

```sh
diff -u file1.txt file2.txt        # single file
diff -ur folder1/ folder2/         # recursive folders
```

**Why unified format?**
- Shows context around changes (lines before/after)
- Standard format for patches, git, and pull requests
- Much more readable than default diff output

**Example output:**
```diff
--- folder1/script.sh
+++ folder2/script.sh
@@ -10,7 +10,7 @@
 echo "Starting..."
 
 # Configuration
-DB_HOST="localhost"
+DB_HOST="production.example.com"
 DB_PORT=5432
```

**Key symbols:**
- `---` / `+++`: file headers (original / modified)
- `@@ -10,7 +10,7 @@`: line numbers and counts
- ` ` (space): unchanged context lines
- `-`: removed lines
- `+`: added lines

3. Other Useful Options

```sh
diff -w folder1/ folder2/      # ignore whitespace
diff -B folder1/ folder2/      # ignore blank lines
diff -i folder1/ folder2/      # ignore case
diff -u --color folder1/ folder2/  # colored output
```

### Creating Patches

1. From Files or Folders

```sh
# Single file
diff -u old_file.txt new_file.txt > changes.patch

# Folder
diff -ur old_folder/ new_folder/ > changes.patch
```

2. From Git

```sh
# Unstaged changes
git diff > changes.patch

# Staged changes
git diff --cached > changes.patch

# Between commits
git diff abc123 def456 > changes.patch

# Specific file
git diff file.txt > file.patch
```

### Applying Patches

1. Test Before Applying (Recommended)

```sh
# Dry-run to see what would change
patch --dry-run < changes.patch
```

2. Apply the Patch

```sh
# Simple case
patch < changes.patch

# If patch has directory paths (e.g., `a/` and `b/` prefixes)
patch -p1 < changes.patch
```

**`-p` flag:** strips directory levels
- `-p0`: use full path as-is
- `-p1`: strip first level (common for git patches)
- `-p2`: strip two levels

3. Reverse a Patch (Undo)

```sh
patch -R < changes.patch
```

### Using Patches with Git

1. Apply Without Committing

```sh
git apply changes.patch
```

2. Apply as a Commit

```sh
git am changes.patch
```

Creates a commit with metadata from the patch header.

3. Convert Git Diff to Patch

```sh
git format-patch -1 HEAD        # last commit as patch
git format-patch -3 HEAD        # last 3 commits as patches
```

### Workflow Example

```sh
# Developer A: Create changes
diff -ur myapp.py.old myapp.py > fix.patch

# Developer B: Receive fix.patch, then:

# 1. Verify the patch
patch --dry-run < fix.patch

# 2. Apply it
patch < fix.patch

# 3. Verify changes
cat myapp.py

# 4. If something went wrong, undo
patch -R < fix.patch
```

**Note:** Patches only work if the base file is similar. If the target file has drifted significantly, the patch may fail or apply incorrectly.
