# immortalwrt-artifacts

ImmortalWrt **25.12** x86-64 自编译产物仓库。

**源码与编译流程不在本仓库**，这里只放编译输出。

## 为什么单独开一个仓库

- 固件 + 母包 + 包仓库加起来 **几百 MB**
- GitHub 硬限：**单文件超 100 MB 直接拒 push**，仓库推荐 **< 1 GB**
- 所以产物全部走 **Releases**（单文件上限 **2 GB**），本仓库的 git 里只留索引

## 产物在哪

**Releases** 里每个 tag = 一次完整编译。

tag 命名：`v<源码版本>-x86-64`，例如 `v25.12.2-x86-64`

## 每次 Release 包含什么

| 文件名 | 是什么 | 怎么用 |
|---|---|---|
| `*squashfs-combined-efi.img.gz` | 完整固件（UEFI 启动） | 解压后 `dd` 到 U 盘，或直写硬盘 |
| `immortalwrt-imagebuilder-*.tar.zst` | ImageBuilder 母包，内含编好的包仓库 | 解压后 `make image PACKAGES="..."` 二次组装 |
| `packages-x86_64-*.tar.zst` | 单独打出的 `.apk` 包仓库 | 解开后当本地 apk 源用 |
| `sha256sums` | 校验和 | 见下 |

## 校验

```sh
sha256sum -c sha256sums
```

## 组装固件（母包用法）

```sh
tar --zstd -xf immortalwrt-imagebuilder-25.12.2-x86-64.Linux-x86_64.tar.zst
cd immortalwrt-imagebuilder-25.12.2-x86-64.Linux-x86_64
make image PROFILE=generic PACKAGES="luci luci-app-xxx kmod-xxx"
# 产物在 bin/targets/x86/64/
```

## 一个必须注意的点

**kmod 与内核配置哈希绑定。**

本仓库固件里的包带 `D:kernel=6.12.94~<32位哈希>-r1` 依赖。这个哈希由内核配置算出，**改了任何内核选项它就会变**。

后果：

- 用本仓库固件 → 只能用**同一 tag** 下的 kmod
- 换成官方 ImmortalWrt 固件 → 官方 kmod 全部装不上（哈希对不上）
- 自己重新编内核 → 之前下的 kmod 全部失效

## 相关仓库

| 仓库 | 用途 |
|---|---|
| [xiealon/IMMORTALWRT](https://github.com/xiealon/IMMORTALWRT) | 源码、config、workflow |
| [xiealon/CloudRunFilesBuilder](https://github.com/xiealon/CloudRunFilesBuilder) | 第三方 apk（honk、clashoo、taskplan 等） |

## License

编译产物遵循 ImmortalWrt / OpenWrt 各自的上游许可（GPL-2.0 等）。
