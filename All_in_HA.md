# 万物皆可Home Assistant

废话不多说，本期教程教大家把常见的智能家居平台设备接入Home Assistant（以下简称 HA），包括米家、Aqara Home、涂鸦，以及其他支持HomeKit的设备。其他平台因为我没相应设备，就烦请大家多搜搜教程啦，推荐一个论坛“[瀚思彼岸](https://bbs.hassbian.com)”。

以下是部分智能平台的接入方法，我这里就具体讲前五种。

| 平台                                          | 设备类型                                                     | 接入方式（集成）                                             | 备注                                                   |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| 米家（被控设备）                              | 灯、开关、空调、风扇、窗帘等电器，温湿度计等环境设备         | [Xiaomi Miot Auto](https://github.com/al-one/hass-xiaomi-miot) | 大部分WiFi设备为本地                                   |
| 米家（触发设备）                              | 无线开关、门磁、人体等触发类蓝牙传感器                       | [Xiaomi Gateway 3](https://github.com/AlexxIT/XiaomiGateway3) | 需要多模网关1/2 且不能使用小米中枢网关（需拔掉）       |
| ZigBee网关（米家/Aqara Home）                 | M1S、M2、P3等网关及其子设备<br />（其中空调功能无法接入）    | [Aqara Gateway](https://github.com/niceboygithub/AqaraGateway) | 需要telnet客户端，M2网关需要拆机                       |
| Aqara Home Wifi 设备<br />及其他 HomeKit 设备 | 妙控开关S1E、FP2，P3的空调功能<br />涂鸦 ZigBee (HomeKit) 网关等 | HomeKit反向接入                                              | 功能会有阉割                                           |
| 涂鸦（Wi-Fi）                                 | 灯、空调、风扇等Wi-Fi设备                                    | [Local Tuya](https://github.com/rospogrigio/localtuya)       |                                                        |
| 涂鸦（ZigBee/BLE）                            | ZigBee、蓝牙网关及其子设备                                   | Tuya（云端）                                                 | 需要开发者平台账号且购买IoT Core连接服务（免费两个月） |
| 小燕                                          | 灯、开关、窗帘、插座、传感器等                               | [homeassistant-terncy-component](https://github.com/rxwen/homeassistant-terncy-component) |                                                        |
| Apple                                         | Apple TV/HomePod                                             | Apple TV                                                     |                                                        |
| 索尼                                          | 电视                                                         | Sony Bravia TV                                               |                                                        |
| 安卓电视/盒子                                 | 支持ADB的安卓电视/盒子                                       | Android TV                                                   |                                                        |

HA的安装我[之前的视频](https://www.bilibili.com/video/BV1DU4y1m74Z)讲过，网上各种设备的安装教程也有很多，我就不再赘述。由于不同人安装的 HA 的版本不同，界面可能会有些许不同，不过基本思路相同，不会影响操作。但为了避免各种奇奇怪怪的问题，建议通过官方渠道安装，并升级到最新版的 HA 。我使用的 core 版本是 2023.3。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303142304.png" alt="20230303142304" style="zoom:50%;" />

本教程均不会影响设备在原生态平台的操作及联动。我也会把图文教程发在B站专栏。

### 本教程需要的环境：

1、安装了 [HACS 商店](https://hacs.xyz/docs/setup/download) 的Home Assistant。

2、流畅的**互联网**环境。

3、SSH/Telnet工具（按需）。Windows 推荐 [MobaXterm](https://mobaxterm.mobatek.net/)，MacOS 推荐 [Royal TSX](https://royalapps.com/ts/mac/features)。

## 一、米家设备

米家设备分为 **Wi-Fi 设备**（大部分电器，部分旧款灯具等）、**蓝牙/蓝牙Mesh设备**（大部分传感器、开关、灯具等）和 **ZigBee 设备**（绿米相关传感器、开关等）。

### 1、Xiaomi Miot Auto

表格第一行 `Xiaomi Miot Auto` 集成，因为它是通过**轮询**来获取设备的状态，所以**不支持实时监听无线开关、人体门窗传感器**之类设备的事件。所以你如果只是想把米家设备接入 HA ，并且桥接到 HomeKit 来控制的话，这个集成也够用了。且大部分 Wi-Fi 设备能本地接入，支持断网控制，蓝牙和 ZigBee 设备则需要**云端**控制。

#### (1) 在 HACS 商店下载集成

点击左侧 HACS ，点击右边 “集成”，选择右下角 “浏览并下载存储库”。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303142425.png" alt="20230303142425" style="zoom:50%;" />

搜索 “Miot Auto”。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303142558.png" alt="20230303142558" style="zoom:50%;" />

进入详情页后点击右下角 “下载”，在选择版本界面直接点击下载，稍等片刻。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303142704.png" alt="20230303142704" style="zoom:50%;" />

下载完之后返回 HACS 首页，会提示等待重启，点击 “前往”，点击 “重新启动”。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303142934.png" alt="20230303142934" style="zoom:50%;" />

#### (2) 添加集成并绑定小米账号

重启完成后，点击左侧 “配置”-“设备与服务”，右下角 “添加集成”。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303143339.png" alt="20230303143339" style="zoom:50%;" />

搜索 “Miot Auto”，点击进入并选择第一个 “账号集成”，下一步。输入小米账号和密码后，其他不用改，点击 “提交”。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303143606.png" alt="20230303143606" style="zoom:50%;" />

#### (3) 筛选你需要的设备

在 “筛选设备” 界面，选择你需要接入的设备，建议使用包含模式。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303144313.png" alt="20230303144313" style="zoom:50%;" />

设备列表中，括号里是 IP 地址的代表的是 Wi-Fi 设备，基本可以本地控制；其他显示型号代码的则是蓝牙或者 ZigBee 设备，只能**云端**接入。点击 “提交” 稍等片刻，出现成功界面，选择一下设备所在的房间，点击 ”完成“。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303144536.png" alt="20230303144536" style="zoom:50%;" />

#### (4) 尽情控制吧

在集成界面点击设备，就可以随心控制啦。

### 2、Xiaomi Gateway 3

如果要对米家**蓝牙/蓝牙Mesh**或者**接入多模的 ZigBee 设备**进行本地控制，或者想要把米家的人体门窗传感器、无线开关等**触发类设备**接入 HA ，就需要用到这个 `Xiaomi Gateway 3` 集成。使用此集成需要一台**小米多模网关**，一代二代都可以。如果你有小米中枢网关，由于它会跟同网络下的多模网关“抢设备”，造成接入 HA 后部分设备状态丢失，显示不可用，所以需要把中枢网关**断电**，虽然损失了跨网关本地自动化和极客版自动化的功能，但是 HA 强大的本地联动和 Node-Red 自动化编辑器完全可以替代它们。

#### (1) 下载集成

在 HACS 商店下载集成 `Xiaomi Gateway 3`，方法同上。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120100149.png" alt="image-20230309120100149" style="zoom:50%;" />

#### (2) 添加集成并绑定小米账号

重启后，在集成页面点击右下角，添加 `Xiaomi Gateway 3` 集成，选择 “Add Mi Cloud Account”。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120421488.png" alt="image-20230309120421488" style="zoom:50%;" />

输入小米账号和密码，点击提交。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120448103.png" alt="image-20230309120448103" style="zoom:50%;" />

#### (3) 添加网关及子设备

再次添加集成，会自动列出支持的设备，选择添加多模网关，点击提交。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120536039.png" alt="image-20230309120536039" style="zoom:50%;" />

稍等片刻，刷新下界面，就能显示出网关下的子设备啦。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120641320.png" alt="image-20230309120641320" style="zoom:50%;" />

如果显示不可用，等待一会或者手动触发一下传感器，让子设备主动上报一次网关就可以了。

## 二、Aqara ZigBee  网关

如果你用的是 Aqara（绿米）的 ZigBee 网关，比如M1S、M2、空调伴侣P3等，可以通过 `Aqara Gateway` 集成来把子设备接入 HA ，无论接的是米家还是 Aqara Home （M2、H1网关需要**拆机**焊接）。不过要注意的是，此集成只是将**网关**及其**子设备**接入 HA ，所以空调伴侣P3的空调功能是无法接入 HA 的（可以使用下面的 HomeKit 反向接入）。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120824141.png" alt="image-20230309120824141" style="zoom:50%;" />

我这里以 Aqara 空调伴侣 P3 为例。**首先要在 App 中查看一下网关目前的固件版本。**

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120913347.png" alt="image-20230309120913347" style="zoom:50%;" />

**然后到[这个仓库](https://github.com/niceboygithub/AqaraM1SM2fw)，自制固件里查看一下目前支持的版本号，如果大于或等于你网关的版本，就可以放心刷机。**

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309120944638.png" alt="image-20230309120944638" style="zoom:50%;" />

#### (1) 把网关切换为米家模式（米家用户忽略此步）

如果你的网关目前为 Aqara Home 模式，需要手动切换到米家模式来打开 Telnet 功能。**注意，此步骤会将网关恢复出厂设置，所以之后需要重新配对子设备。**快速点击网关按钮十次，当听到 “恢复出厂设置成功” 之后，再双击一下按钮，听到提示音，再等待片刻，网关播报 “等待连接中，请打开米家 App”。然后和平常一样把网关接入米家即可。

#### (2) 下载 AqaraGateway 集成

进入 HACS 商店的集成页面，点击右上角三个点，选择 “自定义存储库”，输入集成地址 `niceboygithub/AqaraGateway` ，类别选择集成，点击 ”添加“ 并关闭窗口。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121052693.png" alt="image-20230309121052693" style="zoom:50%;" />

点击自动发现的 ”Aqara Gateway“ 集成（如果没有手动搜索即可），并下载和重启。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121136634.png" alt="image-20230309121136634" style="zoom:50%;" />

#### (3) 获取网关 Token

这时候需要用到前面的 `Xiaomi Gateway 3` 集成，如果你之前已经添加过该集成（没有的话按照上面的步骤添加一次），直接点击你的账号-选项。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121227546.png" alt="image-20230309121227546" style="zoom:50%;" />

在列表中选择你的网关。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121302544.png" alt="image-20230309121302544" style="zoom:50%;" />

提交后就可以看到设备的 IP 和 Token，复制一下。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121404804.png" alt="image-20230309121404804" style="zoom:50%;" />

#### (4) 添加集成并连接网关

右下角搜索并添加 `Aqara Gateway` 集成，输入 IP 和 Token，”Model“ 选择你网关的型号，提交。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121610732.png" alt="image-20230309121610732" style="zoom:50%;" />

稍等片刻，显示成功后，就可以在集成中看到网关设备了，默认就是一个网关的警戒实体。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121640730.png" alt="image-20230309121640730" style="zoom:50%;" />

**如果你只是在米家中使用网关，那现在已经可以用了**，在米家中添加的子设备会自动同步到这个集成下的设备中。如果你要连接 Aqara Home 使用，那还需要下面的步骤。

#### (5) 刷写官改固件

打开 Telnet 工具，我这里使用 Windows 下的 MobaXterm，其他工具操作类似。添加一个 Telnet 会话，Host 处输入网关 IP，点击确认进行连接。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309121937653.png" alt="image-20230309121937653" style="zoom:50%;" />

用户名输入 `admin`，按回车。连接成功后，会显示这样一个界面。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309122536804.png" alt="image-20230309122536804" style="zoom:50%;" />

我们打开 `Aqara Gateway` 的[仓库](https://github.com/niceboygithub/AqaraGateway)，找到你网关型号的相应刷机代码，比如我的P3，一共三行代码，复制一行，粘贴一行，回车 ，重复三次。

M1S:

```bash
cd /tmp && wget -O /tmp/curl "http://master.dl.sourceforge.net/project/mgl03/bin/curl?viasf=1" && chmod a+x /tmp/curl
/tmp/curl -s -k -L -o /tmp/m1s_update.sh https://raw.githubusercontent.com/niceboygithub/AqaraM1SM2fw/main/modified/M1S/m1s_update.sh
chmod a+x /tmp/m1s_update.sh && /tmp/m1s_update.sh
```

P3:

```bash
cd /tmp && wget -O /tmp/curl "http://master.dl.sourceforge.net/project/mgl03/bin/curl?viasf=1" && chmod a+x /tmp/curl
/tmp/curl -s -k -L -o /tmp/p3_update.sh https://raw.githubusercontent.com/niceboygithub/AqaraM1SM2fw/main/modified/P3/p3_update.sh
chmod a+x /tmp/p3_update.sh && /tmp/p3_update.sh
```

显示 “Update Done” 之后，刷入成功。然后输入 `reboot` 重启。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230306195125.png" alt="20230306195125" style="zoom:50%;" />

#### (6) 切换回 Aqara Home 模式

和第 (1) 步一样，再次重置并切换模式。听到网关播报 “等待连接中，请打开 Aqara Home App“ 后，和平常一样把网关接入 Aqara Home。

#### (7) 再次添加 Aqara Gateway 集成

把之前添加的集成删除，会自动发现待连接的网关，如果没有在右下角手动添加，点击提交。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309122800286.png" alt="image-20230309122800286" style="zoom:50%;" />

稍等片刻，刷新一下，就能看到网关及子设备了，我这里添加了一个温湿度传感器。此时就可以进行重命名等操作了。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309122843416.png" alt="image-20230309122843416" style="zoom:50%;" />

## 三、HomeKit反向接入

如果遇到支持 HomeKit 的 Wi-Fi 设备（例如 Aqara 的妙控开关S1E、人在FP2）或者其他平台支持 HomeKit 的网关（例如涂鸦的 ZigBee 网关），比较通用的做法就是先将设备接入 HA 里的 HomeKit 控制器，再通过 HA 里的 HomeKit 桥接器接到 HomeKit，即 HomeKit 反向接入。这种方法适用于**几乎所有支持 HomeKit 的设备**，但是由于许多设备接入 HomeKit 都有阉割，所以也**无法在 HA 里使用这些阉割的功能**，比如功率。

#### (1) 确保设备未连接到任何 HomeKit 家庭

首先，需要确保你的设备**未连接**到 HomeKit，如果有，请在家庭 App 中删除这个设备，如果之后还是无法添加，请重置设备。

#### (2) 添加 HomeKit 控制器 集成

在集成页面，右下角添加 `HomeKit 控制器` 集成。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309122938583.png" alt="image-20230309122938583" style="zoom:50%;" />

会自动搜索并列出局域网中待连接的设备。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309123007876.png" alt="image-20230309123007876" style="zoom:50%;" />

选择你的设备并提交，之后输入设备的配对代码，一般能在设备上或者包装上找到。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309123039664.png" alt="image-20230309123039664" style="zoom:50%;" />

提交后就添加成功了。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309123116055.png" alt="image-20230309123116055" style="zoom:50%;" />

#### (3) 通过桥接器将设备添加到 HomeKit 家庭

如果你还想在家庭 App 中控制设备，就需要把 HA 里的设备再次通过 HomeKit 桥接器接入到 HomeKit 家庭。具体方法可以看我[这期视频](https://www.bilibili.com/video/BV1xS4y1n7Mh/?share_source=copy_web&vd_source=940b9719d196ab3498895d986dab9e95&t=245)喔。

要注意的是，由于 HA **无法桥接无线开关之类的按钮实体**，所以 HA 中的无线开关，包括妙控开关中的无线开关，是**无法桥接到 HomeKit** 的，只能在 HA 里做自动化。

## 四、涂鸦设备

涂鸦设备也分为 Wi-Fi 直连、ZigBee 和 BLE（低功耗蓝牙）设备。

### 1、涂鸦 Wi-Fi 设备

涂鸦 Wi-Fi 设备大部分可以使用 `Local Tuya` 集成来本地接入，支持断网控制。需要涂鸦开发者账号辅助添加，可以**不开通** IoT Core 连接服务（付费，免费试用一个月）。

#### (1) 下载 Local Tuya 集成

在 HACS 商店下载 `Local Tuya` 集成并重启。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309123555278.png" alt="image-20230309123555278" style="zoom:50%;" />

#### (2) 注册登录涂鸦 IoT 平台

进入[涂鸦 IoT 平台网页](https://iot.tuya.com)，如果没有账号就注册一个。登录完成后选择左边的云开发，点击 ”创建云项目“。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309123649806.png" alt="image-20230309123649806" style="zoom:50%;" />

随便输入项目名称，服务行业和开发方式都选择全屋智能，数据中心选择中国。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124050852.png" alt="image-20230309124050852" style="zoom:50%;" />

由于我这边已经创建过了，就用这个 Local 项目来演示。

#### (3) 绑定涂鸦 App

在项目中，点击 ”设备“ 选项卡，选择 ”关联涂鸦APP账号“，点击 ”添加 App 账号“，用涂鸦智能或者智能生活 App 扫描二维码进行绑定。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124141950.png" alt="image-20230309124141950" style="zoom:50%;" />

设备权限这里选择 ”读、写、管理“。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124256202.png" alt="image-20230309124256202" style="zoom:50%;" />

确定之后就能看到账号下绑定的设备了，可以留意一下这里的 `设备ID` ，待会会用到。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124739653.png" alt="image-20230309124739653" style="zoom:50%;" />

#### (4) 添加 Local Tuya 集成并绑定设备

回到 HA，添加 `Local Tuya` 集成，区域选择 ”cn“，其他留空，由于我们没有开通 IoT Core 连接服务，所以**勾选**最下面的复选框，不配置 API 账号。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124827444.png" alt="image-20230309124827444" style="zoom:50%;" />

提交之后，点击集成的选项，选择添加新设备。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124903040.png" alt="image-20230309124903040" style="zoom:50%;" />

此时会自动发现局域网中的涂鸦 Wi-Fi 设备，这里的代码就是 IoT 平台项目中的 `设备ID` 。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309124938682.png" alt="image-20230309124938682" style="zoom:50%;" />

选择要添加的设备，我这里以 GoSund 智能灯泡为例，自定义一个名字。然后复制设备ID。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309125043460.png" alt="image-20230309125043460" style="zoom:50%;" />

回到 IoT 平台，鼠标移到左边的 ”云开发“，选择 ”API 调试“。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309125123484.png" alt="image-20230309125123484" style="zoom:50%;" />

选择 ”通用设备管理“ - ”获取设备信息“，然后粘贴刚刚复制的设备ID，点击 ”发起调用“。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309125250552.png" alt="image-20230309125250552" style="zoom:50%;" />

在右边的响应结果里，就能看到设备的 `local_key` 了。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309125324756.png" alt="image-20230309125324756" style="zoom:50%;" />

复制一下值，粘贴至 HA，提交。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309125919957.png" alt="image-20230309125919957" style="zoom:50%;" />

选择需要添加的实体类型，然后需要指定一下不同数值的功能，比如亮度、色温。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309130121015.png" alt="image-20230309130121015" style="zoom:50%;" />

可以在 Api 调试界面的 ”获取单个设备的状态“ 里，边调整设备状态，边发起调用，右边对应的数值则会改变，便可找到相对应的功能。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309130532841.png" alt="image-20230309130532841" style="zoom:50%;" />

也可以参考[这个链接](https://github.com/rospogrigio/localtuya/wiki/HOWTO-get-a-DPs-dump)里的内容。

提交之后，如果所有实体类型都添加完了，**勾选下方的不再添加实体**，即可完成配置。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/image-20230309130607324.png" alt="image-20230309130607324" style="zoom:50%;" />

如果还要添加其他设备，重复上面步骤即可。

此时就可以本地控制设备了，对照 App 的响应，几乎没有延迟。

### 2、涂鸦 ZigBee/BLE 设备

涂鸦的 ZigBee/BLE 设备，包括之前的 Wi-Fi 设备，都可以用官方的 `Tuya` 集成来进行云端接入，但是需要购买IoT Core 连接服务，15万一年，emm~，虽然能免费试用一个月，还是算了吧。

<img src="https://raw.githubusercontent.com/justbin95/HA_Tutorials/main/image/HA_Allinone/20230303133541.png" alt="20230303133541" style="zoom:50%;" />

其他设备的接入方法，也欢迎大家在评论区提问或解答喔。

之后还会出各个生态平台互相联动的方法，教程制作不易，请多多三连加关注喔。