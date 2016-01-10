# Build a rootfs for loongson LS1C0300A

##  Buildroot Commands  
```
git clone https://github.com/FreeBlues/buildroot
cd buildroot
cp ./configs/loongson1c_smartloong_defconfig ./.config
make menuconfig
make
```

In `./output/images/`, the rootfs file `rootfs.yaffs2img` will appear.

```
cd ./output/images
chmod 777 ./rootfs.yaffs2img
mv ./rootfs.yaffs2img ./rootfs.yaffs2.img
```

Put the `rootfs.yaffs2.img` to a `tftp` server, for example, the server IP is `192.168.99.209`.

Reboot loongson 1C board, press `spacebar` to enter `PMON`

##  PMON Commands

- Attantion: Be sure to run `mtd_erase /dev/mtd1` first, after that, then you can run  `devcp tftp://192.168.99.209/rootfs.yaffs2.img /dev/mtd1 yaf nw`, otherwise it is very easy to see the bad block in your `NAND` flash.

```
ifup syn0
ifaddr syn0 192.168.99.161
ping 192.168.99.209
mtd_erase /dev/mtd1
devcp tftp://192.168.99.209/rootfs.yaffs2.img /dev/mtd1 yaf nw
set append 'root=/dev/mtdblock1 console=ttyS2,115200 rootfstype=yaffs2'
```

##  PMON Operation Log

```
PMON> ifup syn0
Bus Mode Reg after reset: 0x00020101
Version = 0x1237
MacAddr = 0x0   0x55    0x7b    0xb5    0x7d    0xf7
=====>enter synopGMAC_mac_init:1000
=====>full duplex
=====>100M
PMON> ifaddr syn0 192.168.99.161
bootp=8000b874
PMON> ping 192.168.99.209
PING 192.168.99.209 (192.168.99.209): 56 data bytes
64 bytes from 192.168.99.209: icmp_seq=0 ttl=64 time=1.107 ms
64 bytes from 192.168.99.209: icmp_seq=1 ttl=64 time=0.678 ms

--- 192.168.99.209 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.678/0.880/1.107 ms
PMON> mtd_erase /dev/mtd1
mtd_erase working:
0x07200000
mtd_erase work done!
PMON>
PMON> devcp tftp://192.168.99.209/rootfs.yaffs2.img /dev/mtd1 yaf nw
10813440
60001920PMON>
60001920PMON> set append 'root=/dev/mtdblock1 console=ttyS2,115200 rootfstype=yaffs2'
PMON> reboot
```

##  Loongson LS1C Chipset Infomation

- [Loongson 1C User Manual V1.0](http://www.loongson.cn/uploadfile/cpumanual/Loongson1C_user_manual_en_v1.0.pdf)    
- [Loongson 1C Data Sheet V1.0](http://www.loongson.cn/uploadfile/cpumanual/Loongson1C_datasheet_en_v1.0.pdf)

---

# 为龙芯 LS1C0300A 构建根文件系统

本项目以[基础版本-rootfs]()为基础配置文件, 在这两个基础版本的配置文件基础上进行裁剪得到[板上开发环境-busybox配置文件]()和[板上开发环境-rootfs配置文件](), 我们后续讨论的都是基于这两个开发环境配置文件构建生成的 `rootfs` 镜像.

已经构建好的 `rootfs` 镜像可以到百度网盘[rootfs.yaffs2.img](http://pan.baidu.com/s/1qXt9f7u)下载, 文件大小为 `76,946,496` 字节, 大约 `76M`.

如果想要自己构建, 可以按照下面的操作步骤进行.

##  板上开发环境Busybox 和根文件系统的新增特性:

### busybox 新增了如下内容:

-     dpkg
-     dkpg-deb
-     mkfs.vfat
-     mkfs.ext2
-     mount.exfat
-     mount.exfat-fuse
-     nandwrite

应该把 bash 加进来
应该把 modprobe 加进来

### rootfs 新增了如下内容:
* 程序:
  -     lua
  -     make
  -     tmux
  -     git
  -     curl
  -     tree
  -     htop
  -     fbterm
  -     at
  -     nano
  -     openssh
  -     openssl
  -     tufted
  -     sudo
  -     wget  —busybox版本不太好用, 准备去掉
  -     ipkg-cl     —没搞清楚怎么用, 准备去掉

* 库:
  -     libmpc
  -     libgpm
  -     libmpfr
  -     libreadline

##  构建板上开发环境的 Buildroot 命令

先克隆本项目, 然后分别配置 `busybox` 和 `rootfs`, 最后开始构建, 在我的机器上大概需要接近2小时.
```
git clone https://github.com/FreeBlues/buildroot
cd buildroot
cp ./configs/loongson-config ./.config
make busybox-menuconfig
make menuconfig
make
```
根文件系统镜像 `rootfs.yaffs2img` 会被生成在 `./output/images/` 目录中

```
cd ./output/images
chmod 777 ./rootfs.yaffs2img
mv ./rootfs.yaffs2img ./rootfs.yaffs2.img
```

把生成的 `rootfs.yaffs2.img` 文件放到一个 `tftp` 服务器上, 例如, 这里的服务器 `IP`是 `192.168.99.209`.

重启龙芯 1C 开发板, 按下`空格`键, 进入 `PMON` 界面

##  PMON 命令

- 注意: 一定要先执行 `mtd_erase /dev/mtd1`, 才可以执行 `devcp tftp://192.168.99.209/rootfs.yaffs2.img /dev/mtd1 yaf nw`, 否则很容易在你的 `NAND` 闪存上出现坏块.

```
ifup syn0
ifaddr syn0 192.168.99.161
ping 192.168.99.209
mtd_erase /dev/mtd1
devcp tftp://192.168.99.209/rootfs.yaffs2.img /dev/mtd1 yaf nw
set append 'root=/dev/mtdblock1 console=ttyS2,115200 rootfstype=yaffs2'
```

##  PMON 操作记录

```
PMON> ifup syn0
Bus Mode Reg after reset: 0x00020101
Version = 0x1237
MacAddr = 0x0   0x55    0x7b    0xb5    0x7d    0xf7
=====>enter synopGMAC_mac_init:1000
=====>full duplex
=====>100M
PMON> ifaddr syn0 192.168.99.161
bootp=8000b874
PMON> ping 192.168.99.209
PING 192.168.99.209 (192.168.99.209): 56 data bytes
64 bytes from 192.168.99.209: icmp_seq=0 ttl=64 time=1.107 ms
64 bytes from 192.168.99.209: icmp_seq=1 ttl=64 time=0.678 ms

--- 192.168.99.209 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.678/0.880/1.107 ms
PMON> mtd_erase /dev/mtd1
mtd_erase working:
0x07200000
mtd_erase work done!
PMON>
PMON> devcp tftp://192.168.99.209/rootfs.yaffs2.img /dev/mtd1 yaf nw
10813440
60001920PMON>
60001920PMON> set append 'root=/dev/mtdblock1 console=ttyS2,115200 rootfstype=yaffs2'
PMON> reboot
```

## Loongson LS1C0300A 芯片资料

- [龙芯1C处理器用户手册](http://www.loongson.cn/uploadfile/cpu/1C/Loongson_1C300_user.pdf)    
- [龙芯1C处理器数据书册](http://www.loongson.cn/uploadfile/cpu/1C/Loongson_1C300_data.pdf)

##  感谢

本项目 fork 自 https://github.com/pengphei/buildroot ,使用了该项目提供的配置文件作为基础版本.

---

Buildroot is a simple, efficient and easy-to-use tool to generate embedded
Linux systems through cross-compilation.

The documentation can be found in docs/manual. You can generate a text
document with 'make manual-text' and read output/docs/manual/manual.text.
Online documentation can be found at http://buildroot.org/docs.html

To build and use the buildroot stuff, do the following:

1) run 'make menuconfig'
2) select the target architecture and the packages you wish to compile
3) run 'make'
4) wait while it compiles
5) find the kernel, bootloader, root filesystem, etc. in output/images

You do not need to be root to build or run buildroot.  Have fun!

Buildroot comes with a basic configuration for a number of boards. Run
'make list-defconfigs' to view the list of provided configurations.

Please feed suggestions, bug reports, insults, and bribes back to the
buildroot mailing list: buildroot@buildroot.org
You can also find us on #buildroot on Freenode IRC.
