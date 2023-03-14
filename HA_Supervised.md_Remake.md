# Armbian 安装 Home Assistant (Supervised)

本教程由群友 星火燎原 收集整理

## 1.前提条件

### 1.1 OS

Supervised 安装的方式官方只支持Debian，并且不支持Ubuntu, Armbian, Raspberry Pi OS等系统（其实也能用，只是有些不支持的提示，后面会有办法解决，其实不解决也可以，强迫症不想看到不支持的红色标志，可以参考附录解决）

## 安装

更新系统

```shell
apt-get update
```

安装依赖

```
apt-get install \
apparmor \
jq \
wget \
curl \
udisks2 \
libglib2.0-bin \
network-manager \
dbus \
systemd-journal-remote -y
```

安装 Docker CE

```shell
curl -fsSL get.docker.com | sh
```

安装 home assistant

查看 cpu架构

```
lscpu
Architecture:     aarch64
```



下载相应 cpu 架构的包

```shell
wget https://github.com/home-assistant/os-agent/releases/download/1.4.1/os-agent_1.4.1_linux_aarch64.deb
dpkg -i os-agent_1.4.1_linux_aarch64.deb
```

运行安装脚本

```shell
wget https://github.com/home-assistant/supervised-installer/releases/download/1.3.1/homeassistant-supervised.deb
dpkg -i homeassistant-supervised.deb
```

会弹出弹窗选择 qumuarm，开始安装 home assistant,会下载 docker 镜像，此步骤需要科学上网，并且需要一段时间。

可能通过以下命令查看进度

```
watch -n 5 docker ps
```

什么时候出现了很多容器，并且其中有qemuarm-homeassistant这个容器，就可以通过IP:8123访问了。

访问地址

```
http://your.ip.address.here:8123
```

## 安装图形化系统管理工具 cockpit



```shell
apt-get install cockpit cockpit-storaged cockpit-storaged
systemctl start cockpit
systemctl enable cockpit.socket
```

Cockpit安装成功后，您可以使用Web浏览器在以下位置访问它。

>  https://ip-address:9090



默认没有安装管理docker 的插件，想用 cockpit 管理 docekr,需要安装 cockpit docker 插件,以下为通过 linxu 直接下载，也可以通过 windows 下载，通过 winscp 之类的工具传到 armbian 中。

```shell
wget https://github.com/mrevjd/cockpit-docker/releases/download/v2.0.3/cockpit-docker.tar.gz
```

ssh到 armbian或通过cockpit 中的终端工具，进入/usr/share/cockpit 目录

```
cd /usr/share/cockpit
```

解压软件包

```
tar xf cockpit-docker.tar.gz -C .
```

这时间你就能通过 cockpit 管理 docker 了。

## 附录

### 告警处理

当你访问如下地址

```
http://your.ip.address.here:4357
```

会提示

Supervisor:Connected

Supported:Unsupported

Healthy:     Healthy

想让Supported显示Supported可以参考如下方法解决该问题。

步骤：1. 编辑/boot/uEnv.txt，在APPEND=...的最后加上：

```bash
 apparmor=1 security=apparmor systemd.unified_cgroup_hierarchy=false
```

2. 编辑/etc/os-release，把第一行PRETTY_NAME=...改为：

```makefile
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
```

3. 重启系统

注意：如果不想让Healthy显示为Unhealthy 不要在该系统上再启用和 home assistant 不相关的容器，否则会使Healthy显示为Unhealthy。例如你在该系统下安装管理Docker 的 Portainer 后，Healthy就会显示为Unhealthy。

### 修改IP

由于安装network-manager 后，网络配置权限被该服务接管，不再能通过直接编辑配置文件修改 IP 地址了，安装完成后如有修改 IP 的需求，请使用 nmtui 修改或直接登录 cockpit 在网络处直接在线修改。



### 其它玩法

也可以把有线的音箱转换成支持苹果 airplay 的音箱，在 armbian 中安装一个软件shairport-sync，再将音箱的输入线接入到电视盒子上（经测试移动魔百盒 cm311a 自带的耳机输出口不可用，需要购买一个linux 免驱的 usb 转3.5mm 的线，重点是 linux 免驱）再通过这个转换线接入的音箱。

接好线后，输入以下命令是否能识别 usb 转换线，可以看到下面识别了编号为1的 usb audio ttgk audio 的转换线。

```
cat /proc/asound/cards
 0 [U200           ]: axg-sound-card - U200
                      U200
 1 [Audio          ]: USB-Audio - TTGK Audio
                      TTGK Technology Co.,Ltd TTGK Audio at usb-xhci-hcd.3.auto-1, full speed
```

安装shairport-sync服务

```shell
apt get install shairport-sync
systemctl start shairport-sync
systemctl enable shairport-sync
```



修改默认输出设备为 usb to 3.5mm 设备。

通过 aplay 命令查看设备编号，可以看到 usb 是card 1 的设备。

```
aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: U200 [U200], device 0: fe.dai-link-0 (*) []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: U200 [U200], device 1: fe.dai-link-1 (*) []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: U200 [U200], device 2: fe.dai-link-2 (*) []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: Audio [TTGK Audio], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

card 后面的1写入前三个字段，device后面的0写入最后一个，最终成以下内容，保存。

```
vi /etc/asound.conf 
defaults.ctl.card 1
defaults.pcm.card 1
defaults.timer.card 1
defaults.pcm.device 0
```



输入以下命令查看输出设备，有 headset或speaker 的音量条，并且是非0就可以，如果没有场景可以通过上或下键调整音量。

```
alsamixer 
```

![image-20221110103520397](/Users/renlixing/Library/Application Support/typora-user-images/image-20221110103520397.png)

如果默认不是显示的 ttgk 的声卡，可以通过配置将其设置为默认。



