---
title: "N60 Pro 与红米 AX6 的 WDS 无线组网：命令行版完整配置"
date: 2026-11-19T19:00:00+08:00
author: "逃跑"
draft: false
categories: tool
number: "006"
tags: ["OpenWrt", "WDS", "802.11kvr", "usteer", "无线组网"]
---

> 这是[那篇 LuCI 网页版教程](../n60pro-ax6-openwrt-wireless-mesh-kvr-roaming/)的命令行版本。网页版适合手动点，这篇适合让 AI 读进去之后照着敲——每一步都是可以直接执行的命令，附带检查点和预期结果。两台都是 OpenWrt 25.12.2 官方主线，包管理器是 apk。

> 2026-11-19 按 OpenWrt v25.12.2 源码树、hostapd `ca266cc2`、mt76 `39c960c3` 和官方 wiki 复核过一遍，正文已直接改到位，修订明细见文末「修订记录」。

## 一、先把两件事分清楚

我一开始把这两个概念搞混过，写教程前必须说清楚，否则后面会越配越乱。

| | WDS / 4addr | 802.11k/v/r |
|---|---|---|
| 管什么 | 两台 AP 之间的无线网线，把二层网络透传过去 | 手机在多个 AP 之间切换时的引导 |
| 作用在哪 | AP 和 AP 之间 | AP 和手机之间 |
| 缺了会怎样 | 子节点下设备拿不到主路由的 IP | 网照常通，但漫游黏、切换卡一两秒 |
| 依赖 | 内核 mac80211 + 无线驱动，跟 wpad 版本无关 | hostapd 协议；11k/11r basic 就有，只有 11v 必须完整版 wpad |

两者互相独立。网能通不等于切得顺，这是两笔账。本教程两样都配。

顺带把 wpad 版本这件事一次说清：25.12.2 的 `wpad-basic-mbedtls` 已经带 802.11r、802.11w 和 WPA3 SAE，802.11k 的 `rrm` 代码也是无条件编译的；它唯一缺的是 802.11v（WNM / BSS Transition），也就是 AP 主动建议手机「隔壁信号更好，切过去」的能力。网上「basic 不支持 11r 所以要换」的说法是错的，但 11v 确实要完整版。

## 二、动手前

全程用网线连着设备操作。换包、改无线、保存应用都会让 Wi-Fi 断几十秒，用 Wi-Fi 连着改等于把自己关在门外。

| 前提 | 要求 |
|---|---|
| 固件 | 两台都是 OpenWrt 25.12.2 官方主线稳定版 |
| 网络 | N60 Pro 已是正常主路由，能拨号能上网 |
| 物理连接 | AX6 先放在主路由旁边：**先直连电脑改 IP**，再把 LAN 口接到主路由 LAN 口；桥通了再断电搬到目标房间 |
| 备份 | 两台都在 LuCI 里生成一份配置备份 |

物理连接要分两段，别搞反：

1. **第一次配置**：AX6 单独上电，网线直连电脑。它默认 LAN 是 `192.168.1.1`，电脑能自动拿到地址，先登录进去把 LAN 改成 `192.168.1.2`、关掉 DHCP（第 3 步的前半段）。
2. **改完 IP 之后**：把网线从电脑拔下来，插到**主路由的 LAN 口**，电脑也接回主路由（或用主路由 Wi-Fi），从主路由侧访问 `192.168.1.2` 继续配无线。

不要一上来就把还在 `192.168.1.1` 的 AX6 插到主路由 LAN 口——两台都叫 `192.168.1.1`，会直接地址冲突。也不要改完 IP 后还指望电脑直连能自动拿地址：AX6 的 DHCP 已经关了，电脑只能靠旧租约或手动配 `192.168.1.x` 静态地址。

下文约定：主路由 LAN 是 `192.168.1.1`，子节点是 `192.168.1.2`，SSID 是 `MyHome`，密码 `MyHome@2026`。这些值按你的实际环境替换，但两台必须一致的部分我会标出来。

## 三、第 0 步：确认 radio 编号和驱动

`radio0` / `radio1` 到底哪个是 5 GHz，不要照抄别人的，实测：

```bash
iw dev
wifi status
```

看每个 `wlanX` 属于哪个 `phy`、跑在哪个频段。N60 Pro 的无线部分是 MT7986AV（SoC）+ MT7976GN（2.4 G）+ MT7976AN（5 G），不是 SoC 内置双频；AX6 通常是 `radio0` 走 QCN5024（2.4 G）、`radio1` 走 QCN5054（5 G）——但仍以你自己看到的为准。

再确认驱动和固件正常，这一步能提前排除一大半"无线起不来"：

```bash
# N60 Pro
lsmod | grep mt7915
dmesg | grep -i -E 'mt76|firmware' | tail -20

# AX6
lsmod | grep ath11k
dmesg | grep -i -E 'ath11k|firmware|board' | tail -20
```

检查点：看到驱动模块，且没有 firmware 加载失败的报错。AX6 尤其要确认没有 `board-2.bin` 找不到的提示。

## 四、第 1 步：补齐软件包并换成完整版 wpad

25.12 用 apk，不是 opkg。先把必需的包过一遍：

```bash
apk update
apk info -e iw

# N60 Pro
apk add kmod-mt7915e kmod-mt7986-firmware iw

# AX6
apk add kmod-ath11k-ahb ath11k-firmware-ipq8074 ipq-wifi-redmi_ax6 iw
```

AX6 的 `ipq-wifi-redmi_ax6` 是板级校准数据，缺了射频根本起不来，而且最容易被精简版固件删掉。

完整版选 `wpad-openssl`，一个包把 11k/v/r 和 802.11s 都覆盖了，将来想改用 mesh 回程也不用再换包。两台都执行：

```bash
/etc/init.d/wpad stop
apk del wpad-basic-mbedtls
apk add wpad-openssl
```

两个包提供同一个可执行文件，顺序反了会报冲突，必须先删后装。先 `stop` 是为了避免覆盖正在运行的 `/usr/sbin/wpad` 时报 `Text file busy`。移除到装好之间 Wi-Fi 是断的。

验证：

```bash
strings /usr/sbin/wpad | grep bss_transition
```

有输出才是完整版。这个串的解析在 hostapd 的 `#ifdef CONFIG_WNM_AP` 门内，basic 没有 WNM、full 有，正好对应我们要补的 802.11v。以前流传的 `grep auth_server_shared_secret` 也能碰巧区分两个包，但它测的是 RADIUS 有没有编进去（basic 恰好开了 `CONFIG_NO_RADIUS`），证明的不是 11v 这件事。

然后重启无线：

```bash
wifi down && wifi up
```

## 五、第 2 步：N60 Pro 开 WDS 接入点

先保证主路由本身是好的——能拨号、DHCP 正常、Wi-Fi 能连。

假设 `radio1` 是 5 GHz（按第 0 步实测替换）：

```bash
uci set wireless.radio1.channel='149'
uci set wireless.radio1.htmode='HE80'
uci set wireless.radio1.country='CN'

uci set wireless.default_radio1.mode='ap'
uci set wireless.default_radio1.network='lan'
uci set wireless.default_radio1.ssid='MyHome'
uci set wireless.default_radio1.encryption='psk2'
uci set wireless.default_radio1.key='MyHome@2026'
uci set wireless.default_radio1.isolate='0'
uci set wireless.default_radio1.wds='1'

uci commit wireless
wifi down && wifi up
```

几个值的选择理由：

| 设置 | 选它 | 原因 |
|---|---|---|
| 信道 | 149 | 5.8 G 段功率上限高、不碰 DFS。下拉里没有就选 36，但 AX6 必须跟着一致 |
| 频宽 | 80 MHz | AX6 的 5 G 在 80 MHz 下是 4 流，160 MHz 会掉到 2 流 |
| 加密 | psk2 | `ft_psk_generate_local` 只在 FT-PSK 密钥套件下生效，SAE 下会被静默忽略；psk2 也能照顾不认 SAE/11r 的老客户端 |
| wds | 1 | 关键。普通 AP 不接受四地址帧，AX6 连不上 |

关于加密再补一句：WPA3/SAE 确实强制 PMF，但「11r 本身就要 11w」是不成立的——FT-PSK 在 `ieee80211w=0` 下也能工作，需要 PMF 的是 SAE/FT-SAE。这里选 psk2 的真实原因是本地生成 PMK 这个省事开关只对 FT-PSK 有效，跟 PMF 不冲突。想上 WPA3 就得改用 FT-SAE，并放弃 `ft_psk_generate_local`，那是另一套配置。

2.4 GHz 设成同 SSID，但不承担回程，可以和 AX6 错开信道：

```bash
uci set wireless.radio0.channel='1'
uci set wireless.radio0.htmode='HT20'
uci set wireless.default_radio0.ssid='MyHome'
uci set wireless.default_radio0.key='MyHome@2026'
uci set wireless.default_radio0.encryption='psk2'
uci set wireless.default_radio0.network='lan'
uci commit wireless && wifi reload
```

检查点：手机能连上 MyHome 并上网。此时只有一台，还看不出漫游效果，正常。

## 六、第 3 步：AX6 改成纯二层桥

AX6 如果还开着 DHCP 和 NAT，就变成双层 NAT，设备拿到的网关是 AX6，主路由上的插件拦不到流量。必须让它变成"无线交换机"。

这一步的「关 DHCP + 改 IP」建议在**网线直连电脑**时完成（AX6 此时还是 `192.168.1.1`）；改完把网线换到主路由 LAN 口，再从 `192.168.1.2` 继续。如果已经接在主路由 LAN 口上，先把 AX6 断开，避免两个 `192.168.1.1` 冲突。

先把三层服务关掉：

```bash
uci set dhcp.lan.ignore='1'
uci set dhcp.lan.ra='disabled'
uci set dhcp.lan.dhcpv6='disabled'
uci set dhcp.lan.ndp='disabled'
uci commit dhcp

/etc/init.d/dnsmasq disable
/etc/init.d/odhcpd disable
/etc/init.d/firewall disable

uci set network.lan.proto='static'
uci set network.lan.ipaddr='192.168.1.2'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='192.168.1.1'
uci set network.lan.dns='192.168.1.1'
uci commit
```

只关 dnsmasq 不够：odhcpd 还会在 `br-lan` 上发 IPv6 RA 和 DHCPv6，主路由和 AX6 同时发 RA 会造成 IPv6 地址混乱。把 `ra` / `dhcpv6` / `ndp` 设成 `disabled`，再把 odhcpd 服务停掉，才算干净。

5 GHz 改成 WDS 客户端（`radio1` 按实测替换）：

```bash
uci set wireless.radio1.channel='149'
uci set wireless.radio1.htmode='HE80'
uci set wireless.radio1.country='CN'

uci set wireless.default_radio1.mode='sta'
uci set wireless.default_radio1.wds='1'
uci set wireless.default_radio1.network='lan'
uci set wireless.default_radio1.ssid='MyHome'
uci set wireless.default_radio1.encryption='psk2'
uci set wireless.default_radio1.key='MyHome@2026'

uci commit wireless
```

`network` 必须是 `lan` 不是 `wwan`——这是最容易踩的坑，写成 wwan 就变成二级路由，桥接全废。

两台都开 STP，防止将来误插网线形成环路。不要盲写 `@device[0]`，先确认 `br-lan` 到底排在哪个 bridge 段：

```bash
uci show network | grep "type='bridge'"
# 输出形如 network.@device[0].name='br-lan'
```

然后按实际段名设置（下例假设就是 `@device[0]`），**N60 Pro 和 AX6 都要执行**：

```bash
uci set network.@device[0].stp='1'
uci commit network
```

重启：

```bash
reboot
```

## 七、第 4 步：验证桥是否真的通了 ⚠️ 门禁

这是整个方案最关键的检查，ATH11K 和 MT76 两个驱动之间做 WDS 没有公开实测报告，这一步就是验证它。不通就停下来，别往下配。

在 AX6 上：

```bash
iw dev
ls /sys/class/net/br-lan/brif/
ping -c 3 192.168.1.1
```

检查点：`wlan` 接口出现在 `br-lan` 的 `brif` 目录里，能 ping 通主路由。

这里别用 `ip link show master br-lan`：BusyBox 的 `ip link show` 只认 `to/scope/up/label/dev` 这几个过滤参数，不支持 `master`，多数官方固件上会直接报参数错误。`/sys/class/net/br-lan/brif/` 是内核导出的桥端口列表，不依赖 iproute2，最稳。

最终判据：拿一台设备插到 AX6 的 LAN 口，它应该拿到 `192.168.1.x` 的 IP，网关是 `192.168.1.1`。

| 现象 | 判断 |
|---|---|
| 拿到 192.168.1.x | 二层桥成功，继续 |
| 拿到别的网段 | 配成了三层中继，回头查 `network lan` 和两端 `wds` |
| 完全拿不到 | 检查两端信道、频宽、加密、密码是否一字不差 |

顺带说一句，Linux 内核 2021 年有一个 `ath11k: fix 4-addr tx failure for AP and STA modes`（commit `34c67dc3`，2021-07-21，Tested-on IPQ8074），给 ath11k 的 AP 和 STA 两种模式补上 4addr 发帧支持，首次随内核 v5.16 发布。所以驱动层理论上是支持的，缺的只是真机实测。这比"完全没依据的赌注"让人放心一些，但仍要以你这步的实测为准。

另外提醒一句安全边界：`psk2` 加 4addr 二层透传，意味着任何知道 Wi-Fi 密码的设备都能直接进入你的内网桥，而不只是"连上一个 Wi-Fi"。密码别外泄；如果家里有访客或 IoT 设备，最好单独划 VLAN，而不是全屋一个扁平二层网。

## 八、第 5 步：AX6 放出同名 Wi-Fi

前提：第 4 步全部通过。

5 GHz 上再加一个普通 AP 接口给手机连（和回程用的 STA 接口共存于同一个 radio）：

```bash
uci add wireless wifi-iface
uci set wireless.@wifi-iface[-1].device='radio1'
uci set wireless.@wifi-iface[-1].mode='ap'
uci set wireless.@wifi-iface[-1].network='lan'
uci set wireless.@wifi-iface[-1].ssid='MyHome'
uci set wireless.@wifi-iface[-1].encryption='psk2'
uci set wireless.@wifi-iface[-1].key='MyHome@2026'
```

用 `uci show wireless` 确认新 section 的名字，后面配漫游要用。

2.4 GHz 设成同 SSID、错开信道：

```bash
uci set wireless.radio0.channel='11'
uci set wireless.radio0.htmode='HT20'
uci set wireless.default_radio0.ssid='MyHome'
uci set wireless.default_radio0.key='MyHome@2026'
uci set wireless.default_radio0.encryption='psk2'
uci set wireless.default_radio0.network='lan'

uci commit wireless
wifi down && wifi up
```

## 九、第 6 步：开 k/v/r 漫游

两台都要配，而且 Mobility Domain 必须完全一样。给所有 **AP 模式** 的接口配，不要给 WDS STA 配（sta 模式不支持这些）。

这一步千万别只配 `default_radio0`。N60 Pro 的 5 G 是 `default_radio1`，AX6 的 5 G 是第 5 步新建的那个 iface——只配 2.4 G 的结果就是「2.4 G 能漫游、5 G 不能」，而手机绝大多数时间待在 5 G。最稳的写法是遍历所有 `mode='ap'` 的接口：

```bash
for s in $(uci show wireless | sed -n "s/^wireless\.\([^.=]*\)\.mode='ap'$/\1/p"); do
  echo "configuring $s"
  uci set wireless.$s.ieee80211r='1'
  uci set wireless.$s.mobility_domain='a1b2'
  uci set wireless.$s.ft_over_ds='0'
  uci set wireless.$s.ft_psk_generate_local='1'

  uci set wireless.$s.ieee80211k='1'
  uci set wireless.$s.rrm_neighbor_report='1'
  uci set wireless.$s.rrm_beacon_report='1'
  uci set wireless.$s.bss_transition='1'
done

uci commit wireless
wifi down && wifi up
```

跑完用 `uci show wireless | grep -E 'mode|ieee80211|bss_transition|mobility'` 核对一遍，N60 的 `default_radio0` / `default_radio1`、AX6 的 `default_radio0` / 新建 5 G AP 四个接口都应该有这些选项。

| 选项 | 作用 |
|---|---|
| 802.11r | 切换时复用已派生密钥，从秒级降到毫秒级 |
| Mobility Domain | 标识同一个漫游域，两台必须相同 |
| 本地生成 PMK | 每台自己算漫游密钥，省去手工交换密钥列表。只在 WPA2-PSK（FT-PSK）下成立 |
| 802.11k | 给手机邻居列表，不用全信道扫描 |
| 802.11v | 主动建议手机切换，这正是第 1 步换包要补的能力 |

配完要是某个 SSID 消失了，先别怀疑 k：`rrm` 是无条件编译的，basic 也支持 802.11k。真正会让 hostapd 报 `unknown configuration item` 退出、netifd 把接口置为 disabled 的是 `bss_transition` 这一行——也就是 802.11v。确认 wpad 已经是完整版，或者先把 `bss_transition` 去掉验证。

## 十、第 7 步：装 usteer

802.11k 只是让 AP 具备报告邻居的能力，真正让多台 AP 互相知道对方、做引导决策要靠 usteer。dawn 和 usteer 二选一，别同时装。

两台都装：

```bash
apk add usteer luci-app-usteer
/etc/init.d/usteer enable
/etc/init.d/usteer restart
```

默认配置里 `network lan` 就能让两台通过局域网互相发现。但只装不配，usteer 基本不会动作：`policy.c` 里 `better_signal_strength()` 在 `signal_diff_threshold` 为 0 时直接返回 false，而默认值就是 0；`roam_scan_snr` 默认也是 0。官方 wiki 推荐的两个值恰好都不在默认里，所以补上：

```bash
# 默认 /etc/config/usteer 是匿名 section，用 @usteer[0] 定位
uci set usteer.@usteer[0].roam_scan_snr='-65'
uci set usteer.@usteer[0].signal_diff_threshold='8'
uci commit usteer
/etc/init.d/usteer reload
```

如果你的配置里给 section 起过名字，先 `uci show usteer` 看一眼，把 `@usteer[0]` 换成实际名字。

查邻居和客户端：

```bash
logread | grep -i usteer
ubus call usteer get_clients
```

看到对端 MAC、以及手机 MAC 落在某台节点上就对了。邻居列表一直空的真实原因通常是另一台 wpad 不是完整版、`network` 配错、或者防火墙拦了 usteer 的通信，不是缺什么补丁。

`static-neighbor-reports` 是另一条路：它只做「静态 802.11k 邻居报告」，需要你手工填邻居信息（`ubus call hostapd.<ifname> rrm_nr_get_own` 可以取本机信息），适合跨厂商、usteer 实在发现不了时兜底，不是 usteer 的插件，装了也不会让 usteer 自动发现对端。

## 十一、第 8 步：验收

ping 要从**手机侧**发起。在 AX6 上 ping 主路由测的是回程链路，人在走动、手机在漫游，这个 ping 根本反映不出来。

手机上装个终端 App（或拿另一台连着 Wi-Fi 的设备），持续 ping 主路由，然后从一台附近走到另一台附近：

```bash
ping 192.168.1.1
```

同时在 AP 上看手机落在哪台：

```bash
ubus call usteer get_clients
# 或者
iwinfo wlan1 assoclist
iw dev wlan1 station dump | grep -E 'Station|signal|connected time'
```

（`wlan1` 换成实际承载 AP 的那个接口，用 `iw dev` 确认。）

| 现象 | 判断 |
|---|---|
| 几乎不丢包 | 802.11r 生效 |
| 丢 1–3 个包 | 手机自己在漫游，11r 没起作用 |
| 一直不切 | k/v 没生效，核对两台 Mobility Domain、5 G 接口是否都配了、wpad 是否完整版 |

再确认漫游是真的在切：手机的 MAC 应该会从一台的「已连接工作站」里消失、出现在另一台上。

吞吐别期望太高。AX6 的 5 GHz 单 radio 同时做回程和覆盖是半双工共享 airtime：回程 4×4 HE80 理论能到 500–700 Mbps，但手机侧通常是 2×2，实测 300–450 Mbps 属于手机侧的正常预期，不是回程上限，也不是配置问题。

## 十二、排障速查

| 症状 | 原因 | 处理 |
|---|---|---|
| AX6 无线完全起不来 | 缺 BDF 或固件包 | 补 `ipq-wifi-redmi_ax6`、`ath11k-firmware-ipq8074` |
| 开了 `bss_transition` 后 Wi-Fi 消失 | 还在用 basic 版 wpad，不认 802.11v | 换 `wpad-openssl`，或先去掉 `bss_transition` |
| 桥通了但子节点没网 | `network` 写成 wwan，或 DHCP/防火墙/odhcpd 没关 | 改成 lan，确认第 3 步 |
| WDS 时断时通、吞吐极低（且手动开过 WED） | WED 硬件卸载与 4addr 冲突 | `echo 'options mt7915e wed_enable=N' >> /etc/modules.conf` 后重启 |
| 整个网突然瘫痪 | 二层环路 | 确认 STP 开了，拔掉多余网线 |
| 装/升级 wpad 包时报 `Text file busy` | 正在运行的 `/usr/sbin/wpad` 被覆盖 | 先 `/etc/init.d/wpad stop`，再 `apk del` / `apk add` |
| AX6 高负载卡死、自动重启，或 sysupgrade 刷固件失败 | ipq807x 512 MB 机型内存紧张，ath11k RX buffer 吃内存（#24154） | `grep -E 'MemTotal|MemAvailable' /proc/meminfo`；升级前停 wpad、关掉不必要服务再刷 |

MT7986 那个 WED 冲突值得多说一句。补丁是 Shengyu Qu 2025 年 4 月发的 v10 RESEND（`mt76: add wds support when wed is enabled`），它**没有**进 Linux 主线——torvalds master 和 6.12～6.18 的 `mt76.h` 里都搜不到 `MT76_WED_WDS`；进的是 OpenWrt 的 mt76 包，25.12.2 的 `kmod-mt7915e` 用的就是 mt76 仓库 `39c960c3`，已经包含。它是驱动代码，不是无线固件；wcid 256–271（另一 band 768–783）同步 4addr。`wed_enable` 在 `mt7915/mmio.c` 里是 bool 模块参数且默认 false，所以除非你手动开过 WED，这条在新版本上多半不是故障源，当历史坑看即可。

参数写在 `/etc/modules.conf` 是对的：它是 ubox 的 conffile，改过会被 sysupgrade 备份，kmodloader 的 `load_options()` 也只从这个文件读 `options` / `blacklist`。`/etc/modules.d/` 里放的是「模块名 [参数]」逐行加载，不认 `options ...` 这种语法；自己新建的文件也不在 conffile 列表里，sysupgrade 后大概率丢，不建议往那儿写。

## 十三、第 4 步不通怎么办

按代价从低到高：

1. **拉一根网线**。AX6 当有线的 dumb AP，漫游立刻能用，只是要多一根线。最省事，最可靠。
2. **换一台 mt76 的机器**，比如红米 AX6000，和 N60 Pro 同平台同驱动，回程成功率最高。AX6 挪去别处当 AP。
3. **试 802.11s**。两端模式都改成 802.11s、设同一个 Mesh ID。`wpad-openssl` 自带 mesh 不用换包，但跨驱动的 11s 同样没验证过，是另一个赌注。

## 十四、参考来源

- [openwrt v25.12.2 hostapd Makefile — wpad 各变体描述原文](https://github.com/openwrt/openwrt/blob/v25.12.2/package/network/services/hostapd/Makefile)
- [openwrt v25.12.2 hostapd-basic.config — basic 实际编译选项](https://github.com/openwrt/openwrt/blob/v25.12.2/package/network/services/hostapd/files/hostapd-basic.config)
- [hostapd 2.11 config_file.c — bss_transition 的 CONFIG_WNM_AP 门](https://sources.debian.org/data/main/w/wpa/2%3A2.11-7/hostapd/config_file.c)
- [ath11k: fix 4-addr tx failure for AP and STA modes（commit 34c67dc）](https://git.zx2c4.com/linux-dev/commit?id=34c67dc366419e06129dad0f32f521842bdff9bc)
- [mt76: add wds support when wed is enabled（WDS+WED 修复）](https://lists.infradead.org/pipermail/linux-mediatek/2025-April/092378.html)
- [OpenWrt mt76 包（25.12.2 使用 39c960c3）](https://github.com/openwrt/mt76/commit/39c960c3ada558b4c2e7915772483d3731573d09)
- [OpenWrt WDS 官方文档](https://openwrt.org/docs/guide-user/network/wifi/wifiextenders/wds)
- [OpenWrt Wifi Roaming — 11k/11r basic 可用，11v 需要完整版 wpad](https://openwrt.org/docs/guide-user/network/wifi/roaming)
- [OpenWrt usteer 配置 — 推荐 roam_scan_snr=-65、signal_diff_threshold=8](https://openwrt.org/docs/guide-user/network/wifi/usteer)
- [openwrt#24154 — ath11k RX buffer 在低内存 ipq807x 上耗尽内存](https://github.com/openwrt/openwrt/issues/24154)

## 十五、修订记录（2026-11-19 复核）

发布后按 OpenWrt v25.12.2 源码树、hostapd `ca266cc2`、mt76 `39c960c3` 和官方 wiki 逐条复核，正文已直接改过。主要变更：

- **第 0 步**：N60 Pro 的无线硬件改为 MT7986AV + MT7976GN（2.4 G）+ MT7976AN（5 G），不是「SoC 内置双频」。
- **第 1 步**：验证串从 `auth_server_shared_secret` 改成 `bss_transition`；basic 其实带 11k/11r/11w/SAE，只缺 11v。
- **第 2 步**：psk2 的理由改成「`ft_psk_generate_local` 只对 FT-PSK 生效」，不再说「WPA3 强制 PMF 冲突」；PMF 是 SAE/FT-SAE 的要求，不是 11r 本身的要求。
- **第 3 步**：补关 odhcpd 和 IPv6 RA/DHCPv6/NDP；STP 改成先查 `br-lan` 段名再设，并明确两台都执行；物理连接改成「先直连改 IP，再把 AX6 LAN 接主路由 LAN」，避免两个 `192.168.1.1` 冲突。
- **第 4 步**：BusyBox 的 `ip link show` 不支持 `master` 过滤，改用 `/sys/class/net/br-lan/brif/`；补 4addr 的安全边界说明。
- **第 6 步**：从「只配 `default_radio0`」改成遍历所有 AP 接口，补上 N60 的 5 G 和 AX6 新增的 5 G AP；接口被禁用的元凶是 `bss_transition`（11v），不是 802.11k。
- **第 7 步**：usteer 补 `roam_scan_snr` / `signal_diff_threshold`；`static-neighbor-reports` 改成「跨厂商发现不了时的静态邻居报告方案」，不是 usteer 插件。
- **第 8 步**：ping 改从手机侧发起；吞吐口径改成「手机侧 2×2 的 300–450 Mbps 预期」。
- **排障表**：WED 降级为历史坑，并保留 `/etc/modules.conf`（它是 ubox 的 conffile，sysupgrade 会备份）；升级失败拆成「wpad 包 Text file busy」和「sysupgrade 低内存」两行；补 #24154。
- **第 4 步**：ath11k 4addr 补「首次随内核 v5.16 发布」的限定。

仍然没有答案的：ath11k 与 mt76 跨驱动做 WDS 回程，没有任何公开实测——这正是第 4 步当门禁设计的价值，改不掉也别绕过。
