# 搭建可重复的内核实验环境


实验环境选择使用`x86-64 Linux开发机`和`QEMU虚拟机`来进行搭建。

**实验目录**

```text
kernel-lab/
├── linux/                 # 内核源码
├── build/                 # O= 输出目录
├── rootfs/                # initramfs 根目录
├── scripts/               # build/run/debug 脚本
├── modules/               # 练习模块
├── experiments/           # 每个实验一个目录
└── notes/                 # 源码地图、故障记录、周报
```
