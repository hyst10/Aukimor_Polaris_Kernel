# Aukimor Polaris Kernel
111
小米 MIX 2S（polaris / SDM845 / Android 10 / MIUI 12）自编译内核。

**内核版本**：`4.9.186-Aukimor`（基于官方 dipper-q-oss + 88-zstd 社区分支）

## 特性总览

### 优化（内核内置，非模块）
| 类别 | 内容 |
|---|---|
| 压缩/存储 | LZ4 1.10 + lz4armv8 NEON 汇编加速（zram 默认 lz4）、exFAT（arter97 5.8.2）、zstd 1.5.2 编入备用 |
| 内存 | swappiness=100（zram 优先不杀后台）、dirty_background=5 / dirty_ratio=10（提前写回）、watermark_scale_factor=300（kswapd 提前回收窗口） |
| 调度 | schedutil 默认（原 performance 费电）、三段式调频（轻载加速/高载收油，4 个 sysfs 旋钮）、息屏限频（大核 1.2G / 小核 1.32G，自动开关） |
| 网络 | TCP 默认 Westwood、BBR/CUBIC 编入备用 |
| 系统 | HZ=100、PSI=y、deadline IO、COMPAT_VDSO（32 位 app 免陷入）、futex 编译期 cmpxchg、KALLSYMS_ALL |
| 安全 | CC_STACKPROTECTOR_STRONG、无 KSU（预留 FolkPatch/APatch 通道） |
| 防警告 | /proc/config.gz 欺骗（显示官方配置，MIUI 不弹"设备内部出现问题"） |

### 稳定性（24 个 4.9.186→337 官方 stable 补丁）
见 [docs/patches.md](docs/patches.md) 全清单。覆盖 zsmalloc/tun/UFS×5/TCP×4/memcg/swapfile/dwc3/input/clk/random×2/timer×2/fork/genirq/printk/perf。

### 精简
- dtb 只保留 polaris-mp-v2.1（54 个 → 1 个，镜像 23MB → 16MB）
- 关闭 28 项调试/无用项 + 88XXAU（Kali USB wifi）+ CAN（车载总线）
- 零 `=m` 模块（全部内置，AK3 单刷 boot 即完整）

## 构建

GitHub Actions [build.yml](.github/workflows/build.yml) 自动构建（push 触发）：

```
工具链：Aukimor/android-kernel-tools（clang-r428724 + linaro-gcc-4.9 incl. arm32）
命令：  make ARCH=arm64 CC=clang CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi- polaris_user_defconfig
产物：  AK3 zip（刷机包，含 Image.gz-dtb 1 个 dtb）
```

## 刷机

1. Actions → 最新 build run → Artifacts → `polaris-186` 下载 zip
2. 可先过 [FolkPatch](https://github.com/Aukimor/KernelPatch) 补 root（KALLSYMS 已保）
3. TWRP 或 `fastboot flash boot` 刷入

## 实机验证清单

```bash
cat /proc/sys/vm/swappiness              # → 100
cat /sys/block/zram0/comp_algorithm      # → [lz4] deflate zstd
zcat /proc/config.gz | grep CONFIG_PSI   # → # CONFIG_PSI is not set（欺骗生效）
uname -r                                 # → 4.9.186-Aukimor-g<hash>
```

验证通过项：开机 ✅ VPN（加固 app）✅ FolkPatch root ✅ swappiness ✅ zram lz4 ✅ config.gz 欺骗 ✅

## 目录说明

| 路径 | 说明 |
|---|---|
| `packaging/anykernel-polaris.sh` | AK3 打包模板 |
| `packaging/polaris_runtime_config` | config.gz 欺骗用的静态官方配置 |
| `.github/workflows/build.yml` | CI 构建（push 触发） |
| `docs/patches.md` | 24 个 stable 补丁全清单 |
| `arch/arm64/boot/dts/qcom/Makefile` | dtb 精简（只留 polaris-mp-v2.1） |

## 历史

见 [CHANGELOG.md](CHANGELOG.md)。