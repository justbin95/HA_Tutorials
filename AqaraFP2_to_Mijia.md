# Aqara 人体场景传感器 FP2 联动米家，接入 Home Assistant，实现万物互联

**建议配合[视频教程](https://www.bilibili.com/video/BV1Vo4y1b7BE/)食用！**

最近，Aqara 发布了人体场景传感器 FP2 ，可以对多个目标进行方向、位置、姿态等的跟踪，而且支持把多个区域同步接入到 HomeKit 中，实现多生态互联。但是也有网上也有许多争议，比如它不支持接入米家。不过办法总归是有的嘛，我们可以通过一些手段，把 FP2 和米家、甚至其他平台设备进行联动。那今天我就来给大家分享几种方法吧，本教程的思路同样适用于其他 Aqara Home 的设备，以后我也会出更加通用的教程喔。

阅读本教程前，请先参阅之前的 [“万物皆可 Home Assistant”](https://github.com/justbin95/HA_Tutorials/wiki/All_in_HA)，视频地址：[【万物皆可HA？教你把各平台的智能设备接入 Home Assistant 教程 柒夏的智能家居Plus05】](https://www.bilibili.com/video/BV1fT411h778/?share_source=copy_web&vd_source=940b9719d196ab3498895d986dab9e95)。 我这里就默认你们都看过之前的教程啦。

### 本次会用到以下几种方法：

1. 通过AM模块，在 Aqara Home 和米家中建立一座 “桥”。
2. 把米家设备接入 HomeKit 之后在 Homekit 中直接进行联动。
3. 通过 Aqara 的虚拟设备，在 HomeKit 中进行更复杂的联动。
4. 把 FP2 接入 Home Assistant （以下简称 HA ），进行更加自由的联动。

### 关于 FP2 的联动条件

先来说明一下 FP2 联动条件中 “有人/无人” 和 “进入/离开” 的区别。

首先，进入/离开 是一个**触发**条件，无论之前检测区域里面有没有人，只要其中一个人**进入**或者离开这个区域，就会触发。

而 有人/无人 既是**触发**条件也是**状态**条件，比如当区域内无人时，有人进入，就会触发这个 “有人” 的动作，之后就会**维持**这个 “有人” 的状态，反之亦然，且当区域中一直有人时，其他人的进入离开都**不会**触发 有人/无人 的条件。

然而经过测试发现，截止目前（2023年4月22日），Aqara Home 里暂时无法把 有人/无人 作为状态，这就导致无法实现其中多个区域同时无人时的自动化。

请等待新版本的固件更新。以下关于 Aqara Home 里的联动，暂时都以修复后的正常情况为例。

## 一、AM模块

要想简单快速地联动 Aqara Home 和米家，就需要一个 “AM模块”，它相当于建立两平台之间沟通的 “桥梁”。

### AM模块介绍

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423200951093.png" alt="image-20230423200951093" style="zoom:33%;" />

AM模块是 ZigBee 协议的子设备，所以在米家和 Aqara Home 都需要 ZigBee 网关才能接入。接入 Aqara Home 之后，可以看到它有一个”设置参数“的执行动作，一共有18个选项。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201127794.png" alt="image-20230423201127794" style="zoom:50%;" />

再接入米家后，可以看到它其实是一个六键的无线场景开关，每个按键分别有单击、双击和长按的动作，也就是一共18个触发动作，分别对应 Aqara Home 中的18个执行动作。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201157181.png" alt="image-20230423201157181" style="zoom:50%;" />

简单来说，只要先在 Aqara Home 中设置自动化，给 A-M 模块一个设置参数的执行动作，它就会在米家模拟按了一下无线开关的按键，这时如果在米家也设置了按下按键就控制设备的联动，就能实现 Aqara 联动米家设备的自动化了。

### 联动方法

比如，我家里客厅有米家的灯带和落地灯灯泡，和接在 S1E 妙控开关的吊灯筒灯，现在设置一个客厅有人且亮度低于10lux就开灯，无人就关灯的自动化。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201233224.png" alt="image-20230423201233224" style="zoom:50%;" />

我的客厅在FP2中被划分为了 “客厅过道”、“沙发区” 和 “钢琴区”。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201302184.png" alt="image-20230423201302184" style="zoom:50%;" />

先打开 Aqara Home，新建自动化：当 “FP2-区域检测-客厅过道-有人” 且满足条件 “FP2-光照度低于10lux” 就执行 “AM模块-设置参数-11”，并且把在 Aqara Home 里面需要联动的设备也加上，比如打开我 S1E 上的客厅吊灯。保存自动化。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201407543.png" alt="image-20230423201407543" style="zoom:50%;" />

然后再打开米家，添加一个自动化，当 “AM模块-第四键双击（这里的选项按顺序对应设置参数的1-18）”，就执行 “打开灯带和灯泡”，保存。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201444194.png" alt="image-20230423201444194" style="zoom:50%;" />

这样，一条完整的开灯自动化就设置好了。同样的，如果需要无人一段时间关灯，在联动条件中选择 “FP2-区域无人且超过一定时长-客厅过道-自定义时长” ，如果要多个区域同时无人才关灯，要加上 同时满足 “沙发区域无人” 和 “钢琴区域无人”的状态即可（目前不可用）。这里建议用经常出入的区域来做无人一段时间的触发，比如我的客厅过道。

### 此方法的优缺点

- 优点：适合**没有安装 HA** ，且只需要联动 Aqara 和米家设备的情况。上手简单，联动稳定且为本地联动。**无需**苹果及家庭中枢设备。
- 缺点：两边均需要购买 ZigBee 网关【米家：推荐多模网关1/2，AqaraHome：推荐M1S（2021款及以前）】，加上 AM 模块，金钱开销较大。且一共只能设置18种联动。

## 二、接入 HomeKit 联动

如果你按照我之前的教程把米家设备接入了 HA 且桥接到了 HomeKit ，那么最方便的方法就是直接把 FP2 接入到 HomeKit，在 Aqara Home 中设置的检测区域也会单独生成人体感应器实体到 HomeKit ，包括光照传感器（注意，添加到 HomeKit 后会比实际区域数多一个传感器实体，这个是**整个** FP2 检测范围的实体）。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201556402.png" alt="image-20230423201556402" style="zoom:50%;" />

然后直接在家庭 App 中建立自动化即可。此方法需要一台 iPhone/iPad 设备及家庭中枢（HomePod 或者 AppleTV）。

### 联动方法

比如还是上面的客厅有人开灯，添加自动化，“感应器检测到变化-FP2客厅过道-检测到有人” 就执行 开启客厅的灯，非常简单。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201635045.png" alt="image-20230423201635045" style="zoom:50%;" />

如果要在光照暗的情况下执行，则需要用到快捷指令。先选择 “客厅过道FP2-有人”，划到最下面选择 “转换为快捷指令”，然后添加一个 “如果” 的操作，“输入” 选择 FP2 的光照传感器，“条件” 选择 “小于”，并且输入相应的光照值，然后把 “设置场景和配件” 拖到 “如果” 下面，选择要操作的设备，比如开客厅灯，最后删除 “否则”，点击下一步即可完成。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201726779.png" alt="image-20230423201726779" style="zoom:50%;" />

HomeKit 的自动化无法实现无人一段时间关灯，所以这里就演示无人就关灯。如果你只要一个区域内无人就关灯，直接设置自动化，检测到某个区域无人，就执行关灯。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201832218.png" alt="image-20230423201832218" style="zoom:50%;" />

如果要多个区域，会有点绕，添加自动化 “FP2客厅过道无人时” 转为快捷指令，如果 “FP2沙发 检测到有人 为 否” 且如果 “FP2钢琴 检测到有人 为 否”，则关客厅的灯。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201859792.png" alt="image-20230423201859792" style="zoom:50%;" />

### 此方法的优缺点

优点：操作简单直观，且均为本地化联动。

缺点：HomeKit 中快捷指令的自动化**响应较慢**，且无法实现 “有人/无人一段时间” 的联动条件。（对比HA联动相应速度）

## 三、通过虚拟设备在 HomeKit 中联动

前面讲到 FP2 在 HomeKit 中无法实现 “有人/无人一段时间” 的条件，那这里还有一种方法，就是在 Aqara Home 里做自动化条件的触发，并通过虚拟设备来把这个触发及状态同步给 HomeKit 。此方法也需要一台 iPhone/iPad 设备及家庭中枢（HomePod 或者 AppleTV）。

### 虚拟设备介绍

Aqara 的虚拟设备可以分为开关、插座和灯三种类型，需要绑定 Aqara Home 里的 ZigBee 网关。它能同时接入 Aqara Home 和 HomeKit 并保持同步开关状态，所以也就成了 Aqara Home 和 HomeKit（甚至米家）设备沟通的 “桥梁”。由于 HomeKit 理论上是不允许虚拟设备存在的，所以需要找 Aqara 的服务商来添加虚拟设备。

这里有两种虚拟设备的玩法。

### 联动方法1

把虚拟设备的开关状态看作场景的触发（比如 开：有人，关：无人一段时间）。这个方法主要适合 FP2 、场景开关用来控制整个灯光场景。

还是以之前客厅有人且光照度低开灯，无人关灯为例。

在 Aqara Home 中新建自动化，当 “FP2-区域检测-客厅过道-有人” 且满足条件 “FP2-光照度低于10lux”，执行 “虚拟设备-开”。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423201955948.png" alt="image-20230423201955948" style="zoom:50%;" />

再新建一个自动化，当 “FP2-区域无人且超过一定时长-客厅过道-自定义时间” 且同时满足 “沙发区域无人” 和 “钢琴区域无人”，执行 “虚拟设备-关”。（目前不可用）

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202045534.png" alt="image-20230423202045534" style="zoom:50%;" />

为了方便管理，我们把这个虚拟设备重命名为 “客厅虚拟存在”，并单独放到 “虚拟设备” 的房间内。

在家庭App中，找到这个虚拟设备，也重命名一下，更改一下房间。然后添加自动化 “配件受控制时-客厅虚拟存在-打开”，就执行开灯，关灯同理。这样，两条自动化就设置好了。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202119310.png" alt="image-20230423202119310" style="zoom:50%;" /><img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202143095.png" alt="image-20230423202143095" style="zoom:50%;" />

### 联动方法2

把虚拟设备与 HomeKit 中的插座、灯的开关状态同步。然后在 Aqara Home 中做自动化，避免了在 HomeKit 中无法使用同一无线开关控制灯具的开和关（快捷指令的方法延迟太大）。具体方法大家应该能举一反三吧，如果有需要，我会在之后全平台自动化的教程中详细讲。

### 此方法的优缺点

优点：相比 HomeKit 自动化的孱弱，使用虚拟设备联动可以充分利用 Aqara Home 里面的联动条件，比如 “有人/无人一段时间”。且联动也均为本地。

缺点：需要一个接入 Aqara Home 的网关，且要找方法添加虚拟设备。

## 四、把 FP2 接入 Home Assistant

要把 FP2 玩出更多花样，就是把它接入 Home Assistant。目前的方法是 HomeKit 反向接入。

具体接入方法可以看我的上期教程，[【万物皆可HA？教你把各平台的智能设备接入 Home Assistant 教程 柒夏的智能家居Plus05】 【精准空降到 11:04】](https://www.bilibili.com/video/BV1fT411h778/?share_source=copy_web&vd_source=940b9719d196ab3498895d986dab9e95&t=664) 。注意，需要先将 FP2 配网，把它从 HomeKit 家庭中**删除**后再进行反向接入，如果 HA 中搜索不到或者无法添加，重新拔插 FP2 即可。

接入 HA 之后，会看到所有设置的区域都同步了进来，还包括光照传感器，但是名字还是默认的，得手动经过相应区域辨别一下并且重命名，同样的，多出来的一个是总的检测范围。此联动方法需要把需要联动的灯具、设备都接入 HA 。

### HA 自动化介绍

点击 “配置” - “场景自动化” - “创建自动化”，可以看到这里有 “触发条件”，“环境条件” 和 “动作”。和米家自动化2.0类似，在触发条件里满足**任意一条**时，且同时满足**所有环境（状态）条件**，就会执行自动化。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202238320.png" alt="image-20230423202238320" style="zoom:50%;" />

### 联动方法

还是以之前两个开关灯自动化举例。添加一个触发条件，状态，实体选择相应的检测区域，比如客厅过道，下面的 “变为” 选择 “Detected”（检测到有人）。如果要设置多个区域，直接在上面添加实体即可，这里各实体是 “或” 的关系，即只要有一个区域有人就执行。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202334825.png" alt="image-20230423202334825" style="zoom:50%;" />

然后在环境条件（状态）里，选择 “设备” 或者 “数字型状态”，选择 “FP2 光照度” - “小于10”。动作里选择需要打开的灯即可。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202448219.png" alt="image-20230423202448219" style="zoom:50%;" />

再添加一个关灯的自动化，触发条件仍旧选择状态，实体选择客厅过道，变为选择 “Clear” （无人）。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202530396.png" alt="image-20230423202530396" style="zoom:50%;" />

下面自定义无人的时长，我这里演示就用2秒。如果要检测多个区域同时无人，在环境条件里再加上相应区域的 “Clear” 状态。再添加关灯的动作即可。

<img src="https://justbin-picgo.oss-cn-hangzhou.aliyuncs.com/img/FP2_to_Mijia/image-20230423202617030.png" alt="image-20230423202617030" style="zoom:50%;" />

### 此方法的优缺点

优点：可以更丰富地联动更多设备，可玩性高，且均为本地联动。**不需要**任何苹果设备（包括 iPhone/iPad 、家庭中枢等）

缺点：我还没想到。可能比较折腾？

好啦，这就是本期教程的全部内容啦，如有问题或者建议欢迎在评论区讨论喔，也可以加入我的交流群交流。那我们下期再见。
