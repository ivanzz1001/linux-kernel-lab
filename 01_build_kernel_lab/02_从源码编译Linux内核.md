# 从源码编译Linux内核

## 1. 前期准备

这里根据自己编译过程中遇到的相关问题可能需要预先安装如下包或工具:
```bash
# apt install build-essential
# apt install make flex bison
# apt install libncurses5-dev
```
---

## 2. 下载指定版本Linux内核源代码

Linux内核源代码可以到https://mirrors.edge.kernel.org/pub/linux/kernel/下载，这里我们下载当前最新版本：

```bash
# ls
build  busybox  experiments  linux  modules  notes  rootfs  scripts
# cd linux
# wget https://mirrors.edge.kernel.org/pub/linux/kernel/v7.x/linux-7.3.tar.gz
# tar -zxvf linux-7.3.tar.gz
# ls linux-7.3/
arch   certs    CREDITS  Documentation  fs       init      ipc     Kconfig  lib       MAINTAINERS  mm   README  samples  security  tools  virt
block  COPYING  crypto   drivers        include  io_uring  Kbuild  kernel   LICENSES  Makefile     net  rust    scripts  sound     usr
```
---

## 3. 内核的配置
内核的目的在管理硬件与提供系统内核功能，因此你必须要先找到系统硬件，并且规划主机的未来任务，这样才能够编译出适合这部主机的内核。所以，真个内核编译的重要工作就是在 “挑选我们想要的功能”。

### 3.1 硬件环境查看与内核功能要求

```bash
# lscpu
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         45 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  2
  On-line CPU(s) list:   0,1
Vendor ID:               GenuineIntel
  Model name:            11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz
    CPU family:          6
    Model:               140
    Thread(s) per core:  1
    Core(s) per socket:  1
    Socket(s):           2
    Stepping:            1
    BogoMIPS:            5606.40
    Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology tsc_reliable nonstop_tsc cpuid
                          tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single ssbd ibrs ibpb st
                         ibp ibrs_enhanced fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves arat avx512vbm
                         i umip pku ospke avx512_vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg avx512_vpopcntdq rdpid movdiri movdir64b fsrm avx512_vp2intersect md_clear flush_l1d arch_capabilities
Virtualization features: 
  Hypervisor vendor:     VMware
  Virtualization type:   full
Caches (sum of all):     
  L1d:                   96 KiB (2 instances)
  L1i:                   64 KiB (2 instances)
  L2:                    2.5 MiB (2 instances)
  L3:                    24 MiB (2 instances)
NUMA:                    
  NUMA node(s):          1
  NUMA node0 CPU(s):     0,1
Vulnerabilities:         
  Gather data sampling:  Unknown: Dependent on hypervisor status
  Itlb multihit:         KVM: Mitigation: VMX unsupported
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Mmio stale data:       Not affected
  Retbleed:              Not affected
  Spec rstack overflow:  Not affected
  Spec store bypass:     Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:            Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:            Mitigation; Enhanced / Automatic IBRS, IBPB conditional, RSB filling, PBRSB-eIBRS SW sequence
  Srbds:                 Not affected
  Tsx async abort:       Not affected

# dmidecode | grep "Base Board Information"          //主板
Base Board Information

# lspci | grep Ethernet                             //网卡
02:01.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)

# lspci | grep vga
```

### 3.2 第一次构建

这里我们始终以out-of-tree的方式来构建，以免污染Linux源代码目录。

构建步骤概览:

```bash
mkdir -p ../../build
make O=../../build x86_64_defconfig
make O=../../build menuconfig
make O=../../build -j"$(nproc)"
```

#### 3.2.1 指定CPU arch

这里我们编译出来的内核是需要运行在x86上面的，因此这里执行如下命令导出ARCH环境变量:

```bash
# export ARCH=x86
```
> 说明: 如果我们不想使用此方式导出ARCH环境变量，那么我们可以在执行make命令时显示传递ARCH值。例如 make ARCH=x86 menuconfig

#### 3.2.2 配置board config

当我们在执行`make menuconfig`时，其会执行scripts/kconfig/Makefile => arch/$(ARCH)/Makefile, 然后根据一定的规则来自动选择board。因为在Makefile自动推导board类型时会假设我们要编译出的内核是要运行在当前编译机上，而事实上我们可能需要运行在平台上，因此这里我们直接手动指定board类型即可:
```bash
# make O=../../build/ x86_64_defconfig
```
执行完成后会在`../../build`目录生成`.config`配置文件

#### 3.3.3 对x86_64_defconfig进行微调

这一步其实是对上面第2步的菜单进行微调，我们需要内核支持ramdisk驱动，所以需要选中如下配置:

> 注: 在执行menuconfig的时候，根目录下的Makefile文件如果发现有.config文件(参看KCONFIG_CONFIG变量设置)，那么就会加载该配置文件，此时就相当于对第2步的菜单进行微调

```text
# make O=../../build menuconfig
General setup  --->
      ----> [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support

Device Drivers  --->
    [*] Block devices  --->
            <*>   RAM block device support
            (65536) Default RAM disk size (kbytes) 
```
按上述修改之后记得保存。

>**说明**: 关于这里为什么要选择内核支持ramdisk驱动，以后会进行探讨

#### 3.3.4 建议调试配置

建议开启的配置概览：
```text
CONFIG_DEBUG_INFO=y
CONFIG_GDB_SCRIPTS=y
CONFIG_FRAME_POINTER=y
CONFIG_KALLSYMS=y
CONFIG_KALLSYMS_ALL=y
CONFIG_DEBUG_FS=y
CONFIG_TRACE_FS=y
CONFIG_FTRACE=y
CONFIG_FUNCTION_TRACER=y
CONFIG_KPROBES=y
```

下面逐个介绍如何开启:

1. `开启CONFIG_DEBUG_INFO`

   

3. `开启CONFIG_GDB_SCRIPTS`







