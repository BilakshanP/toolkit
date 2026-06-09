## GPG

### Setup

1. Install pinentry

```sh
sudo dnf install pinentry-curses
```

2. `~/.gnupg/gpg-agent.conf`

```sh
pinentry-program /usr/bin/pinentry-curses
```

3. `~/.gnupg/dirmngr.conf`

```sh
keyserver hkps://keys.openpgp.org
```

4. Restart agents

```sh
gpgconf --kill gpg-agent
gpgconf --kill dirmngr
```

### Key Generation

```sh
gpg --full-generate-key
# Choose: ECC (sign and encrypt), Curve 25519, expiry as needed
```

GPG automatically creates two keys:

| Key | Type | Usage |
|---|---|---|
| Primary (`sec`) | ed25519 | **[S]** Sign + **[C]** Certify - your identity |
| Subkey (`ssb`) | cv25519 | **[E]** Encrypt only |

**Note:** If the subkey is compromised, revoke and replace it - your identity and web of trust stay intact. If the primary key is compromised, everything is lost.

### Backup

```sh
# Private key (contains both primary + subkey)
gpg --armor --export-secret-keys your@email.com > private_key.asc

# Revocation certificate (already generated at)
~/.gnupg/openpgp-revocs.d/<fingerprint>.rev
```

**Note:** Store both offline on an encrypted USB. The private key is passphrase-protected, but treat it as highly sensitive. No need to back up the public key - it is always derivable from the private key.

### Export
```sh
# Public key (to share with others)
gpg --armor --export your@email.com > public_key.asc

# Private key (for backup/migration)
gpg --armor --export-secret-keys your@email.com > private_key.asc

# Subkeys only (safer for day-to-day use on other machines)
gpg --armor --export-secret-subkeys your@email.com > subkeys.asc

# All public keys
gpg --armor --export > all_public_keys.asc
```

If you don't know your email, list keys first:

```sh
gpg --list-keys          # public keys
gpg --list-secret-keys   # private keys
```

Then export by key ID or fingerprint:

```sh
gpg --armor --export <fingerprint> > public_key.asc
```

### Publish

```sh
gpg --send-keys <fingerprint>
```

Then verify your email at https://keys.openpgp.org/upload so your key is searchable by name.

### Restore on Another Machine

```sh
gpg --import private_key.asc          # also derives and imports public key
gpg --list-keys your@email.com        # verify public key is present
gpg --list-secret-keys your@email.com # verify private key is present
gpg --edit-key your@email.com
# then: trust → 5 (ultimate) → save
```

### Update Password

```sh
gpg --edit-key your@email.com
```

Inside the GPG prompt:

```
passwd
save
```

### Update Expiry

Both the primary key and subkey have independent expiry dates - both must be updated.

```sh
gpg --edit-key your@email.com
```

Inside the GPG prompt:

```
expire      # updates primary key
274         # or 1y, 6m, etc.
y
key 1       # selects subkey (* appears next to ssb)
expire      # updates subkey
274
y
save
```

Then push updated expiry to keyserver:

```sh
gpg --send-keys <fingerprint>
```

### Inspect Encrypted Files

```sh
# Verify signature only
gpg --verify file.txt.asc

# Decrypt + auto-verify signature
gpg --decrypt file.txt.asc

# Inspect file metadata without decrypting (recipient, algorithm, etc.)
gpg --list-packets file.txt.asc
```

**Note:** To verify a signature, the recipient only needs the signer's public key - no secrets involved.

### Trust Model

The keyserver is just a convenience directory - not a trust authority. Security is in your hands:

```sh
# Always verify a contact's fingerprint out-of-band (phone, in person)
gpg --fingerprint their@email.com

# Set trust after verifying
gpg --edit-key their@email.com
# then: trust → 4 (full) or 5 (ultimate, only for your own keys) → save
```

### Daily Usage

```sh
# Get someone's key
gpg --search-keys their@email.com

# Encrypt (to them)
gpg --encrypt --armor --recipient their@email.com file.txt

# Encrypt + sign (to them)
gpg --encrypt --sign --armor --recipient their@email.com file.txt

# Decrypt
gpg --decrypt file.txt.asc

# Sign only
gpg --clearsign document.txt         # signature embedded
gpg --detach-sign document.txt       # separate .sig file

# Verify signature
gpg --verify document.txt.sig document.txt

# Refresh all keys (picks up revocations and expiry updates)
gpg --refresh-keys

# Revoke your key
gpg --import <fingerprint>.rev
gpg --send-keys <fingerprint>
```

### Delete Keys
 
```sh
gpg --delete-keys <fingerprint>                    # public key only
gpg --delete-secret-keys <fingerprint>             # private key only
gpg --delete-secret-and-public-keys <fingerprint>  # both at once
```

### One-off Verification
 
#### Method 1: Temporary Keyring (simple)
 
```sh
TMPGPG=$(mktemp -d)
GNUPGHOME=$TMPGPG gpg --import public_key.asc
GNUPGHOME=$TMPGPG gpg --verify file.sig file
rm -rf $TMPGPG
```
 
Completely isolated from `~/.gnupg` — no key touches your real keyring.
 
#### Method 2: `--no-default-keyring` (broken with keyboxd)
 
```sh
gpg --no-default-keyring --keyring ./tmp-keyring.gpg --import key.asc
gpg --no-default-keyring --keyring ./tmp-keyring.gpg --verify file.sig file
rm tmp-keyring.gpg
```
 
**Warning:** This is silently broken if `use-keyboxd` is set in `~/.gnupg/common.conf` (the default for new GnuPG 2.4 installations). The `--keyring` option is ignored without error, and the key gets imported into your real keyring anyway. This is a [known upstream issue](https://dev.gnupg.org/T7265). Use Method 1 instead.

### Password Based Encryption

```
gpg --symmetric --armor file.txt    # encrypt
gpg --decrypt file.txt.asc          # decrypt
```

### Armoring

`--armor` flag converts it from binary (.gpg) to plaintext (.asc)

1. `.gpg` to `.asc` (binary to text)

```sh
gpg --enarmor < file.gpg > file.asc
```

2. `.asc` to `.gpg` (text to binary)

```sh
gpg --dearmor < file.asc > file.gpg
```

### Directory Encryption

1. Single Step

```sh
# Tar + encrypt in one go
tar czf - folder/ | gpg --symmetric --armor > folder.tar.gz.asc

# Decrypt + extract in one go
gpg --decrypt folder.tar.gz.asc | tar xzf -
```

2. Multi Step

```sh
# Encrypt
tar czf folder.tar.gz folder/
gpg --symmetric --armor folder.tar.gz

# Decrypt
gpg --decrypt folder.tar.gz.asc > folder.tar.gz
tar xzf folder.tar.gz
```

### YubiKey (Recommended)

YubiKey 5 series supports OpenPGP - private key operations happen inside the device, the key never leaves it.

```sh
# Check if GPG sees the YubiKey
gpg --card-status

# Reset OpenPGP applet (wipes all keys and PINs)
ykman openpgp reset
```

- Default User PIN: `123456` (for decrypt/sign operations)
- Default Admin PIN: `12345678` (for settings)
- Change both immediately after setup

**Recommended setup:** Keep the primary key completely offline (cold storage), load only subkeys onto the YubiKey. Even if the YubiKey is stolen, your core identity is safe.

## Netcat (nc)

### File Transfer

```sh
# Receiver (run first)
nc -lvp 9001 > output_file

# Sender
nc -N <receiver-ip> 9001 < input_file
```

- `-l`: listen mode
- `-v`: verbose
- `-p`: port
- `-N`: close connection after EOF (so you don't have to Ctrl+D)

**Note:** nc transfers in plaintext. Always encrypt sensitive files with GPG first before sending over nc.

### Chat

```sh
# Side A
nc -lvp 9001

# Side B
nc <ip> 9001
```

Type and press Enter to send. Ctrl+C to quit.
