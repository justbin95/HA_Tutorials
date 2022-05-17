# n1盒子安装官方Home Assistant Supervised（原HassIO）文字教程

> 本文参考自hassbian论坛 https://bbs.hassbian.com/thread-14469-1-1.html
>
> Supervisor: （中文=管理员）就是以前的HassIO/Hass.io，是用来管理和更新Home Assistant Core，管理操作系统，管理docker（HA和加载项），以及管理前三者之前的API和互动，它自己在docker容器里面，并且管理着其他容器。
>
> Home Assistant Core：这个以前就叫Home Assistant（core=核心）
>
> Home Assistant OS（HAOS）: 以前叫HassOS，是官方为树莓派打造的基于Linux的操作系统，包含了Home Assistant core， Supervisor，也就是完整的全套，可以直接安装于树莓派或者虚拟机，这是官方推荐安装方法。
>
> Home Assistant Supervised: 这个也是全套，跟HAOS的区别是可以装在普通Linux上因此适合更多硬件，N1用的就是这个。安装原理就是手动把docker，Home Assistant Core、Supervisor和其他所有必要组件安装在普通Linux系统上。为了花更多精力提升HA本上而不是debug各种兼容性问题，去年官方大幅减少支持的环境，目前唯一支持的是Debian 11，否则，轻则安装完后显示“不支持的操作系统”，重则无法安装）

本教程特点：非一键安装脚本，使用官方原生源安装，内容干净，支持在线更新

本教程前提：n1已经降级，且打开adb调试，（如果没有请参考 https://bbs.hassbian.com/thread-7659-1-1.html ）及16GB以上高质量品牌u盘，且网络能顺畅访问国际互联网。

## 一、写入Armbian镜像到U盘（Debian11 bullseye）

在Debian的官方论坛帖子（ https://forum.armbian.com/topic/12162-single-armbian-image-for-rk-aml-aw-aarch64-armv8/ ），找到Arm-64的固件并下载。

固件目录：https://users.armbian.com/balbes150/ ，选择n1的arm-64的Bullseye固件。

![](https://github.com/justbin95/HA-Supervised/blob/main/image/iShot2022-05-17_20.47.43.png?raw=true)

下载解压后用BalenaEtcher（官网下载：https://www.balena.io/etcher/ ）将镜像写入U盘，记得提前备份U盘文件。

![](https://github.com/justbin95/HA-Supervised/blob/main/image/iShot2022-05-17_21.02.59.png?raw=true)

写入之后，打开U盘，如果要使用蓝牙功能，把我提供的 `meson-gxl-s905d-phicomm-n1.dtb` 文件，复制到 `/dtb/amlogic/` 覆盖。

然后进入`/extlinux/extlinux.conf` 修改。

如图所示，把`aml s9xxx`下面的第一行换为

```xml
FDT /dtb/amlogic/meson-gxl-s905d-phicomm-n1.dtb
```

并取消注释，最后一行也取消注释，前面rk3399的也全部注释。保存。

![](https://github.com/justbin95/HA-Supervised/blob/main/image/iShot2022-05-17_21.11.08.png?raw=true)

如果嫌麻烦，也可以直接粘贴以下内容进去，覆盖原先内容。（删减了注释掉的内容）

```xml
LABEL Armbian
LINUX /zImage
INITRD /uInitrd

# aml s9xxx
FDT /dtb/amlogic/meson-gxl-s905d-phicomm-n1.dtb
APPEND root=LABEL=ROOTFS rootflags=data=writeback rw console=ttyAML0,115200n8 console=tty0 no_console_suspend consoleblank=0 fsck.fix=yes fsck.repair=yes net.ifnames=0
```

重命名根目录`u-boot-s905x-s912`文件，改成`u-boot.ext`，弹出U盘。

## 二、U盘启动Armbian

给n1插上网线，U盘插入n1靠近hdmi的usb口，注意安卓系统启动的状态下不要插入u盘，否则可能破坏文件结构。

n1 U盘启动教程见 https://bbs.hassbian.com/thread-7659-1-1.html (见Armbian刷写，b部分，跳过a)

U盘首次启动需等待几分钟，然后在路由器后台里找到n1的ip地址，然后win打开命令提示符，mac打开终端，SSH登录。

```shell
ssh root@192.168.x.x #你查询到的ip地址
```

（win10自带OpenSSH，可以直接在CMD里面用SSH。如果没有，则用第三方SSH，比如Xshell、Putty等，我这里用MobaXterm举例）用户名root，密码1234。

![](https://github.com/justbin95/HA-Supervised/blob/main/image/iShot2022-05-17_21.32.41.png?raw=true)

按提示创建用户。因为联网了，时间和时区会自动设置。用date命令看时间，如果时间不自动更新，请关掉防火墙，不然时间不对的话后面会出错。然后重新SSH到你刚创建的用户。

## 三、把Armbian写入自带eMMC（可选）

如果你的U盘还有他用，或者嫌U盘速度太慢，可以选择把系统写入n1内置的eMMC。**（由于n1内置只有8GB存储，装完Home Assistant后只会剩下1GB多的空间，插件装的多了可能会提示空间不足，请自行考虑是否进行此步骤）**

在SSH中运行以下代码

```shell
sudo -i
./install-aml.sh
```

等待数分钟，若显示“Complete Copy OS to eMMC”，则写入成功。接下来poweroff关机，拔电源和U盘后重新插点开机，就可以直接进入Armbian系统了。

## 四、更新软件包、安装必备组件及Docker

分别在SSH输入以下代码并回车（一行一回车）（尽量能顺畅连接国际互联网）。

```shell
sudo -i
apt update && sudo apt upgrade -y && sudo apt autoremove -y
apt --fix-broken install
apt-get install jq curl avahi-daemon apparmor-utils udisks2 libglib2.0-bin network-manager dbus wget -y
curl -fsSL get.docker.com | sh
```

如果在安装过程中有选项需要选择（如下图），直接按回车即可。

![](https://github.com/justbin95/HA-Supervised/blob/main/image/iShot2022-05-17_21.36.41.png?raw=true)

![](https://github.com/justbin95/HA-Supervised/blob/main/image/iShot2022-05-17_21.54.43.png?raw=true)


