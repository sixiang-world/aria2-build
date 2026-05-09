# Aria2 Build (Custom)

A customizable Aria2 compilation workflow — build static/dynamic binaries with your own compile-time limits, no extra features.

## Features

- **Customizable compile-time defaults** — `max-connection-per-server`, `min-split-size`, `max-concurrent-downloads`, `split`
- **Multiple platforms** — `linux-x86_64`, `linux-aarch64`, `windows-x86_64`
- **Static or dynamic linking** — OpenSSL-based build
- **Manual trigger** — full control over every build parameter via `workflow_dispatch`
- **Scheduled builds** — auto-rebuild every Monday at UTC 03:00 with latest `master` source

## Compile-Time Defaults vs Runtime Defaults

Aria2 has two layers of defaults:

| Option | Runtime Default | This Workflow's Compile-Time Default |
|--------|----------------|--------------------------------------|
| `--max-connection-per-server` / `-x` | `16` | `16` (patched to allow up to 32) |
| `--min-split-size` / `-k` | `20M` | `1M` |
| `--max-concurrent-downloads` / `-j` | `5` | `5` |
| `--split` / `-s` | `5` | `16` |

The runtime defaults (CLI flags) are always changeable. The compile-time defaults baked into the binary are what this workflow patches.

## Usage

### Manual Build

1. Go to **[Actions](https://github.com/sixiang-world/aria2-build/actions)**
2. Select **Build Aria2 (Custom)** → **Run workflow**
3. Fill in parameters and click **Run workflow**

### Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `aria2_ref` | Aria2 source branch / tag / commit | `master` |
| `max_connection_per_server` | `MAX_CONNECTION_PER_SERVER` | `16` |
| `min_split_size` | `MIN_SPLIT_SIZE` | `1M` |
| `max_concurrent_downloads` | `MAX_CONCURRENT_DOWNLOADS` | `5` |
| `max_split` | `MAX_SPLIT` | `16` |
| `target` | Target platform | `linux-x86_64` |
| `enable_static` | Static linking | `true` |
| `extra_configure_flags` | Extra `./configure` flags | (empty) |

### Download Latest Artifact

After a successful run, download the built binary from the **Artifacts** section of the run.

## Build Locally

```bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get install build-essential autoconf automake autopoint libtool pkg-config \
  libcppunit-dev libssl-dev zlib1g-dev libc-ares-dev libsqlite3-dev libssh2-1-dev \
  libgmp-dev libgcrypt20-dev gettext git ca-certificates

# Clone aria2
git clone https://github.com/aria2/aria2.git
cd aria2

# Patch defaults (example)
sed -i 's/"max-connection-per-server"[^,]*,\s*"\([0-9]\+\)"/"max-connection-per-server"..., "16"/' src/OptionHandlerFactory.cc

# Build
autoreconf -i
./configure --without-gnutls --with-openssl --with-libcares --with-libsqlite3 --enable-libaria2
make -j$(nproc)
```

## License

Aria2 is licensed under [GPLv2](https://github.com/aria2/aria2/blob/master/COPYING). This repository's workflow code is MIT.
