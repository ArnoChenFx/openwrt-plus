# OpenWrt Plus 23.05

### NanoPi R4S/R5S/R5C & X86_64 固件下载:

[Plus version](https://github.com/pmkol/openwrt-plus/releases) | [Lite version](https://github.com/pmkol/openwrt-plus/releases/tag/v23.05.5-lite)

```
【首次登陆】
地址：10.0.0.1（默认）
用户：root
密码：空

【分区挂载】
系统/磁盘管理 将系统盘剩余空间创建新分区
系统/挂载点   启用新分区并挂载至/opt目录
```

---------------

## 固件说明

- 优化系统内核
  - [x] Full cone NAT
  - [x] TCP BBRv3
  - [x] TCP Brutal
  - [x] LLVM-BPF
  - [x] Shortcut-FE
- 使用 OpenWrt+ImmortalWrt 软件源，支持更多插件的在线安装与升级
- 最小化集成常用插件，修复多处上游插件BUG
- Lite版本仅包含 Mihomo, Sing-box, WireGuard, DDNS, SMB, UPnP

  Plus版本包含以下插件：

| ⚓ 服务 | 🗳️ Docker  | 🩺 网络  |
|  :----  |  :----  |  :----  |
| PassWall | Dockerman | 网速测试 |
| Mihomo | Docker | SQM队列管理 |
| MosDNS | Dockerd | UPnP |
| DDNS | Docker-compose | 带宽监控 |
| 硬盘休眠 | | Socat |
| Watchcat | | 访问控制 |
| Aira2 | | IP限速 |
| FRP客户端 | | |
| 网络共享 | | |
| 网络唤醒 | | |
| ZeroTier | | |

自定义预装插件建议fork上游原项目，以免因本项目未及时同步导致编译失败

---------------

## 本地编译环境安装（根据 debian 11 / ubuntu 22）
```shell
sudo apt-get update
sudo apt-get install -y build-essential flex bison g++ gawk gcc-multilib g++-multilib gettext git libfuse-dev libncurses5-dev libssl-dev python3 python3-pip python3-ply python3-distutils python3-pyelftools rsync unzip zlib1g-dev file wget subversion patch upx-ucl autoconf automake curl asciidoc binutils bzip2 lib32gcc-s1 libc6-dev-i386 uglifyjs msmtp texinfo libreadline-dev libglib2.0-dev xmlto libelf-dev libtool autopoint antlr3 gperf ccache swig coreutils haveged scons libpython3-dev jq
```

---------------

### 启用 [Clang/LLVM](https://docs.kernel.org/kbuild/llvm.html) 构建内核
##### 脚本支持使用 Clang/LLVM 构建内核，NanoPi & X86_64 设备将同时启用 LLVM LTO 链接时优化，这会增加编译的时间，但会获得更优的性能
##### 只需在构建固件前执行以下命令即可启用 Clang/LLVM 构建内核与内核模块

```
export KERNEL_CLANG_LTO=y
```

### 启用 [GCC13](https://gcc.gnu.org/gcc-13/)/[GCC14](https://gcc.gnu.org/gcc-14/)/[GCC15](https://gcc.gnu.org/gcc-15/) 工具链编译
##### 只需在构建固件前执行以下命令即可启用 GCC13/GCC14/GCC15 交叉工具链

```
# GCC13
export USE_GCC13=y
```

```
# GCC14
export USE_GCC14=y
```

```
# GCC15
export USE_GCC15=y
```

### 启用 [LTO](https://gcc.gnu.org/onlinedocs/gccint/LTO-Overview.html) 优化
##### 只需在构建固件前执行以下命令即可启用编译器 LTO 优化

```
export ENABLE_LTO=y
```

### 启用 [MOLD](https://github.com/rui314/mold) 现代链接器（需要启用 `USE_GCC13=y` 或 `USE_GCC14=y` 或 `USE_GCC15=y`）
##### 只需在构建固件前执行以下命令即可启用 MOLD 链接，如果使用它建议同时启用 LTO 优化

```
export ENABLE_MOLD=y
```

### 启用 [eBPF](https://docs.kernel.org/bpf/) 支持
##### 只需在构建固件前执行以下命令即可启用 eBPF 支持

```
export ENABLE_BPF=y
```

### 启用 [LRNG](https://github.com/smuellerDD/lrng)
##### 只需在构建固件前执行以下命令即可启用 LRNG 内核随机数支持

```
export ENABLE_LRNG=y
```

### 启用 [Glibc](https://www.gnu.org/software/libc/) 库构建 （实验性）
##### 启用 glibc 库进行构建时，构建的固件将会同时兼容 musl/glibc 的预构建二进制程序，但缺失 `opkg install` 安装源支持
##### 只需在构建固件前执行以下命令即可启用 glibc 构建

```
export ENABLE_GLIBC=y
```

### 启用本地 Kernel Modules 安装源 （For developers）
##### 启用该标志时，将会拷贝全部 target packages 到 rootfs 并替换 openwrt_core 源为本地方式，以供离线 `opkg install kmod-xxx` 安装操作
##### 这会增加固件文件大小（大约 70MB），对项目内核版本、模块、补丁 有修改的需求时，该功能可能会有用
##### 只需在构建固件前执行以下命令即可启用本地 Kernel Modules 安装源

```
export ENABLE_LOCAL_KMOD=y
```

### 启用 [DPDK](https://www.dpdk.org/) 支持
##### DPDK（Data Plane Development Kit）是一个开源工具集，专为加速数据包处理而设计，通过优化的数据平面技术，实现高性能、低延迟的网络应用
##### 只需在构建固件前执行以下命令即可启用 DPDK 工具集支持

```
export ENABLE_DPDK=y
```

### 快速构建（仅限 Github Actions）
##### 脚本会使用 [toolchain](https://github.com/sbwml/toolchain-cache) 缓存代替源码构建，与常规构建相比能节省大约 60 分钟的编译耗时，仅适用于 Github Actions `ubuntu-24.04` 环境
##### 只需在构建固件前执行以下命令即可启用快速构建

```
export BUILD_FAST=y
```

### 构建 Lite 版本
##### 仅包含少量必备第三方插件，接近官方 OpenWrt 固件
##### 只需在构建固件前执行以下命令即可构建 Minimal 版本

```
export MINIMAL_BUILD=y
```

### 更改 LAN IP 地址
##### 自定义默认 LAN IP 地址
##### 只需在构建固件前执行以下命令即可覆盖默认 LAN 地址（默认：10.0.0.1）

```
export LAN=10.0.0.1
```

-----------------

# 基于本仓库进行自定义构建 - 本地编译

#### 如果你有自定义的需求，建议不要变更内核版本号，这样构建出来的固件可以直接使用 `opkg install kmod-xxxx`

### 一、Fork 本仓库到自己 GitHub 存储库

### 二、修改构建脚本文件：`openwrt/build.sh`（使用 Github Actions 构建时无需更改）

将脚本默认 github raw 链接替换为你的 github raw 链接（不带 https://），像这样 `raw.githubusercontent.com/你的用户名/openwrt-plus/master`

```diff
# github actions - automatically retrieve `github raw` links
if [ "$(whoami)" = "runner" ] && [ -n "$GITHUB_REPO" ]; then
    export mirror=raw.githubusercontent.com/$GITHUB_REPO/master
else
-    export mirror=raw.githubusercontent.com/pmkol/openwrt-plus/master
+    export mirror=raw.githubusercontent.com/你的用户名/openwrt-plus/master
fi
```

### 三、在本地 Linux 执行基于你自己仓库的构建脚本，即可编译所需固件

#### nanopi-r4s openwrt-23.05
```shell
# linux-6.6
bash <(curl -sS https://raw.githubusercontent.com/你的用户名/openwrt-plus/master/openwrt/build.sh) rc2 nanopi-r4s
```

#### nanopi-r5s/r5c openwrt-23.05
```shell
# linux-6.6
bash <(curl -sS https://raw.githubusercontent.com/你的用户名/openwrt-plus/master/openwrt/build.sh) rc2 nanopi-r5s
```

#### x86_64 openwrt-23.05
```shell
# linux-6.6
bash <(curl -sS https://raw.githubusercontent.com/你的用户名/openwrt-plus/master/openwrt/build.sh) rc2 x86_64
```

-----------------

# 使用 Github Actions 构建

### 一、Fork 本仓库到自己 GitHub 存储库

### 二、构建固件

- 在存储库名称下，单击（<img src="https://github.com/user-attachments/assets/f1db14da-2dd9-4f10-8e37-d92ef9651912" alt="Actions"> Actions）。
  
- 在左侧边栏中，单击要运行的工作流的名称：**Build releases**。
  
- 在工作流运行的列表上方，单击“**Run workflow**”按钮，选择要构建的设备固件并运行工作流。
  
  ![image](https://github.com/user-attachments/assets/0c2eb064-a130-47b3-a5a3-1e9a9bb6f50d)
