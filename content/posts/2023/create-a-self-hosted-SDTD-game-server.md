---
title: "搭建自托管的七日杀服务器端"
description: "在 Linux 服务器上搭建七日杀游戏服务器端"
date: 2023-06-13T11:09:03+08:00
lastmod: 2023-06-13T11:09:03+08:00
categories: ["教程", "游戏"]
tags: ["七日杀", "SDTD", "7 Days To Die", "LinuxGSM"]

weight:
draft: false
---

## 前情提要

之前和朋友玩了一段时间的七日杀, 一直是我创建游戏, 朋友加入我的游戏. 虽也是联机, 但一直苦于家用带宽上传不稳定, 必须我在线, 朋友才可以玩等痛点. 这几天发现可以自己搭建七日杀服务器, 搭建后体验也不错, 很方便.

## 环境准备

### 服务器配置

我的配置: `CPU 8c` `内存 16G` `硬盘 40G` `系统 Ubuntu 22.04 LTS`

两人同时在线游戏的情况下 CPU 占用率大约 20%. 内存持续缓慢增加, 可能跟游戏内物品和地图渲染有关, 游戏内进度 7 日的情况下占用了 22%. 各位可以以此作为参考. 低配服务器要加 SWAP. 硬盘硬性需求 20G.

七日杀要求服务器系统最低版本如下

| 系统   | 版本      |
| ------ | --------- |
| Ubuntu | 18.04 LTS |
| Debian | 10        |
| CentOS | 8         |

本文以下内容均以 `Ubuntu 22.04 LTS` 系统为演示, 其它系统命令如有也会一并附带.

### 网络配置

如果服务器搭建在中国大陆区域, 需要配置代理来下载 Steam 和七日杀游戏本体内容. 需要下载约 15G 的内容.

在海外服务器使用 `tinyproxy` 搭建 HTTP 代理服务器, 假设地址为 `http://43.155.xxx.xxx:25000`

> 搭建 HTTP 代理服务器一定要配置 IP 鉴权, 不然很容易被扫出来滥用.

也可以直接从部分服务商购买 HTTP 代理, 或采用 Clash 等方案, 此处按下不表.

编辑 `/etc/profile` 文件

```bash
nano /etc/profile
```

在末尾添加如下内容并保存

```bash
http_proxy=http://43.155.xxx.xxx:25000
https_proxy=$http_proxy

export http_proxy https_proxy
```

应用环境

```bash
source /etc/profile
```

此时在命令行可以输出代理即可

![](https://imyrs.net/u/i/img/202306131221056.png)

其他系统查找类似设置全局代理的环境变量配置即可

## 部署

### 安装依赖

#### Ubuntu =< 20.04

```bash
sudo dpkg --add-architecture i386; sudo apt update; sudo apt install curl wget file tar bzip2 gzip unzip bsdmainutils python3 util-linux ca-certificates binutils bc jq tmux netcat lib32gcc1 lib32stdc++6 libsdl2-2.0-0:i386 steamcmd telnet expect libxml2-utils
```

#### Ubuntu => 20.10

```bash
sudo dpkg --add-architecture i386; sudo apt update; sudo apt install curl wget file tar bzip2 gzip unzip bsdmainutils python3 util-linux ca-certificates binutils bc jq tmux netcat lib32gcc-s1 lib32stdc++6 libsdl2-2.0-0:i386 steamcmd telnet expect libxml2-utils
```

#### Debian =< 10

```bash
sudo dpkg --add-architecture i386; sudo apt update; sudo apt install curl wget file tar bzip2 gzip unzip bsdmainutils python3 util-linux ca-certificates binutils bc jq tmux netcat lib32gcc1 lib32stdc++6 lib32z1 telnet expect libxml2-utils
```

#### Debian => 11

```bash
sudo dpkg --add-architecture i386; sudo apt update; sudo apt install curl wget file tar bzip2 gzip unzip bsdmainutils python3 util-linux ca-certificates binutils bc jq tmux netcat lib32gcc-s1 lib32stdc++6 lib32z1 telnet expect libxml2-utils
```

#### CentOS

```bash
yum install -y epel-release
yum install -y curl wget tar bzip2 gzip unzip python3 binutils bc jq tmux glibc.i686 libstdc++ libstdc++.i686 telnet expect libxml2
```

### 使用非 root 用户以继续

1. 创建新的用户, 在部分系统可能需要填写一些信息.
   ```bash
   adduser sdtdserver
   ```

2. 如果创建用户时没有提示创建密码, 则单独创建
   ```bash
   passwd sdtdserver
   ```

3. 切换到新的用户
   ```bash
   su - sdtdserver
   ```

### 安装 LinuxGSM

>  LinuxGSM 是一个安装和管理 Linux 端游戏服务器的软件

```bash
wget -O linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh sdtdserver
```

### 安装 Steam 和七日杀本体

```bash
./sdtdserver install
```

安装过程中可能会有一些选择, 一般根据提示按回车就可以了.

下载过程可能比较长 而且刷新很慢, 需要耐心等待.

### `./sdtdserver` 常用功能解释

```
./sdtdserver  [功能]/[简写]
具体功能如下
[功能]		[简写]	| [说明]
start		st   	| 启动服务器
stop		sp   	| 关闭服务器
restart		r    	| 重启服务器
monitor		m    	| 检查服务器状态并在崩溃时重新启动
details		dt   	| 显示服务器信息
skeleton	sk   	| 创建骨架目录
update-lgsm	ul   	| 检查并应用任何LinuxGSM更新
update		u    	| 检查并应用任何服务器更新
force-update	fu   	| 绕过检查应用服务器更新
validate	v    	| 使用streamcmd验证服务器文件
check-update	cu   	| 检查gameserver更新是否可用
backup		b    	| 创建服务器的备份档案
console		c    	| 访问服务器控制台
debug		d    	| 直接在终端中启动服务器
mods-install	mi   	| 查看并安装可用的模组/插件
mods-remove	mr   	| 查看并删除已安装的模组/插件
mods-update	mu   	| 更新已安装的模组/插件
install		i    	| 安装服务器
auto-install  	ai   	| 在没有提示的情况下安装服务器
```

## 修改配置

### 常用重要文件路径

以下路径都以 `sdtdserver` 用户的相对路径记录

```
~/serverfiles/sdtdserver.xml		# 七日杀游戏配置文件
~/.local/share/7DaysToDie/Saves		# 七日杀游戏存档保存目录
~/serverfiles/Data/Worlds		# 自带地图文件夹
```

### 修改服务器配置

```bash
nano ~/serverfiles/sdtdserver.xml
```

配置文件的中文翻译如下, 按需修改即可. `ServerName` 等字段值如果有空格可能导致启动失败, 自行调试即可.

```xml
<!-- 常规服务器设置 -->

<!-- 服务器列表显示设置 -->
<property name="ServerName"						value="A 7 Days to Die server"/>		<!-- 显示在服务器列表中的名称。 -->
<property name="ServerDescription"				value="A 7 Days to Die server"/>		<!-- 服务器描述信息，显示在服务器列表的旁边。 -->
<property name="ServerWebsiteURL"				value=""/>								<!-- 服务器的网站URL将在服务器浏览器中显示为可单击的链接。 -->
<property name="ServerPassword"					value=""/>								<!-- 设置进入服务器的密码。 -->
<property name="ServerLoginConfirmationText"	value="" />								<!-- 如果设置，用户将在加入服务器期间看到该消息，并且必须在继续之前进行确认。要对此窗口进行更复杂的更改，可以在XUi中更改“serverjoinrulesdialog”窗口。 -->
<property name="Region"							value="NorthAmericaEast" />				<!-- 设置服务器所在区域。Values的值为： NorthAmericaEast, NorthAmericaWest, CentralAmerica, SouthAmerica, Europe, Russia, Asia, MiddleEast, Africa, Oceania -->
<property name="Language"						value="English" />						<!-- 设置服务器中玩家主要使用的语言。Values的值应为语言的英文名称 -->

<!-- 网络设置 -->
<property name="ServerPort"						value="26900"/>				<!-- 服务器端口设置。请将端口保持在26900到26905或27015到27020的范围内。-->
<property name="ServerVisibility"				value="0"/>					<!-- 服务器公开性设置。 2 = 公开, 1 = 只显示给朋友, 0 = 不显示.  注意：因为你不是服务器的朋友，所以在第一个人链接前不可以设置为1，否则无法进入服务器 -->
<property name="ServerDisabledNetworkProtocols"	value="SteamNetworking"/>	<!-- 不应使用的网络协议。用逗号分隔。value的值为：LiteNetLib、SteamNetworking。如果用户和服务器之间没有NAT路由器，或者端口转发设置正确，则专用服务器应禁用蒸汽网络  -->
<property name="ServerMaxWorldTransferSpeedKiBs" value="512"/>				<!-- 读取地图或上传保存的速度（kB/s） -->

<!-- 服务器人数设置 -->
<property name="ServerMaxPlayerCount"			value="5"/>					<!-- 服务器最大同时在线人数 -->
<property name="ServerReservedSlots"			value="0"/>					<!-- 服务器满人时预留多少个有权限人进入服务器 -->
<property name="ServerReservedSlotsPermission"	value="100"/>				<!-- 权限大于多少时，才允许使用“ServerReservedSlots”命令进入服务器 -->
<property name="ServerAdminSlots"				value="0"/>					<!-- 即使服务器已达到服务器最大连接人数，允许设置连接的管理员个数 -->
<property name="ServerAdminSlotsPermission"		value="0"/>					<!-- 管理员使用上面的命令需要的权限级别，和第三个是不同的权限 -->

<!-- 管理界面设置 -->
<property name="ControlPanelEnabled"			value="false"/>				<!-- 是否开启网页控制台界面 -->
<property name="ControlPanelPort"				value="8080"/>				<!-- 网页控制台的端口 -->
<property name="ControlPanelPassword"			value="123"/>				<!-- 登入网页控制台的密码 -->

<property name="TelnetEnabled"					value="false"/>				<!-- 是否开启telnet -->
<property name="TelnetPort"						value="8081"/>				<!-- telnet端口 -->
<property name="TelnetPassword"					value=""/>					<!-- 登入telnet的密码 -->
<property name="TelnetFailedLoginLimit"			value="10"/>				<!-- 多少次密码错误后禁止登入 -->
<property name="TelnetFailedLoginsBlocktime"	value="10"/>				<!-- 禁止登入的时间，以秒为单位 -->

<property name="TerminalWindowEnabled"			value="true"/>				<!-- 是否显示输出服务器日志和命令输入的终端窗口 -->

<!-- 服务器文件和文件夹设置 -->
<property name="AdminFileName"					value="serveradmin.xml"/>		<!-- 服务器管理员文件的名字，位置在"SaveGameFolder" 规定的位置-->
<!-- <property name="UserDataFolder"			value="absolute path" /> -->	<!-- 使用此选项可以覆盖服务器存储所有生成数据的位置，包括RWG生成的世界。不要忘记取消对条目的注释！ -->
<!--<property name="SaveGameFolder"				value="" /> -->					<!-- 设置游戏保存路径。不要忘记取消对条目的注释！ -->

<!-- 其他技术设置 -->
<property name="EACEnabled"						value="false"/>				<!-- 是否开启EAC反作弊 -->
<property name="HideCommandExecutionLog"		value="0"/>					<!-- 隐藏命令执行的日志记录。 0 = 显示一切, 1 = 仅在Telnet/ControlPanel上隐藏, 2 = 在远程控制台隐藏 , 3 = 全部隐藏 -->
<property name="MaxUncoveredMapChunksPerPlayer"	value="131072"/>			<!-- 覆盖每个玩家可以在游戏地图上发现的块数。每个播放器的最大地图文件大小限制为（x*512字节），未覆盖区域为（x*256平方米）。默认值131072意味着在任何时候最多可以覆盖32 km^2 -->
<property name="PersistentPlayerProfiles"		value="false" />			<!-- 如果禁用，玩家可以加入任何选定的个人资料。如果为true，则将使用最后一个连接的配置文件连接 -->



<!-- 游戏设置 -->

<!-- 地图 -->
<!-- RWG 为 随机地图  -->
<property name="GameWorld"						value="Navezgane"/>			<!-- 使用“RWG(随机地图)”（请参阅下面的WorldGenSeed和WorldGenSize选项）或Worlds文件夹中已有的任何世界名称（目前附带有“Navezgane”、“PREGEN01”和…） -->
<property name="WorldGenSeed"					value="asdf"/>				<!-- 使用种子生成一个RWG(随机地图)。如果已存在具有结果名称的世界，则只需加载它-->
<property name="WorldGenSize"					value="6144"/>				<!-- 控制所创建 RWG（随机地图） 世界的宽度和高度。它还与WorldGenSeed结合使用，以创建内部RWG种子，因此即使使用相同的WorldGenSeed，也会创建唯一的地图名称。2048到16384之间必须是2048的倍数，数值越大则生成/下载/加载大型地图会越长时间-->
<property name="GameName"						value="My Game"/>			<!-- 设置游戏名称。无论你想要什么游戏名称。这会影响保存游戏名称以及在世界上放置装饰（树等）时使用的种子。如果创建RWG（随机世界）生成世界，则它不会控制世界的常规布局 -->
<property name="GameMode"						value="GameModeSurvival"/>	<!-- 设置游戏模式生存 -->

<!-- 游戏难度设置 -->
<property name="GameDifficulty"					value="3"/>					<!-- 0 到 5, 0=简单, 5=最难 -->
<property name="BlockDamagePlayer"				value="100" />				<!-- 玩家对方块的伤害有多大（占总数的百分比） -->
<property name="BlockDamageAI"					value="100" />				<!-- AI（僵尸）对方块的损坏程度（占总数的百分比） -->
<property name="BlockDamageAIBM"				value="100" />				<!-- AI（僵尸）在血月对方块造成多少伤害（占总数的百分比） -->
<property name="XPMultiplier"					value="200" />				<!-- XP增益乘数（整数百分比） -->
<property name="PlayerSafeZoneLevel"			value="5" />				<!-- 如果玩家低于或等于此等级，他将在生存时创建一个安全区域（无僵尸） -->
<property name="PlayerSafeZoneHours"			value="5" />				<!-- 此安全区存在的世界时间小时数 -->

<!-- 游戏时间以及作弊设置  -->
<property name="BuildCreate"					value="false" />			<!-- 是否开启作弊模式 -->
<property name="DayNightLength"					value="60" />				<!-- 设置游戏中一天的时间，游戏中一天是实时分钟数：60分钟  -->
<property name="DayLightLength"					value="18" />				<!-- 设置游戏中一天的白天时间，在游戏中，太阳每天照耀18小时 -->
<property name="DropOnDeath"					value="0" />				<!-- 设置死亡掉落。 0 = 无, 1 = 全部, 2 = 仅工具袋 , 3 = 仅背包, 4 = 全部删除 -->
<property name="DropOnQuit"						value="0" />				<!-- 设置退出游戏掉落 0 = 无, 1 = 全部, 2 = 仅工具袋 , 3 = 仅背包-->
<property name="BedrollDeadZoneSize"			value="15" />				<!-- 在床的多少距离内不刷僵尸（框“半径”，即每侧长度为给定值的2倍的框），该区域内不会繁殖僵尸。 -->
<property name="BedrollExpiryTime"				value="45" />				<!-- 离线玩家的床还可以保存 -->

<!-- 性能相关 -->
<property name="MaxSpawnedZombies"				value="20" />				<!-- 整个地图上生成的僵尸数量。更改此设置会对性能产生巨大影响。 -->
<property name="MaxSpawnedAnimals"				value="10" />				<!-- 如果您的服务器有大量玩家，您可以增加此限制以添加更多野生动物。动物消耗的CPU不如僵尸多。注意：这不会导致更多的动物任意产卵：生物群落产卵系统只在给定区域产卵一定数量的动物，但如果你有很多玩家都分散在一起，那么你可能达到了极限，可以增加它。-->
<property name="ServerMaxAllowedViewDistance"	value="6" />				<!-- 玩家的视距（6-12）。对内存使用和性能有很大影响。 -->

<!-- 僵尸设置 -->
<property name="EnemySpawnMode"					value="true" />				<!-- 是否生成僵尸 -->
<property name="EnemyDifficulty"				value="0" />				<!-- 僵尸的难度 0 = 正常, 1 = 疯狂 -->
<property name="ZombieFeralSense"				value="0" />				<!-- 僵尸的活跃程度   		0-3 (休息, 白天, 晚上, 整天) -->
<property name="ZombieMove"						value="0" />				<!-- 僵尸移动速度     		0-4 (走路, 慢跑, 奔跑, 冲刺, 噩梦) -->
<property name="ZombieMoveNight"				value="3" />				<!-- 僵尸晚上移动速度 		0-4 (走路, 慢跑, 奔跑, 冲刺, 噩梦) -->
<property name="ZombieFeralMove"				value="3" />				<!-- 僵尸活跃时的移动速度 	0-4 (走路, 慢跑, 奔跑, 冲刺, 噩梦) -->
<property name="ZombieBMMove"					value="3" />				<!-- 僵尸血月移动速度 		0-4 (走路, 慢跑, 奔跑, 冲刺, 噩梦) -->
<property name="BloodMoonFrequency"				value="7" />				<!-- 血月发生的频率（天）。设置为“0”表示无血月 -->
<property name="BloodMoonRange"					value="0" />				<!-- 实际的血月日可以随机偏离上述设置的多少天。将此设置为0将使血月恰好每n天发生一次，如血月频率中指定的那样 -->
<property name="BloodMoonWarning"				value="8" />				<!-- 几点开始提示当天是血月，设置为“-1”不提示  -->
<property name="BloodMoonEnemyCount"			value="4" />				<!-- 这是在血月尸群中，每个玩家在任何时候都可以存活（同时繁殖）的僵尸数量，然而，在多人游戏中，超级僵尸 会覆盖这个数量。还要注意的是，你的游戏阶段设定了每一方僵尸的最大数量。较低的游戏阶段值会导致僵尸数量低于血月数设置。更改此设置会对性能产生巨大影响。 -->

<!-- 游戏物资设置  -->
<property name="LootAbundance"					value="200" />				<!-- 战利品掉落率:整数百分比 -->
<property name="LootRespawnDays"				value="7" />				<!-- 物资刷新的天数 -->
<property name="AirDropFrequency"				value="72"/>				<!-- 空投刷新的频率，0 =从不 -->
<property name="AirDropMarker"					value="true"/>				<!-- 是否在地图/指南针上标记空投位置 -->

<!-- 多人游戏设置 -->
<property name="PartySharedKillRange"			value="100"/>				<!-- 共享经验和僵尸积分的距离设置，你必须在距离之内才能获得团队共享杀死经验和任务团队杀死目标积分。 -->
<property name="PlayerKillingMode"				value="0" />				<!-- 玩家击杀设置 (0 = 不能击杀 , 1 = 仅击杀盟友 , 2 = 仅击杀陌生人 , 3 = 杀死所有人) -->

<!-- 土地设置 -->
<property name="LandClaimCount"					value="1"/>					<!-- 每个玩家允许的最大领地数量。 -->
<property name="LandClaimSize"					value="41"/>				<!-- 领地的范围 -->
<property name="LandClaimDeadZone"				value="30"/>				<!-- 每个领地之间的距离（除非你是他的盟友） -->
<property name="LandClaimExpiryTime"			value="7"/>					<!-- 玩家离线时领地不受保护的天数 -->
<property name="LandClaimDecayMode"				value="0"/>					<!-- 玩家离线时领地保护程度的衰减程度。 0=慢 (线性) , 1=快 (指数), 2=无 (在领地到期前全面保护). -->
<property name="LandClaimOnlineDurabilityModifier"	value="4"/>				<!-- 玩家在线时领地保护程度。 0 表示无限 (不会受到伤害). 默认为 4x -->
<property name="LandClaimOfflineDurabilityModifier"	value="4"/>				<!-- 玩家离线时领地保护程度。 0 表示无限 (不会受到伤害). 默认为 4x -->
<property name="LandClaimOfflineDelay"			value="0"/>					<!-- 玩家离线时领地的活跃程度从在线变成离线需要的时间（分钟）。 默认为 0 -->

<!-- 动态网格系统设置 -->
<property name="DynamicMeshEnabled"				value="true"/>				<!-- 是否启用动态网格系统 -->
<property name="DynamicMeshLandClaimOnly"		value="true"/>				<!-- 动态网格系统是否只在玩家的土地要求区域激活-->
<property name="DynamicMeshLandClaimBuffer"		value="1"/>					<!-- 动态网格土地所有权基础块的半径大小。 -->
<property name="DynamicMeshMaxItemCache"		value="1"/>					<!-- 动态网格系统可以同时存在多少项，值越高，使用的RAM越多 -->

<!-- twitch设置 -->
<property name="TwitchServerPermission"			value="90"/>				<!-- 在服务器上使用twitch集成所需的权限级别 -->
<property name="TwitchBloodMoonAllowed"			value="false"/>				<!-- 如果服务器允许在血月期间进行twitch操作。这可能会导致服务器延迟，在血月期间会产生额外的僵尸。 -->
```

## 配置防火墙

如果服务器存在防火墙或 NAT 服务器需要端口映射, 则需要配置一下端口规则.

默认情况下最少需要如下几个端口

| 端口    | 协议  |
| ------- | ----- |
| `26900` | `TCP` |
| `26900` | `UDP` |
| `26902` | `UDP` |
| `38664` | `UDP` |

如果需要 Web Panel 还需要 `8080` 的 `UDP` 协议

图为 NAT 情况下的映射规则, 可作为参考

![](https://imyrs.net/u/i/img/202306131301588.png)

## 启动服务器

修改完配置后, 使用 `./sdtdserver start` 启动游戏

![](https://imyrs.net/u/i/img/202306131253657.png)

此时使用 `./sdtdserver dt` 命令可以查看服务器状态

![](https://imyrs.net/u/i/img/202306131255393.png)

## 更多

### 配置管理员权限

推荐从 Web Panel 使用命令更改

```bash
admin add <玩家名> <权限级别>	# 给予玩家管理权限（最高级别为0）
admin remove <玩家名>		# 移除玩家的管理权限
admin update <玩家名> <权限等级>	# 更改管理权限级别
```

或在存档文件夹中的 `serveradmin.xml` 中使用 64 位 `steamid` 来添加.

```xml
<admins>
     <user steamID="YOUR_STEAM_ID" name="as remark" permission_level="0" />
</admins>
```

### 添加 Mod

服务器的 mod 和自己平时玩的 mod 基本通用.

首先创建 Mods 文件夹 (以 `sdtdserver` 用户操作)

```bash
mkdir ./serverfiles/Mods && cd ./serverfiles
```

安装 `zip` `unzip` 用于解压 mod 压缩包

```bash
# Ubuntu/Debian
apt install zip
 
# RedHat/CentOS
yum install zip unzip -y
```

上传 mod 文件到 `./serverfiles/Mods` 并解压.

### 常用管理员指令

```
admin add <玩家名> <权限级别>		# 给予玩家管理权限（最高级别为0）
admin remove <玩家名>			# 移除玩家的管理权限
admin update <玩家名> <权限等级>		#提高管理权限级别
# 建议先在 web panel 用上述指令给自己管理权限
# 然后就可以直接在游戏中按F1使用下面的指令了

cm				# 打开或关闭 create 模式
dm				# 打开或关闭 debug 模式
ban <玩家名> <时间>		# 禁止玩家登陆服务器一段时间 (minutes, hours, days, weeks, months, years)
kill <id/name>			# 杀死指定玩家
listplayers lp			# 获取在线玩家信息
give <id/name> <物品> <数量>	# 给玩家某样东西
shutdown			# 关闭服务器
say <信息>			# 以server的名义广播一条信息
```

## 最后

文中 Mods 部分的内容是总结他人内容而来, 暂没有实际操作实践, 如有问题请谅解.