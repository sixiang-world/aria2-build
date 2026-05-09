# Aria2 Build (Custom)

两个 Aria2 编译 workflow，基于 [DoTheBetter/aria2_build](https://github.com/DoTheBetter/aria2_build) 的完整魔改方案。

## 两个 Workflow 对比

| | `build.yml` | `worker.yml` |
|--|--|--|
| 架构 | Ubuntu 原生编译 | musl (Linux) / MinGW 交叉编译 (Windows) |
| 静态链接 | 可选 | 默认 |
| 补丁方式 | `git am` | `git am` |
| 目标平台 | linux-x86_64, linux-aarch64, windows-x86_64 | linux-x86_64-musl, linux-aarch64-musl, windows-x86_64 |
| Windows 特性 | 基础功能（无 sqlite3/ssh2 等） | 完整功能（含 libxml2） |

## 魔改内容（DoTheBetter 6 Patch）

| Patch | 内容 |
|-------|------|
| `0001` | 默认路径改为当前目录（`~/.netrc` → `./.netrc` 等） |
| `0002` | **`max-connection-per-server` 上限 16→无限制（-1），默认值 1→16** |
| `0003` | 下载速度过慢或连接关闭时自动重试 |
| `0004` | MinGW 环境下支持 `--daemon` |
| `0005` | 新增 `--retry-on-400/403/406/unknown` 选项 |
| `0006` | Windows 新版本号识别（Win8+ / Server 2012+） |

### 编译后默认值变化

| 选项 | 原版默认值 | 魔改后 |
|------|-----------|--------|
| `--max-connection-per-server` | `1`（上限 16） | `16`（**上限无限制**） |
| `--max-concurrent-downloads` | `5` | `16` |
| `--min-split-size` | `20M`（最小 1M） | `1M`（最小 1K） |
| `--split` | `5` | `128` |
| `--continue` | `false` | `true` |
| `--connect-timeout` | `60s` | `30s` |
| `--retry-wait` | `0s` | `1s` |
| `--piece-length` | 最小 1M | 最小 1K |

## 使用方法

### workflow_dispatch（手动触发）

**build.yml（灵活自定义）**

1. 进入 [Actions](https://github.com/sixiang-world/aria2-build/actions) → **Build Aria2 (Custom)**
2. 填写参数（`target`、`aria2_ref` 等）
3. 点击 **Run workflow**

**worker.yml（DoTheBetter 完整方案）**

1. 进入 [Actions](https://github.com/sixiang-world/aria2-build/actions) → **Build Aria2 (Worker)**
2. 选择目标平台（`linux-x86_64-musl` / `linux-aarch64-musl` / `windows-x86_64`）
3. 点击 **Run workflow**

### schedule（定时自动构建）

- `build.yml` 每周一 UTC 03:00 自动用最新 master 源码构建
- `worker.yml` 不含定时任务（按需手动触发）

### 参数说明

#### build.yml

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `target` | `linux-x86_64` | 目标平台 |
| `aria2_ref` | `master` | Aria2 源码分支/tag/commit |
| `enable_static` | `true` | 静态编译 |
| `extra_configure_flags` | (空) | 附加 configure 参数 |

#### worker.yml

| 参数 | 说明 |
|------|------|
| `target` | `linux-x86_64-musl` / `linux-aarch64-musl` / `windows-x86_64` |
| `aria2_ref` | Aria2 源码分支/tag/commit |

## 构建产物

构建完成后在 **Artifacts** 下载：

```
aria2-{target}-{commit}.tar.gz
```

解压后得到单个可执行文件 `aria2c`（Linux）或 `aria2c.exe`（Windows）。

## 本地构建

```bash
git clone https://github.com/sixiang-world/aria2-build.git
cd aria2-build

# 应用补丁
cd aria2-src
for p in ../.github/patches/*.patch; do git am --3way "$p"; done

# Linux 原生编译
./configure --with-openssl --with-libcares --with-libsqlite3 --with-libssh2 --enable-libaria2
make -j$(nproc)

# Windows MinGW 交叉编译
./configure --host=x86_64-w64-mingw32 --prefix=/usr/x86_64-w64-mingw32 \
  --enable-static --with-openssl --with-libxml2 ARIA2_STATIC=yes
make -j$(nproc)
```

## 致谢

本项目的魔改方案基于以下项目和作者：

- **[DoTheBetter/aria2_build](https://github.com/DoTheBetter/aria2_build)** — 核心魔改补丁（6 个 patch）
  - `myfreeer` — 所有 patch 的原始作者
  - Issue #1 libxml2 修复由 `lemonsn` 贡献
- **[P3TERX/Aria2-Pro-Core](https://github.com/P3TERX/Aria2-Pro-Core)**
- **[myfreeer/aria2-build-msys2](https://github.com/myfreeer/aria2-build-msys2)**
- **[abcfy2/aria2-static-build](https://github.com/abcfy2/aria2-static-build)**
- **[q3aql/aria2-static-builds](https://git.q3aql.dev/q3aql/aria2-static-builds)**
- **[aria2/aria2](https://github.com/aria2/aria2)** — Aria2 主项目

## License

Aria2 遵循 [GPLv2](https://github.com/aria2/aria2/blob/master/COPYING)。本仓库 workflow 代码遵循 MIT。
