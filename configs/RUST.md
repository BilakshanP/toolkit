## Rust

### Better Releases

```toml
[profile.release]
lto = "fat"
strip = true
opt-level = 3
panic = "abort"
codegen-units = 1
```

### Quick Debug Builds

1. Install dependencies

```sh
sudo dnf install mold clang
cargo install sccache
rustup toolchain install nightly
rustup override set nightly # in project directory
rustup component add rustc-codegen-cranelift-preview --toolchain nightly
```

2. `.cargo/config.toml` (project directory)

```toml
[build]
rustc-wrapper = "sccache"

[unstable]
codegen-backend = true

[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=mold"]
```

3. `Cargo.toml` (project-directory)

```toml
[profile.dev]
opt-level = 0
debug = 0
incremental = true
codegen-units = 256
lto = "off"
panic = "unwind"
split-debuginfo = "unpacked"
codegen-backend = "cranelift"

[profile.dev.build-override]
opt-level = 3
```

4. Verify

```sh
rustc --version       # should say nightly
mold --version
cargo build           # dev build, should be fast
sccache --show-stats  # confirm cache hits after build
```
