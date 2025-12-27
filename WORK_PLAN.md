# Orange Pi R2S 支持工作计划

本文档记录为 ImmortalWrt (openwrt-24.10) 添加 Orange Pi R2S 支持的工作进度。

## 项目信息

- **源仓库**：`/data/openwrt-xunlong`（OpenWrt 24.10 + KY X1 支持）
- **目标仓库**：`/data/r2s`（ImmortalWrt openwrt-24.10 分支）
- **目标设备**：Orange Pi R2S（RISC-V KY X1 SoC）
- **内核版本**：6.6.119

## 工作清单

### 阶段一：文件准备

- [ ] 1.1 拷贝 `target/linux/ky/` 目录到目标仓库
  - [ ] Makefile
  - [ ] patches-6.6/ （14 个有效补丁文件）
  - [ ] image/ （Makefile、riscv64.mk、bootscript）
  - [ ] riscv64/ （target.mk、config-6.6、base-files/）
  - [ ] files/firmware/esos.elf

- [ ] 1.2 创建备份目录并备份大补丁
  - [ ] 创建 `/data/r2s/patches-backup/` 目录
  - [ ] 移动 `0001-ky-x1-support.patch` 到备份目录（不完整，仅存档）
  - [ ] 拷贝 `0001-part_1.patch` 到备份目录（保留原始副本用于比对）
  - [ ] 拷贝 `0001-part_2.patch` 到备份目录（保留原始副本用于比对）

- [ ] 1.3 拷贝 defconfig 文件
  - [ ] 创建 `defconfigs/` 目录
  - [ ] 拷贝 `opir2s_defconfig`

### 阶段二：环境准备

- [ ] 2.1 执行 `./scripts/feeds update -a`
- [ ] 2.2 执行 `./scripts/feeds install -a`
- [ ] 2.3 执行 `cp defconfigs/opir2s_defconfig .config`
- [ ] 2.4 执行 `make defconfig`

### 阶段三：补丁修复（核心工作）

- [ ] 3.1 执行 `make V=s` 进行首次构建
- [ ] 3.2 识别并记录所有补丁冲突
- [ ] 3.3 逐个修复补丁文件

#### 补丁文件清单（patches-6.6/）

| 序号 | 文件名 | 大小 | 状态 | 备注 |
|------|--------|------|------|------|
| ~~1~~ | ~~0001-ky-x1-support.patch~~ | ~~67M~~ | 排除 | ~~不完整的合并补丁，不使用~~ |
| 1 | 0001-part_1.patch | 45M | [ ] | KY X1 支持第1部分（使用） |
| 2 | 0001-part_2.patch | 95M | [ ] | KY X1 支持第2部分（使用） |
| 3 | 0002-mtd-spi-nor-add-fmsh-support.patch | 1.8K | [ ] | FMSH SPI NOR flash |
| 4 | 0003-riscv-add-ky-x1-tcm-register.patch | 933B | [ ] | RISC-V TCM 寄存器 |
| 5 | 0004-Update-drivers-net-ethernet-ky-x1-emac.c.patch | 1.5K | [ ] | EMAC 网络驱动 |
| 6 | 0005-drm-ky-fix-driver-name.patch | 685B | [ ] | DRM 驱动名称 |
| 7 | 0006-Fix-arch-riscv-Kconfig.socs.patch | 1.2K | [ ] | Kconfig SOCs |
| 8 | 0007-delete-unuse-dts.patch | 537K | [ ] | 删除未用设备树 |
| 9 | 0008-Update-arch-riscv-boot-dts-ky-x1_orangepi-rv2.dts.patch | 2.2K | [ ] | Orange Pi RV2 DTS |
| 10 | 0009-Add-dt-overlay-support.patch | 3.2K | [ ] | 设备树覆盖层 |
| 11 | 0010-Add-dt-overlay-for-opirv2.patch | 5.4K | [ ] | RV2 覆盖层 |
| 12 | 0011-Update-for-40pin.patch | 2.4K | [ ] | 40pin 引脚 |
| 13 | 0012-Add-r2s-support.patch | 22K | [ ] | Orange Pi R2S 支持 |
| 14 | 0013-net-ethernet-adding-RTL8125-driver.patch | 1.1M | [ ] | RTL8125 驱动 |

### 阶段四：验证

- [ ] 4.1 使用全新内核代码重新应用补丁
- [ ] 4.2 确认所有补丁正确应用
- [ ] 4.3 记录最终修复情况

## 重要说明：0001 补丁处理

### 背景

`0001-ky-x1-support.patch`（67M）是之前同事处理的合并补丁，它合并了 `0001-part_1.patch`（45M）和 `0001-part_2.patch`（95M），但**严重不完整**。

### 处理方案

**方案一：使用原始分割补丁（推荐）**
- 删除或移动 `0001-ky-x1-support.patch`
- 直接应用 `0001-part_1.patch` 和 `0001-part_2.patch` 并修复
- 优点：原始补丁完整，无需恢复缺失内容

**方案二：修复合并补丁**
- 移动 `0001-part_1.patch` 和 `0001-part_2.patch` 到其他位置
- 应用 `0001-ky-x1-support.patch`
- 从原始补丁恢复缺失内容，补充到 `0001-ky-x1-support.patch`
- 缺点：需要复杂的内容比对和恢复工作

### 选择：方案一

**理由**：
1. 原始补丁（part_1 + part_2 = 140M）内容完整
2. 合并补丁（67M）明显缺失大量内容（约 73M）
3. 直接使用原始补丁更可靠，避免遗漏
4. 减少人工比对和恢复的工作量和错误风险

**执行步骤**：
1. 拷贝时排除 `0001-ky-x1-support.patch`，或拷贝后将其移动到备份目录
2. 保留并使用 `0001-part_1.patch` 和 `0001-part_2.patch`
3. 按顺序应用：part_1 → part_2 → 其他补丁

**备份策略**：
- 创建备份目录：`/data/r2s/patches-backup/`
- 将 `0001-ky-x1-support.patch` 移动到备份目录（不完整，仅存档）
- 将 `0001-part_1.patch` 和 `0001-part_2.patch` **同时拷贝**到备份目录
- 原因：这两个文件较大（共 140M），改动很多，备份后方便在修复过程中不断比对和参考原始内容

---

## 补丁冲突常见原因

1. **上游已合并**：OpenWrt 或内核已经自己实现了补丁中的内容
2. **代码变更**：上下文代码已发生变化，导致补丁无法定位
3. **文件移动**：目标文件已被重命名或移动
4. **冲突修改**：同一位置有不同的修改

## 补丁修复策略

1. 对于已合并的补丁：删除该补丁或相关 hunk
2. 对于上下文变化：更新补丁的上下文行
3. 对于文件移动：更新补丁的文件路径
4. 对于冲突修改：分析实际需求，手动合并

## 修复日志

（在此记录每个补丁的修复过程和结果）

---

## 后续优化：大补丁拆分计划

### 背景

`0001-part_1.patch`（45M）和 `0001-part_2.patch`（95M）两个补丁非常大，不利于维护和调试。

### 目标

将大补丁按功能模块拆分成多个小文件，便于：
1. 单独修复和调试
2. 清晰了解每个补丁的作用
3. 方便后续内核版本升级时的适配

### 拆分方案（待实施）

按目录/功能模块拆分，建议的拆分结构：

| 序号 | 补丁名称 | 涵盖内容 | 预计大小 |
|------|----------|----------|----------|
| 0001 | arch-riscv-core.patch | arch/riscv/ 核心架构代码 | - |
| 0002 | arch-riscv-dts.patch | arch/riscv/boot/dts/ 设备树 | - |
| 0003 | drivers-clk-ky.patch | drivers/clk/ 时钟驱动 | - |
| 0004 | drivers-gpio-ky.patch | drivers/gpio/ GPIO 驱动 | - |
| 0005 | drivers-mmc-ky.patch | drivers/mmc/ MMC/SD 驱动 | - |
| 0006 | drivers-net-ky.patch | drivers/net/ 网络驱动 | - |
| 0007 | drivers-gpu-ky.patch | drivers/gpu/ GPU 驱动 | - |
| 0008 | drivers-usb-ky.patch | drivers/usb/ USB 驱动 | - |
| 0009 | drivers-misc-ky.patch | 其他驱动 | - |
| 0010 | sound-ky.patch | sound/ 音频驱动 | - |

### 拆分步骤

1. 首先完成当前补丁的冲突修复，确保能正常构建
2. 分析补丁内容，按目录统计各部分大小
3. 使用 `filterdiff` 或脚本按路径提取补丁片段
4. 验证拆分后的补丁顺序应用能正确构建
5. 更新补丁文件清单

### 工具

```bash
# 查看补丁涉及的目录
grep "^diff --git" 0001-part_1.patch | cut -d' ' -f3 | cut -d'/' -f1-2 | sort | uniq -c | sort -rn

# 按目录提取补丁
filterdiff -p1 -i 'drivers/clk/*' 0001-part_1.patch > drivers-clk-ky.patch
```

---

*文档创建时间：2025-12-27*
*最后更新：2025-12-27*
