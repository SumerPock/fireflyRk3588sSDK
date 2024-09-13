![Guo](Guo.jpg)
# 编译Linux固件
## 获取SDK

此文件夹用于存储SDK
```
mkdir ~/proj
```
将SDK从共享文件夹拷贝出来
```
cp /mnt/hgfs/ROC-RK3588S-PC/Linux_SDK/rk3588_linux_release_20230114_v1.0.6c_0* ~/proj/rk3588_sdk
```

`注意`
1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上
2. 编译环境请使用 Ubuntu18.04（真机或 docker 容器），如果使用其他版本可能导致编译出错
3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK
4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）
### 安装工具

更新软件及安装repo、git 和 python 软件包
```
sudo apt update
sudo apt install -y repo git python
```
### 初始化仓库
验证MD5码
```
md5sum rk3588_linux_release_20230114_v1.0.6c_0*
```
结果如下
```
c3bcb3f92bd139f72551c89f75d39bfa  rk3588_linux_release_20230114_v1.0.6c_00
ebb658571a645d4af1e2b569709480b7  rk3588_linux_release_20230114_v1.0.6c_01
9761cc324e9f7133500b590c441b0307  rk3588_linux_release_20230114_v1.0.6c_02
7adc9fe2158d7681554dce1def238f49  rk3588_linux_release_20230114_v1.0.6c_03
3d9201e3849b8a523c05920bebe28b39  rk3588_linux_release_20230114_v1.0.6c_04
6faaee006fe60fc9be60a64a01506cb6  rk3588_linux_release_20230114_v1.0.6c_05

```

在用户路径创建/proj/rk3588_sdk文件夹，这里面的-p指的是创建父目录`(如果指定的目录路径中包含的父目录还不存在，-p 选项会自动创建这些父目录)`和避免错误`(如果指定的目录已经存在，使用 -p 选项不会报错。没有 -p 选项时，如果目录已经存在，mkdir 会返回错误)`
```
mkdir -p ~/proj/rk3588_sdk
```
解压 rk3588_linux_release_20230114_v1.0.6c_0* 路径下的拆分文件并将其内容提取到当前目录
```
cat rk3588_linux_release_20230114_v1.0.6c_0* | tar -xv
```
同步仓库`需要良好稳定的互联网`
```
.repo/repo/repo sync -l
```

### 同步代码


- repo/repo/repo 代表工具的路径
- sync 代表同步代码仓库
- -c 仅同步当前的仓库最新提交，而不是整个历史
- --no-tags 不下载标签
`这步最容易出错，并且最容易出问题，解决方法就是删除出问题的目录，然后重新下载`

```
.repo/repo/repo sync -c --no-tags
```

创建下git 目录，否则下一步会一直报错

```
git init
```

- start 创建并切换到一个新的分支
- firefly 新分支的名字。你可以根据需要替换为其他名字
- --all 在所有项目中创建并切换到这个新分支
```
.repo/repo/repo start firefly --all
```

## Linux SDK配置介绍

### 目录介绍
安装下树状目录查看工具
```
sudo apt install tree
```

整个SDK目录结构如下所示
```
eushally@ubuntu:~/proj/rk3588_sdk$ tree -L 1
.
├── app
├── buildroot
├── build.sh -> device/rockchip/common/build.sh
├── device
├── docs
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── firefly-update.sh -> device/rockchip/common/firefly-update.sh
├── kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh
├── prebuilts
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh
├── tools
└── u-boot

10 directories, 6 files
```
### 配置文件介绍

~/proj/rk3588_sdk/device/rockchip/rk3588/ 目录下，有不同板型的配置文件(xxxx.mk)，用于管理 SDK 每个环节的编译配置，相关配置介绍，`目前选择 哪个？？？？？？`

### 分区说明
#### parameter 分区表
#### package-file

## 编译Ubuntu固件
推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整

本教程的编译部分适用于 v1.0.6e 以上 SDK 版本

先查看下当前SDK版本信息
```
readlink -f .repo/manifest.xml 
/home/eushally/proj/rk3588/.repo/manifests/rk3588/rk3588_linux_release_20240517_v1.4.0b.xml</pre>
```

### 准备工作
#### 搭载编译环境
```
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool
sudo apt-get install expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support
sudo apt-get install qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib
sudo apt-get install unzip
sudo apt-get install device-tree-compiler ncurses-dev

```
#### 编译前配置
在 device/rockchip/rk3588/ 目录下，有不同板型的配置文件，选择配置文件

```
./build.sh roc-rk3588s-pc-ubuntu.mk 
processing option: roc-rk3588s-pc-ubuntu.mk
switching to board: /home/eushally/proj/rk3588/device/rockchip/rk3588/roc-rk3588s-pc-ubuntu.mk

```
#### 编译
##### 全自动编译
需要下载跟文件系统，并放在SDK的路径下，同时需要注意高速相机的软件适应的Ubuntu系统为20.04,这里我使用的是`ROC-RK3588S-PC_Ubuntu_v1.4.0b_240905.img`

```
mkdir ubuntu_rootfs
mv ROC-RK3588S-PC_Ubuntu_v1.4.0b_240905.img ubuntu_rootfs/rootfs.img

```
开始编译
```
./build.sh
```
编译完成的固件会保存在`rockdev/pack`目录
##### 部分编译
###### 编译u-boot
`./build.sh uboot`
###### 编译kernel
`./build.sh extboot`
###### 编译recovery
`./build.sh recovery`
###### 下载跟文件系统
将Ubuntu 根文件系统(64位)，放到 SDK 路径下
`mkdir ubuntu_rootfs`
`mv ubuntu-aarch64-rootfs.img ubuntu_rootfs/rootfs.img`
###### 更新各部分镜像链接
更新链接到`rockdev/`目录
`./mkfirmware.sh`
###### 打包固件
生成的固件会保存在`rockdev/pack/`目录
`./build.sh updateimg`



## 针对高速相机SDK的添加与修改
### kernel的修改
#### imx586-240fps.c文件
* 将真大提供的imx586-240fps.c替换到Ubuntu18.04虚拟机SDK中的目录中的imx586.c并重命名
    > 位置：`~/rk3588/kernel/drivers/media/i2c`
    > 属性：`-rw-rw-r-- 1 eushally eushally 52492 Sep  5 02:41 imx586.c`
* 将上一步中移动过来的imx586.c文件权限修改，chmod 664 imx586.c，使其权限与之前
#### rockchip_linux_defconfig文件增加内容
* 在Linux虚拟机SDK目录中找到`rockchip_linux_defconfig`文件然后增加`CONFIG_VIDEO_IMX586=y`
    >位置：`~/rk3588/kernel/arch/arm64/configs`

#### 修改rk3588-roc-pc-cam-8ms1m.dtsi文件
* 找到rk3588-roc-pv-cam-8ms1m.dtsi
    >位置：`~/rk3588/kernel/arch/arm64/boot/dts/rockchip`
    >属性：`-rw-rw-r-- 1 eushally eushally 3599 Sep  5 02:41 rk3588-roc-pc-cam-8ms1m.dtsi`
* 按照imx586-dts.path文件修改rk3588-roc-pv-cam-8ms1m.dtsi中的内容，其中- 代表删除， + 代表添加，@@ -150,7 +184,7 @@ cif_mipi2_in0: endpoint { ，代表多少行
#### 替换rk3588-roc-pc-cam-8ms1m.dtsi文件
* 对于修改rk3588-roc-pc-cam-8ms1m.dtsi文件的另外一种解决方法直接替换，没啥好说的，删除原本的`rk3588-roc-pc-cam-8ms1m.dtsi`,然后将新的换进去
#### 配置Pcle2.0硬盘接口
* 在目录`arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`中修改`roc-rk3588s-pc.dtsi`修改其宏定义`#define M2_SATA_OR_PCIE 0 /1 = SATA , 0 = PCIe /`
#### 关闭log
* 此步骤是否真的需要待考虑，对照附件close_log.patch进行修改
#### 单独编译kernel
* `./build.sh extboot`
#### 分区下载kernel
* 使用瑞芯微下载工具分区下载kernel
### 系统文件更新及替换
#### 系统补丁包
* SSH进入板卡执行`sudo dpkg -i camera_engine_rkaiq_rk3588_arm64.deb`
#### 替换配置文件
* `240fps_imx586_default_default.json` 移动并替换到`/etc/iqfiles/imx586_default_default.json`,检查其文件权限是否与该目录的其他文件一致
* `ubuntu_240fps_rkaiq_3A_server` 替换到`/usr/bin/rkaiq_3A_server`,检查权限与该目录的其他文件是否一致
### 导出设备系统
当用户已经在一台设备上完成工作环境的部署，需要将当前环境完整导出，以批量部署到其它同设备上，可以通过导出设备文件系统来备份当前的开发环境。

导出设备系统分为两步：

1. 在设备上导出 Ubuntu 根文件系统 rootfs；
2. 二次打包完整固件，将 Ubuntu rootfs 与发布固件（这里埋了个雷）的其他分区组合，完成二次打包，生成新的完整固件。
#### 导出rootfs
**以下操作在ARM平台Rk3588板卡上进行**
1. 在设备的Ubuntu环境下，安装fireflydev：
`sudo apt update`
`sudo apt install fireflydev`
2. 安装 fireflydev 后，就能使用 ff_export_rootfs 脚本导出根文件系统
* 建议使用容量较大的移动硬盘(我是直接用的挂载的硬盘，如果直接用系统内部Emmc会报容量不够的错)
* 导出工具会执行 apt clean 等操作以减小文件系统大小（自动执行不用管他）
* 将根文件系统导出，例如导出到 /media/firefly/AC91-C4AE/ 目录（需要等待一定时间）
`ff_export_rootfs /media/firefly/AC91-C4AE/`
* 压缩文件系统，删除不必要的空白空间以减少存储器资源的占用：
`# 有部分客户说导出的 rootfs 大小为 3.3G，可实际只用了 3G，原因是没有对 rootfs 进行压缩`
`e2fsck -p -f Firefly_Ubuntu_18.04.6_rootfs.img`
`resize2fs -M Firefly_Ubuntu_18.04.6_rootfs.img`
#### 二次打包完成固件
**以下操作在X86平台上进行**
1. 安装必要的软件包`sudo apt-get install lib32stdc++6`
2. 下载二次打包工具[百度云（提取码1234）](https://pan.baidu.com/share/init?surl=qiOqutl4WW1OFBpom8w6nA)
3. 解压二次打包工具
`tar -xzf firefly-linux-repack.tgz`
`cd firefly-linux-repack`
    树状目录如下
    firefly-linux-repack
        ├── bin
        │   ├── afptool
        │   └── rkImageMaker
        ├── pack.sh # 打包脚本
        ├── Readme_en.md
        ├── Readme.md
        └── unpack.sh # 解包脚本
4. 解包操作：~~~把官方发布的 Ubuntu 固件~~~将自己编译出来的相机固件拷贝到 firefly-linux-repack 根目录，重命名为 update.img，执行解包脚本 unpack.sh。解包完成后，各分区文件在 output 目录下。
`mv /path/to/ROC-RK3566-PC_Ubuntu18.04-r21156_v1.2.4a_220519.img update.img`
`./unpack.sh`
5. 打包操作：保持当前目录结构，文件名等不变，接入移动硬盘到 PC 机，把前面导出的 Ubuntu rootfs 替换 output/Image/rootfs.img，然后执行打包脚本 pack.sh。
`cp /media/customer/1878-4615/Firefly_Ubuntu_18.04.6_rootfs.img /path/to/firefly-linux-repack/output/Image/rootfs.img`
`./pack.sh`
6. 新的完整固件为当前目录的 new_update.img
**这个步骤很耗费系统空间，操作时注意，系统空间满了之后会报错**


