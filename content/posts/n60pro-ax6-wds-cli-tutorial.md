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

## 一、先把两件事分清楚

我一开始把这两个概念搞混过，写教程前必须说清楚，否则后面会越配越乱。

| | WDS / 4addr | 802.11k/v/r |
|---|---|---|
| 管什么 | 两台 AP 之间的无线网线，把二层网络透传过去 | 手机在多个 AP 之间切换时的引导 |
| 作用在哪 | AP 和 AP 之间 | AP 和手机之间 |
| 缺了会怎样 | 子节点下设备拿不到主路由的 IP | 网照常通，但漫游黏、切换卡一两秒 |
| 依赖 | 内核 mac80211 + 无线驱动，跟 wpad 版本无关 | hostapd 协议，要完整版 wpad |

两者互相独立。网能通不等于切得顺，这是两笔账。本教程两样都配。

## 二、动手前

全程用网线连着设备操作。换包、改无线、保存应用都会让 Wi-Fi 断几十秒，用 Wi-Fi 连着改等于把自己关在门外。

| 前提 | 要求 |
|---|---|
| 固件 | 两台都是 OpenWrt 25.12.2 官方主线稳定版 |
| 网络 | N60 Pro 已是正常主路由，能拨号能上网 |
| 物理连接 | AX6 搬到目标房间，先网线直连电脑配置 |
| 备份 | 两台都在 LuCI 里生成一份配置备份 |

下文约定：主路由 LAN 是 `192.168.1.1`，子节点是 `192.168.1.2`，SSID 是 `MyHome`，密码 `MyHome@2026`。这些值按你的实际环境替换，但两台必须一致的部分我会标出来。

## 三、第 0 步：确认 radio 编号和驱动

`radio0` / `radio1` 到底哪个是 5 GHz，不要照抄别人的，实测：

```bash
iw dev
wifi status
```

看每个 `wlanX` 属于哪个 `phy`、跑在哪个频段。N60 Pro 是 MT7986AV 内置双频，AX6 通常是 `radio0` 走 QCN5024（2.4 G）、`radio1` 走 QCN5054（5 G）——但仍以你自己看到的为准。

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

换 wpad 这一步有个流传很广的错误说法要先澄清：网上都说"basic 不支持 802.11r 所以要换"。这是错的。25.12.2 的 basic 包描述里明确写了带 802.11r 和 802.11w，甚至带 WPA3 SAE。真正缺的是 **802.11v（WNM / BSS Transition）**，也就是让 AP 主动建议手机"隔壁信号更好，切过去"的能力。少了它，漫游全靠手机自己判断，而手机普遍很黏，经常信号已经很差了还不切。

完整版选 `wpad-openssl`，一个包把 11k/v/r 和 802.11s 都覆盖了，将来想改用 mesh 回程也不用再换包。两台都执行：

```bash
apk del wpad-basic-mbedtls
apk add wpad-openssl
```

两个包提供同一个可执行文件，顺序反了会报冲突，必须先删后装。移除到装好之间 Wi-Fi 是断的。

验证：

```bash
strings /usr/sbin/wpad | grep auth_server_shared_secret
```

有输出才是完整版。然后重启无线：

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
| 加密 | psk2 | WPA3 强制 PMF，和后面用的本地生成 PMK 冲突，会破坏漫游 |
| wds | 1 | 关键。普通 AP 不接受四地址帧，AX6 连不上 |

2.4 GHz 设成同 SSID，但不承担回程，可以和 AX6 错开信道：

```bash
uci set wireless.default_radio0.ssid='MyHome'
uci set wireless.default_radio0.key='MyHome@2026'
uci set wireless.default_radio0.encryption='psk2'
uci set wireless.default_radio0.network='lan'
uci commit wireless && wifi reload
```

检查点：手机能连上 MyHome 并上网。此时只有一台，还看不出漫游效果，正常。

## 六、第 3 步：AX6 改成纯二层桥

AX6 如果还开着 DHCP 和 NAT，就变成双层 NAT，设备拿到的网关是 AX6，主路由上的插件拦不到流量。必须让它变成"无线交换机"。

用网线连着 AX6，先把三层服务关掉：

```bash
uci set dhcp.lan.ignore='1'
/etc/init.d/dnsmasq disable
/etc/init.d/firewall disable

uci set network.lan.proto='static'
uci set network.lan.ipaddr='192.168.1.2'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='192.168.1.1'
uci set network.lan.dns='192.168.1.1'
uci commit
```

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

两台都开 STP，防止将来误插网线形成环路：

```bash
uci set network.@device[0].stp='1'
uci commit network
```

重启：

```bash
reboot
```

## 七、第 4 步：验证桥是否真的通了 ⚠️ 门禁

这是整个方案最关键的检查，ATH11K 和 MT76 两个驱动之间做 WDS 没有实测报告，这一步就是验证它。不通就停下来，别往下配。

在 AX6 上：

```bash
iw dev
ip link show master br-lan
ping -c 3 192.168.1.1
```

检查点：`wlan` 接口出现在 `br-lan` 里，能 ping 通主路由。

最终判据：拿一台设备插到 AX6 的 LAN 口，它应该拿到 `192.168.1.x` 的 IP，网关是 `192.168.1.1`。

| 现象 | 判断 |
|---|---|
| 拿到 192.168.1.x | 二层桥成功，继续 |
| 拿到别的网段 | 配成了三层中继，回头查 `network lan` 和两端 `wds` |
| 完全拿不到 | 检查两端信道、频宽、加密、密码是否一字不差 |

顺带说一句，我查到 Linux 内核早在 2021 年就有 `ath11k: fix 4-addr tx failure for AP and STA modes`（commit `34c67dc`），专门给 ath11k 的 AP 和 STA 两种模式补上 4addr 发帧支持，测试平台就是 IPQ8074。所以驱动层理论上是支持的，缺的只是真机实测。这比"完全没依据的赌注"让人放心一些，但仍要以你这步的实测为准。

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

以 `default_radio0` 为例：

```bash
uci set wireless.default_radio0.ieee80211r='1'
uci set wireless.default_radio0.mobility_domain='a1b2'
uci set wireless.default_radio0.ft_over_ds='0'
uci set wireless.default_radio0.ft_psk_generate_local='1'

uci set wireless.default_radio0.ieee80211k='1'
uci set wireless.default_radio0.rrm_neighbor_report='1'
uci set wireless.default_radio0.rrm_beacon_report='1'
uci set wireless.default_radio0.bss_transition='1'

uci commit wireless
wifi down && wifi up
```

| 选项 | 作用 |
|---|---|
| 802.11r | 切换时复用已派生密钥，从秒级降到毫秒级 |
| Mobility Domain | 标识同一个漫游域，两台必须相同 |
| 本地生成 PMK | 每台自己算漫游密钥，省去手工交换密钥列表。只在 WPA2-PSK 下成立 |
| 802.11k | 给手机邻居列表，不用全信道扫描 |
| 802.11v | 主动建议手机切换，这正是第 1 步换包要补的能力 |

配完要是某个 SSID 消失了，说明 wpad 没换成完整版——精简版开 k/v 会让无线接口直接禁用，回第 1 步重做。

## 十、第 7 步：装 usteer

802.11k 只是让 AP 具备报告邻居的能力，真正让多台 AP 互相知道对方、做引导决策要靠 usteer。dawn 和 usteer 二选一，别同时装。

两台都装：

```bash
apk add usteer luci-app-usteer
/etc/init.d/usteer enable
/etc/init.d/usteer restart
```

默认配置里 `network lan` 就能让两台通过局域网互相发现。查邻居：

```bash
logread | grep -i usteer
```

看到对端 MAC 出现就对了。要是邻居列表一直空着，补装 `static-neighbor-reports`。

## 十一、第 8 步：验收

持续 ping 主路由，从一台附近走到另一台附近：

```bash
ping 192.168.1.1
```

| 现象 | 判断 |
|---|---|
| 几乎不丢包 | 802.11r 生效 |
| 丢 1–3 个包 | 手机自己在漫游，11r 没起作用 |
| 一直不切 | k/v 没生效，核对两台 Mobility Domain 和 wpad |

再确认漫游是真的在切：手机的 MAC 应该会从一台的「已连接工作站」里消失、出现在另一台上。

吞吐别期望太高。AX6 的 5 GHz 单 radio 同时做回程和覆盖是半双工共享 airtime，中转后大约 300–450 Mbps，千兆宽带拿不满，这是无线回程的固有代价，不是配置问题。

## 十二、排障速查

| 症状 | 原因 | 处理 |
|---|---|---|
| AX6 无线完全起不来 | 缺 BDF 或固件包 | 补 `ipq-wifi-redmi_ax6`、`ath11k-firmware-ipq8074` |
| 开 k/v 后 Wi-Fi 消失 | 还在用 basic 版 wpad | 换 `wpad-openssl` |
| 桥通了但子节点没网 | `network` 写成 wwan，或 DHCP/防火墙没关 | 改成 lan，确认第 3 步 |
| WDS 时断时通、吞吐极低 | MT7986 的 WDS 撞上 WED 硬件卸载 | `echo 'options mt7915e wed_enable=N' >> /etc/modules.conf` 后重启 |
| 整个网突然瘫痪 | 二层环路 | 确认 STP 开了，拔掉多余网线 |
| AX6 升级固件失败 | wpad 没停 | 升级前先 `service wpad stop` |

MT7986 那个 WED 冲突值得多说一句。打进 mt7986 无线固件的 WDS+WED 修复是 2025 年才进主线的，用 wcid 256–271 同步 4addr。25.12.2 大概率已经带上，但要是你的 WDS 时断时通，先关硬件卸载试试，能通就说明是这里的问题。

## 十三、第 4 步不通怎么办

按代价从低到高：

1. **拉一根网线**。AX6 当有线的 dumb AP，漫游立刻能用，只是要多一根线。最省事，最可靠。
2. **换一台 mt76 的机器**，比如红米 AX6000，和 N60 Pro 同平台同驱动，回程成功率最高。AX6 挪去别处当 AP。
3. **试 802.11s**。两端模式都改成 802.11s、设同一个 Mesh ID。`wpad-openssl` 自带 mesh 不用换包，但跨驱动的 11s 同样没验证过，是另一个赌注。

## 十四、参考来源

- [openwrt v25.12.2 hostapd Makefile — wpad 各变体描述原文](https://github.com/openwrt/openwrt/blob/v25.12.2/package/network/services/hostapd/Makefile)
- [ath11k: fix 4-addr tx failure for AP and STA modes（commit 34c67dc）](https://git.zx2c4.com/linux-dev/commit?id=34c67dc366419e06129dad0f32f521842bdff9bc)
- [mt76: add wds support when wed is enabled（WDS+WED 修复）](https://lists.infradead.org/pipermail/linux-mediatek/2025-April/092378.html)
- [OpenWrt WDS 官方文档](https://openwrt.org/docs/guide-user/network/wifi/wifiextenders/wds)
- [OpenWrt Wifi Roaming — k/v 需要完整版 wpad](https://openwrt.org/docs/guide-user/network/wifi/roaming)
- [OpenWrt usteer 配置](https://openwrt.org/docs/guide-user/network/wifi/usteer)
- [openwrt#24154 — ath11k 512 MB 内存耗尽](https://github.com/openwrt/openwrt/issues/24154)
