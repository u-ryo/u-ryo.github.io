---
layout: post
title: "revive hostapd"
date: "2017-09-03 23:42"
author: 'u-ryo'
categories: [hostapd, linux, wifi]
comments: true
published: true
---
家のオンプレミスサーバが最近五月蝿く、
何か常に3.4GHz近くまでいっていて電気食ってるようなので、
1年弱ぶりに止めてみました。
すると、次、立ち上げた時、
繋げていたPlanexの11/n/g/b Real Portable Wi-Fi Router
[MZK-RP150N](www.planex.co.jp/product/router/mzk-rp150n/point.shtml)
が死にました。
通電はしているようなのですが、
Wi-Fiは繋がらず(ランプも消灯)、resetかけようとマニュアル探して、
その通りにボタン10秒以上長押ししてもPowerランプがつかず。
USB刺してすぐはPowerランプとWirelessランプが暫時点灯するので、
ランプの故障ではない模様。
また、ケーブルがInternet側でもLAN側でも、設定したIP、
defaultのIP、どんなIPでも繋がらず(ping不通)。
ノートPCと直接LANで繋いでpacket captureしても、
packetは漏れてこず。
どうしようもないので、家庭内LANから取り外しました。
けどWi-Fiがないのは不便すぎます。
そこで、余っていたUSB WiFiドングルをオンプレミスサーバに挿し、
実に久し振りにhostapdにしてみました。
21世紀初頭、hostapdで頑張っていたんですが、
設定が面倒臭かったから、MZK-RP150N買ってきたのに、
今はhostapd、凄い簡単になったんですね。
かつてはhostapdが出来るchipから気にしなきゃいけなかったのに、
今やその辺のドングル刺しても大丈夫なんですか。隔世の感。
こういうのは時に応じて調べなきゃならないと思うので、
ここで設定の覚書を書いといても無駄な気がしますけど、一応。

* `apt install hostapd`
* `/etc/hostapd/hostapd.conf`を編集
* bridgeは必要(ウチの場合)→`/etc/network/interfaces`に`br0`の設定要

hostapd.conf
===

```
interface=wlan0
# automatically register wlan0 to br0
bridge=br0
# Wireless LAN adapter driver(fixed value)
driver=nl80211
# SSID name
ssid=...
# 802.11g/a/...
hw_mode=g
# Enable 802.11n
ieee80211n=1
# channel=60 when 802.11a
channel=7
wpa=2 # WPA2
# passphrase for WPA2
wpa_passphrase=...
# stealth
ignore_broadcast_ssid=1
# Mac Address ACL
macaddr_acl=1
# file for Mac Address ACL (permission should be 600)
accept_mac_file=/etc/hostapd/hostapd.accept
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

あと前やったのに書き方忘れてたのが、
`/etc/network/interfaces`の`br0`設定。

interfaces
===

```
iface br0 inet static
	bridge_ports eth1
	bridge_stp off
	bridge_maxwait 0
	address 192.168.X.X
	netmask 255.255.255.0
```

iptalesの設定も変更忘れないように。

Planex MZK-RP150NではMac Address制限が出来なかったので、
ちょっぴり安心に。

ですが、`hostapd.accept`書き換えても
`sudo service hostapd restart`しないと反映されず、
restartすると`br0`消えるっぽいので、注意です。
`hostapd stop`して`ifup br0`、
`hostapd start`するのが正しい手順でしょうか。
その間ノートPCからは接続切れるので、これも気を付けませんと。

それと、
[コマンド一発でLinuxマシンを即席無線LANルーターにできる「create_ap」がすごい便利だった](http://qiita.com/KuwabataK/items/5903c7584657151d576a)は、
別件で使ってみましたが、ホントにすぐにhostapからbridge、
dhcpまで出来て感動モノでした。
今読んで、`haveged`というのを初めて知りました。
オンプレミスサーバにはentropyが足りなかったので入れてみて、
確かにentropyは上がりましたが、
Wireless LANがホントに早くなるのかどうか...
(確かに遅い感じはしてました)。
