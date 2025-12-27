# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指导。

**工作语言：中文**

## 项目概述

ImmortalWrt 是 OpenWrt 的一个分支，是一个面向网络设备的嵌入式 Linux 发行版。这是一个完整的构建系统，可以交叉编译 Linux 内核、工具链和软件包，生成路由器和嵌入式设备的固件镜像。

当前分支：`openwrt-24.10`

### Orange Pi R2S 支持

本仓库正在添加对 Orange Pi R2S（基于 RISC-V KY X1 SoC）的支持：
- Target 目录：`target/linux/ky/`
- 内核版本：6.6.119
- 设备定义：`target/linux/ky/image/riscv64.mk`

## 构建命令

### 初始化设置
```bash
./scripts/feeds update -a    # 下载所有软件包 feeds
./scripts/feeds install -a   # 安装软件包符号链接
make menuconfig              # 配置目标、软件包和选项
```

### 构建
```bash
make                         # 完整构建（工具链 + 内核 + 软件包 + 镜像）
make -j$(nproc)              # 并行构建
make V=s                     # 详细构建（显示命令）
make V=s 2>&1 | tee build.log  # 构建并保存日志
```

### 部分构建
```bash
make prepare                 # 仅构建工具链
make target/linux/compile    # 仅构建内核
make package/name/compile    # 构建单个软件包
make package/name/clean      # 清理单个软件包
make package/index           # 重建软件包索引
```

### 清理
```bash
make clean                   # 删除构建产物
make targetclean             # clean + 删除工具链
make dirclean                # 深度清理（保留下载）
make distclean               # 完全清理，包括下载和配置
```

### 配置
```bash
make menuconfig              # 交互式配置（ncurses）
make defconfig               # 应用默认配置
make oldconfig               # 更改后更新配置
make kernel_menuconfig       # 配置内核选项
./scripts/diffconfig.sh      # 显示与默认配置的差异
```

### 软件包 Feeds
```bash
./scripts/feeds update -a       # 更新所有 feeds
./scripts/feeds install <pkg>   # 安装特定软件包
./scripts/feeds search <term>   # 搜索软件包
./scripts/feeds list            # 列出已安装的软件包
```

## 架构

### 构建流程
1. `tools/` - 主机构建工具（cmake、autoconf 等）
2. `toolchain/` - 交叉编译器（gcc、binutils、musl/glibc）
3. `target/linux/` - 所选平台的内核
4. `package/` - 所有用户空间软件包
5. 镜像生成 → `bin/targets/<target>/<subtarget>/`

### 关键目录
- `target/linux/<platform>/` - 每个设备的内核配置、补丁、设备树
- `package/` - 软件包 Makefile（每个子目录是一个软件包）
- `package/feeds/` - 外部 feed 软件包的符号链接
- `include/*.mk` - 构建系统宏和规则
- `scripts/` - 构建工具（feeds、元数据、打补丁）
- `staging_dir/` - 构建输出（主机工具、工具链、目标根）
- `build_dir/` - 软件包构建目录
- `bin/` - 最终固件镜像和软件包
- `dl/` - 下载的源码压缩包（缓存）

### 软件包 Makefile 结构
软件包使用声明式 Makefile 格式：
```makefile
include $(TOPDIR)/rules.mk
PKG_NAME:=example
PKG_VERSION:=1.0
include $(INCLUDE_DIR)/package.mk

define Package/example
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=Example package
  DEPENDS:=+libfoo
endef

define Build/Compile
  $(MAKE) -C $(PKG_BUILD_DIR)
endef

define Package/example/install
  $(INSTALL_BIN) $(PKG_BUILD_DIR)/example $(1)/usr/bin/
endef

$(eval $(call BuildPackage,example))
```

### Target 结构
`target/linux/<name>/` 中的每个目标平台包含：
- `Makefile` - Target 定义
- `config-*` - 内核配置片段
- `image/` - 镜像构建规则
- `patches-*` - 按版本的内核补丁
- `files/` - 覆盖到根文件系统的文件
- `dts/` - 设备树源码

## 外部 Feeds

在 `feeds.conf.default` 中定义：
- `packages` - ImmortalWrt 社区软件包
- `luci` - Web 界面
- `routing` - 网状路由协议
- `telephony` - VoIP 软件包

## 常见调试

```bash
make V=s package/foo/compile  # 查看确切的编译命令
make package/foo/{clean,compile} V=s  # 清理重建
ls build_dir/target-*/foo-*/   # 检查软件包构建目录
```

如果并行构建失败：`make -j1 V=s` 查看实际错误。

## ImmortalWrt 特性

- `package/emortal/` - ImmortalWrt 特有软件包（autocore、default-settings）
- 默认 IP：192.168.1.1，用户：root，无密码
- 针对中国用户的优化和额外设备支持
