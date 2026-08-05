---
layout: post
title: "Viewing/Recording TV (Hiden no Tare)"
date: "2018-09-17 20:57"
author: 'u-ryo'
categories: [linux, vlc, ffmpeg]
comments: true
published: true
---
何を今更ですが、ここ15年、以下のようにTVを見たり録画したりしています。
3000円くらいで買ったUSB Video Capture Box(LiveView:絶版)を使い続けています。
画像と音声は別々だしsizeも320x240と凄く小さいんですけど、
1秒2MBとfile sizeもcompactだし細かい字は読めないものの話の筋は十分わかるので、
いっかなぁと。

```
echo 'ffmpeg -s 320x240 -r 12 -vd /dev/video -ad /dev/dsp -y -t 0:15:00 ~/avi/hanbun_aoi_146.avi &> /dev/null' |at 8:00

vlc --color -I dummy v4l:/dev/video:norm=ntsc:size=640x480:adev=/dev/dsp:audio=0:channel=0:fps=12 --sout "#transcode{vcodec=mp4v,acodec=mpga,vb=3000,ab=256,venc=ffmpeg{keyint=80,vt=800000},deinterlace}:std{access=http,mux=asf,url=0.0.0.0:8080}" --ttl 2

echo '(sleep 45;pkill -f ffmpeg) &> /dev/null'|at 7:59
```

vlcのこんなparameter、もう探し当てられません。
vlcのversionは古い(0.8.2)ので注意です。
上記USB Boxに適合するkernel drvierのため、Debian sarge限定です。
最後のは、朝ドラ始まる前まで安心してTVを見ていられるように、です。

ついでに、いつもsargeをinstallしている手順です。
自分用のメモです。

### /etc/rc.local

```
#!/bin/sh
modprobe tda7313
modprobe saa7111-new
#insmod ~u-ryo/ov511-2.31/tuner.o
sleep 3
#v4lctl setchannel 1
#v4lctl setinput CVBS-0
v4lctl setinput S-Video-0
v4lctl volume 50000
```

kernel2.6なら`tuner.o`は出来ないので不要。

### INSTALL Sarge
1. `unetbootin`でUSB作成可
1. boot直後、`linux26`を選択
1. cdrom mountに失敗するのでmountは手で。`mount /dev/disks/scsi/disk0/lun0/part0 /cdrom`とか何とか
1. symlink張れないので、自分で`rm /cdrom/dists/stable;cp -rp /cdrom/dists/sarge /cdrom/dists/stable`
1. source URLは、`deb http://ftp.riken.jp/Linux/debian/debian-archive/debian sarge main contrib non-free`を自分でedit

### /etc/apt/sources.list
backportを付加。

```
deb http://ftp.riken.jp/Linux/debian/debian-archive/debian sarge main contrib non-free
deb http://archive.debian.org/backports.org/ sarge-backports main contrib non-free
```

```
$ sudo apt-get install -t sarge-backports debhelper dh-make build-essential
$ sudo m-a a-i madwifi
$ sudo apt-get install alsa-tools ntp ntpdate ntp-server screen sudo ffmpeg vlc bzip2 wpasupplicant
$ sudo m-a a-i alsa
$ sudo m-a a-i thinkpad
$ sudo /sbin/modprobe thinkpad
$ sudo /sbin/modprobe thinkpadpm
$ sudo /sbin/modprobe rtcmosram
$ sudo /sbin/modprobe smapi
```

`ffmpeg`は`sarge-backports`のに上げるとVideo4Linux2(V4L2)を入力に要求するようになるのでダメポ。


### alsa
alsaでないと声が割れる。
後から入れても有効にならないので、
`sudo /sbin/modprobe -r i810_audio`
そうすると`/dev/audio`が無くなるので、sndを入れ直し。
`for i in snd_mpu_401 snd_rawmidi snd_seq_device snd_intel8x0 snd_intel8x0m snd_ac97_codec snd_pcm_oss snd_mixer_oss snd_pcm snd_timer snd snd_page_alloc;do sudo /sbin/modprobe -r $i;done`
再度`snd`を`modprobe`
`sudo /sbin/modprobe snd`


### for wpa_supplicant
cf. [ThinkPad 535X Debian Sarge の 無線LANのまとめ](http://near-unix.blogspot.jp/2010/09/thinkpad-535x-debian-sarge-lan_29.html)

### `/etc/default/wpasupplicant`

```
ENABLED=1
OPTIONS="-w -i ath0 -D madwifi -c /etc/wpa_supplicant.conf -dd"
```

### `/etc/wpa_supplicant.conf`

```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
eapol_version=1
ap_scan=1
fast_reauth=1

network={
        ssid="YOUR_SSID"
        psk="YOUR_PreSharedKey"
}
```

```
sudo apt-get install -t sarge-backports wpasupplicant
sudo mv /etc/init.d/wpasupplicant.dpkg-bak /etc/init.d/wpasupplicant
sudo mv /etc/default/wpasupplicant.dpkg-bak /etc/default/wpasupplicant
sudo mv /etc/wpa_supplicant.conf.dpkg-bak /etc/wpa_supplicant.conf
sudo ln -s /sbin/wpa_supplicant /usr/sbin/wpa_supplicant
sudo /usr/sbin/update-rc.d wpasupplicant defaults
```

### `/etc/network/interfaces`

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto ath0
iface ath0 inet dhcp
  wpa-driver madwifi
  wpa-conf /etc/wpa_supplicant.conf
  wpa-scan-ssid 1
  wpa-proto RSN
  wpa-pairwise CCMP
  wpa-group CCMP
  wpa-key-mgmt WPA-PSK
  wpa-ssid Disney
  wpa-psk ru011415
```

`wpa-driver`と`wpa-conf`が無いとダメだった。

### for mosh

色々試したがmoshはダメだった。諦めた。
