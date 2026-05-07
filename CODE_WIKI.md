# iStoreOS-Actions Code Wiki

## 目录

1. [项目概述](#项目概述)
2. [项目结构](#项目结构)
3. [核心模块说明](#核心模块说明)
4. [构建流程](#构建流程)
5. [关键文件详解](#关键文件详解)
6. [依赖关系](#依赖关系)
7. [支持设备列表](#支持设备列表)
8. [开发指南](#开发指南)

---

## 项目概述

### 项目简介

**iStoreOS-Actions** 是一个基于 GitHub Actions 的自动化 iStoreOS 固件构建项目，允许用户通过 GitHub 工作流轻松地为多种 ARM64 架构设备构建自定义的 iStoreOS 24.10.2 固件。

### 项目特性

- 支持多种硬件平台（Amlogic、Rockchip 等）
- 自动化构建流程
- 内置常用插件（FileBrowser、AdGuardHome、Lucky 等）
- 支持静态 IP 和 DHCP 两种网络配置方式
- 可选集成 Docker 支持
- 支持自定义额外软件包

---

## 项目结构

```
/workspace/
├── .github/
│   └── workflows/                # GitHub Actions 工作流
│       ├── St1_Build-Rootfs-release.yml
│       ├── St2_Build-iStoreOS-ib.yml
│       ├── StX3_Build-iStoreOS-src.yml
│       ├── StX_Build-iStoreOS-src.yml
│       └── ye_Build-iStoreOS.yml
├── arm64/                        # ARM64 架构构建工具
│   ├── Makefile                 # 镜像构建 Makefile
│   ├── build24.sh               # 构建脚本
│   └── repositories.conf        # 软件源配置
├── files/                        # 固件文件系统内容
│   ├── etc/
│   │   ├── init.d/
│   │   │   └── pcie_fix
│   │   ├── openclash/core/
│   │   │   └── clash_meta
│   │   ├── opkg/
│   │   │   └── distfeeds.conf   # OPKG 源配置
│   │   ├── uci-defaults/
│   │   │   ├── 98-pcie-fix
│   │   │   └── 99-custom.sh     # 首次启动配置脚本
│   │   ├── banner
│   │   └── rc.local
│   ├── make-openwrt/
│   │   ├── kernel/
│   │   │   └── rk35xx/
│   │   │       └── 6.12.y/
│   │   │           └── rockchip/
│   │   └── u-boot/
│   │       └── rk3399/
│   │           └── emb3531/
│   ├── packages/                # 自定义 IPK 软件包
│   │   ├── luci-app-adguardhome/
│   │   ├── luci-app-amlogic/
│   │   ├── luci-app-filebrowser/
│   │   ├── luci-app-lucky/
│   │   ├── luci-app-openlist2/
│   │   ├── luci-app-ramfree/
│   │   ├── opc-rely/
│   │   └── other-rely/
│   ├── screenshot/
│   │   └── screenshot1.png
│   └── pcie_fix.dts
├── make-openwrt/                # OpenWRT 构建相关文件
│   ├── kernel/
│   │   └── rk35xx/
│   │       └── 6.12.y/
│   │           └── rockchip/
│   └── u-boot/
│       └── rk3399/
│           └── emb3531/
├── shell/                       # Shell 辅助脚本
│   ├── custom-packages.sh       # 自定义软件包配置
│   └── prepare-packages.sh      # 准备软件包脚本
├── .gitignore
├── LICENSE
└── README.md
```

---

## 核心模块说明

### 1. GitHub Actions 工作流模块

项目包含多个 GitHub Actions 工作流文件，实现了固件的分阶段构建：

| 工作流文件 | 功能描述 |
|-----------|---------|
| `St1_Build-Rootfs-release.yml` | 第一阶段：构建通用 RootFS 底包 |
| `St2_Build-iStoreOS-ib.yml` | 第二阶段：打包成设备特定固件 |
| `StX_Build-iStoreOS-src.yml` | 备用：从源码编译 |
| `StX3_Build-iStoreOS-src.yml` | 备用：从源码编译 |
| `ye_Build-iStoreOS.yml` | 备用构建流程 |

### 2. 构建脚本模块

#### build24.sh

- **位置**: [arm64/build24.sh](file:///workspace/arm64/build24.sh)
- **功能**: 主要的固件构建脚本，配置需要包含的软件包和内核模块

#### Makefile

- **位置**: [arm64/Makefile](file:///workspace/arm64/Makefile)
- **功能**: OpenWrt ImageBuilder 的 Makefile，用于构建固件镜像

### 3. 配置文件模块

#### 99-custom.sh

- **位置**: [files/etc/uci-defaults/99-custom.sh](file:///workspace/files/etc/uci-defaults/99-custom.sh)
- **功能**: 首次启动时的配置脚本，负责：
  - 设置时区为 Asia/Shanghai
  - 设置默认语言为简体中文
  - 配置网络（静态 IP 或 DHCP）
  - 配置防火墙规则
  - 配置 SSH 和网页终端访问

#### distfeeds.conf

- **位置**: [files/etc/opkg/distfeeds.conf](file:///workspace/files/etc/opkg/distfeeds.conf)
- **功能**: 配置 OPKG 软件源，使用北大镜像源

### 4. 辅助脚本模块

#### custom-packages.sh

- **位置**: [shell/custom-packages.sh](file:///workspace/shell/custom-packages.sh)
- **功能**: 配置第三方软件包，包含：
  - luci-app-partexp（分区扩容）
  - luci-theme-kucat（酷猫主题）
  - luci-app-advancedplus（进阶设置）
  - luci-app-netspeedtest（网络测速）
  - luci-app-mosdns（MosDNS）
  - luci-app-turboacc（Turbo ACC）
  - 等等

#### prepare-packages.sh

- **位置**: [shell/prepare-packages.sh](file:///workspace/shell/prepare-packages.sh)
- **功能**: 解压缩并整理自定义 IPK 包到构建目录

---

## 构建流程

### 分阶段构建

项目采用两阶段构建流程：

#### 阶段 1：构建 RootFS 底包

使用 `St1_Build-Rootfs-release.yml` 工作流：

1. **输入配置**：
   - 网络配置（静态 IP 或 DHCP）
   - 管理 IP（静态配置时）
   - 网关（静态配置时）
   - 是否集成 Docker

2. **构建步骤**：
   - 下载 iStoreOS ImageBuilder
   - 配置网络设置
   - 安装依赖工具
   - 配置软件包
   - 构建 rootfs.tar.gz
   - 上传到 GitHub Release

#### 阶段 2：打包设备特定固件

使用 `St2_Build-iStoreOS-ib.yml` 工作流：

1. **输入配置**：
   - 设备型号
   - 内核版本
   - RootFS 源
   - 网络配置

2. **构建步骤**：
   - 下载阶段 1 生成的 rootfs.tar.gz
   - 使用 ophub/amlogic-s9xxx-openwrt 打包工具
   - 生成设备特定的 .img.gz 固件
   - 上传到 GitHub Release

---

## 关键文件详解

### 1. St1_Build-Rootfs-release.yml

- **位置**: [.github/workflows/St1_Build-Rootfs-release.yml](file:///workspace/.github/workflows/St1_Build-Rootfs-release.yml)

**关键配置**：
- 使用 Ubuntu 22.04 作为运行环境
- 下载预构建的 ImageBuilder
- 支持自定义网络配置（静态/DHCP）
- 集成 Docker 可选配置
- 输出 rootfs.tar.gz

### 2. St2_Build-iStoreOS-ib.yml

- **位置**: [.github/workflows/St2_Build-iStoreOS-ib.yml](file:///workspace/.github/workflows/St2_Build-iStoreOS-ib.yml)

**关键配置**：
- 使用 ophub/amlogic-s9xxx-openwrt 工具
- 支持 160+ 种设备型号
- 支持多种内核版本（5.10.y - 6.12.y）
- 自动匹配内核

### 3. build24.sh

- **位置**: [arm64/build24.sh](file:///workspace/arm64/build24.sh)

**关键功能**：
- 定义所有要包含的软件包（超过 600+ 项）
- 包含系统基础包、网络工具、驱动、容器支持等
- 配置内核模块
- 集成第三方插件

**主要软件包类别**：
- 系统基础工具（attr, bash, coreutils 等）
- 网络工具（curl, iperf3, nftables 等）
- 文件系统工具（btrfs-progs, e2fsprogs 等）
- 容器支持（containerd, docker, dockerd）
- LuCI 界面及插件
- 驱动模块（网卡、USB、存储等）
- 加密和安全工具

### 4. 99-custom.sh

- **位置**: [files/etc/uci-defaults/99-custom.sh](file:///workspace/files/etc/uci-defaults/99-custom.sh)

**关键功能**：
- 设置时区为 CST-8（亚洲/上海）
- 设置主机名映射（解决 Android TV 联网问题）
- 配置网络（单网口 DHCP / 多网口 WAN+LAN）
- 设置默认语言为 zh_cn
- 配置 SSH 和网页终端
- 更新系统描述信息

**网络配置逻辑**：
- **单网口**：使用 DHCP 模式，从上一级路由获取 IP（NAS/旁路由模式）
- **多网口**：第一个网口作为 WAN（DHCP），其余作为 LAN（静态 192.168.100.1）

---

## 依赖关系

### 外部依赖

| 依赖项 | 用途 | 来源 |
|-------|------|------|
| iStoreOS ImageBuilder | 构建底包 | Releases 下载 |
| ophub/amlogic-s9xxx-openwrt | 设备打包工具 | GitHub Actions 第三方 |
| softprops/action-gh-release | 上传 Release | GitHub Actions 第三方 |
| GitHub Token | 发布权限 | 用户配置 |

### 内部依赖

```
St1_Build-Rootfs-release.yml
  ├─ build24.sh
  │  └─ custom-packages.sh
  │     └─ prepare-packages.sh
  │        └─ files/packages/*.ipk
  └─ 99-custom.sh

St2_Build-iStoreOS-ib.yml
  └─ (依赖 St1 的输出 rootfs.tar.gz)
```

### 软件源

主软件源使用北大镜像站：
- https://mirrors.pku.edu.cn/openwrt/releases/24.10.2/

包含源：
- istoreos_core
- istoreos_base
- istoreos_luci
- istoreos_packages
- istoreos_routing
- istoreos_telephony

---

## 支持设备列表

### Amlogic 芯片

| 芯片型号 | 设备示例 |
|---------|---------|
| A311D | Khadas-VIM3, WXY-OES |
| S922X | Beelink-GT-King, Ugoos-AM6-Plus, ODroid-N2 |
| S905X3 | X96-Max+, HK1-Box, Ugoos-X3 |
| S905X2 | X96Max, KM3-4G, TX5-Max |
| S912 | TX8-Max, TX9-Pro, Phicomm-T1 |
| S905D | MECOOL-KI-Pro, Phicomm-N1 |
| S905X | HG680P, B860H, TX9, X96 |
| S905W | X96-Mini, TX3-Mini |
| S905L/L2/L3/L3a/L3b | 多种运营商盒子 |
| S905 | Beelink-Mini-MX, Sunvell-T95M |

### Rockchip 芯片

| 芯片型号 | 设备示例 |
|---------|---------|
| RK3588 | Radxa-Rock5B/C, OrangePi-5-Plus, HLink-H88K |
| RK3568 | FastRhino-R66S/R68S, Radxa-E25, NanoPi-R5S/R5C |
| RK3566 | Panther-X2, Station-M2 |
| RK3528 | HLink-H28K, Radxa-E20C |
| RK3399 | EAIDK-610, King3399, Emb3531, Firefly-RK3399 |
| RK3328 | BeikeYun, Station-M1 |
| RK3318 | RX3318-Box |

### Allwinner 芯片

| 芯片型号 | 设备示例 |
|---------|---------|
| H6 | Vplus, Tanix-TX6 |

---

## 开发指南

### 如何添加自定义软件包

1. 编辑 [shell/custom-packages.sh](file:///workspace/shell/custom-packages.sh)
2. 取消注释需要的软件包，或添加新的包
3. 如果是外部包，将 IPK 文件放入 [files/packages/](file:///workspace/files/packages/) 对应目录

### 如何修改网络配置

修改 [files/etc/uci-defaults/99-custom.sh](file:///workspace/files/etc/uci-defaults/99-custom.sh)：

```bash
# 静态 IP 配置（第 37-42 行）
uci set network.lan.proto='static'
uci set network.lan.ipaddr='192.168.5.88'  # 修改这里
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='192.168.5.1'  # 修改这里
uci set network.lan.dns='223.5.5.5'
```

### 如何添加新的工作流

1. 在 `.github/workflows/` 目录下创建新的 `.yml` 文件
2. 遵循现有的工作流结构
3. 使用 GitHub Actions 语法定义触发条件和步骤

### 前置条件

- GitHub 仓库
- 配置 `GH_TOKEN` 密钥（Settings → Secrets and variables → Actions）
- 了解 iStoreOS/OpenWrt 构建系统

### 本地测试流程

如需本地测试构建流程：

1. 安装依赖：`sudo apt-get install build-essential libncurses5-dev zstd curl unzip tree`
2. 下载 ImageBuilder
3. 运行 `build24.sh` 脚本
4. 检查输出结果

---

## 内置插件列表

### 已集成插件

| 插件名称 | 功能描述 |
|---------|---------|
| ramfree | 内存释放工具 |
| FileBrowser | 文件管理界面 |
| Lucky | 多功能工具集 |
| AdGuardHome | 广告拦截 DNS 服务器 |
| OpenList2 | 列表管理工具 |
| luci-app-amlogic | Amlogic 设备管理 |
| Docker | 容器引擎（可选） |
| Clash Meta | 代理核心 |

### 可选插件（需手动启用）

见 [shell/custom-packages.sh](file:///workspace/shell/custom-packages.sh) 文件，包括：
- luci-app-partexp（分区扩容）
- luci-app-mosdns（MosDNS）
- luci-app-turboacc（网络加速）
- luci-app-netspeedtest（测速）
- 等等

---

## 固件默认配置

| 配置项 | 默认值 |
|-------|-------|
| 用户名 | root |
| 密码 | 无（首次登录后建议修改） |
| 语言 | 简体中文 |
| 时区 | Asia/Shanghai (CST-8) |
| 单网口 IP | DHCP（从上一级获取） |
| 多网口 LAN IP | 192.168.100.1 |
| 多网口 WAN | DHCP |
| SSH | 所有网口可访问 |
| 网页终端 | 所有网口可访问 |

---

*本 Code Wiki 最后更新时间：2026-05-07*
