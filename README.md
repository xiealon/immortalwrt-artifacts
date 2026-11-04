# immortalwrt-artifacts

ImmortalWrt **25.12** x86-64 自编译产物仓库。

**源码与编译流程不在本仓库**，这里只放编译输出。

## 为什么单独开一个仓库

- 固件 + 母包 + 包仓库加起来 **几百 MB**
- GitHub 硬限：**单文件超 100 MB 直接拒 push**，仓库推荐 **< 1 GB**
- 所以产物全部走 **Releases**（单文件上限 **2 GB**），本仓库的 git 里只留索引

## tag 命名规则

固件、母包、包仓库**各占一个 tag，互不覆盖**。

| tag | 里面是什么 |
|---|---|
| `v25.12.2-fw-x86-64-r2` | 固件 `*squashfs-combined-efi.img.gz` + `sha256sums` |
| `v25.12.2-ib-x86-64-r2` | ImageBuilder 母包 `*.tar.zst` + `sha256sums` |
| `v25.12.2-pkg-x86-64-r2` | 包仓库 `packages-x86_64-*.tar.zst` + `sha256sums` |

**为什么分开**

- 三类体积差一个数量级，单独重传某一种时不用动另外两种
- 下载时只拉自己需要的那一份
- **版本在前、类型在后**（`fw` / `ib` / `pkg`），同一版本的三个 tag 在列表里挨在一起，一眼能看出是同一批产出

**同版本重编**（比如改了 config）不新开 tag，直接删旧 asset 重传，并在 release 说明里记 build 时间与源码 commit。

## 各类产物怎么用

### 固件 —— `*-fw-x86-64`

```sh
gunzip immortalwrt-25.12.2-x86_64-generic-squashfs-combined-efi.img.gz
dd if=immortalwrt-25.12.2-x86_64-generic-squashfs-combined-efi.img of=/dev/sdX bs=4M conv=fsync
```

写 U 盘或直写硬盘都行，UEFI 启动。

### 母包 —— `*-ib-x86-64`

ImageBuilder，内含**编好的包仓库**，可二次组装固件而不用重新编译。

```sh
tar --zstd -xf immortalwrt-imagebuilder-25.12.2-x86-64.Linux-x86_64.tar.zst
cd immortalwrt-imagebuilder-25.12.2-x86-64.Linux-x86_64
make image PROFILE=generic PACKAGES="luci luci-app-xxx kmod-xxx"
# 产物在 bin/targets/x86/64/
```

### 包仓库 —— `*-pkg-x86-64`

单独的 `.apk` 包仓库，可以当本地 apk 源挂上，也可以只挑几个装。

```sh
mkdir -p /www/apk-repo
tar --zstd -xf packages-x86_64-25.12.2.tar.zst -C /www/apk-repo
# 然后 /etc/apk/repositories 加一行 http://<LAN IP>/apk-repo
```

## 校验

每个 tag 都带 `sha256sums`：

```sh
sha256sum -c sha256sums
```

## 一个必须注意的点

**kmod 与内核配置哈希绑定。**

本仓库固件里的包带 `D:kernel=6.12.94~<32位哈希>-r1` 依赖。这个哈希由内核配置算出，**改了任何内核选项它就会变**。

后果：

- 用本仓库固件 → 只能用**同一版本**（`v25.12.2-*`）的 kmod
- 换成官方 ImmortalWrt 固件 → 官方 kmod 全部装不上（哈希对不上）
- 自己重新编内核 → 之前下的 kmod 全部失效

所以 `-fw` 和 `-pkg` 两个 tag **必须是同一次编译的产出**，不能混搭不同版本。

## 相关仓库

| 仓库 | 用途 |
|---|---|
| [xiealon/IMMORTALWRT](https://github.com/xiealon/IMMORTALWRT) | 源码、config、workflow |
| [xiealon/CloudRunFilesBuilder](https://github.com/xiealon/CloudRunFilesBuilder) | 第三方 apk（honk、clashoo、taskplan 等） |

## License

- **本仓库分发的编译产物**（固件 / 母包 / 包仓库）遵循 ImmortalWrt 与 OpenWrt 上游各自的许可，主体为 **GPL-2.0**。全文见仓库根目录 [`LICENSE`](LICENSE)。
- **打包、校验与文档整理**由 xiealon 完成，`Copyright (c) 2026-11-05 xiealon`。
- 本仓库不对上游代码附加任何额外限制；上游许可优先于本文档表述。

> GPL-2.0 与 GPL-3.0 **不兼容**。ImmortalWrt 的 `COPYING` 标的是 `GPL-2.0`（不带 `-or-later`），
> 因此本仓库产物不能按 GPL-3.0 再分发。
