---
layout: post
title: "peeking an encrypted luks lvm"
date: "2018-09-19 20:09"
author: 'u-ryo'
categories: [linux, lvm, cryptsetup, luks]
comments: true
published: true
---
純粋に知的好奇心から、
とある暗号化diskをmountして中身を見てみたくなりまして。
targetはubuntuで、立ち上がる時にdisk復号化keyを入力して
立ち上がるような代物です。
それを`dd`でcopyして(最後の方50GBくらいは何かI/Oエラーで
copyがうまく行きませんでしたがそれはそもそもそうなのか、
直前にちょっと蹴っ躓いてコードに少し足が引っかかったせいなのか、
よくわかりません)、
copyした方に対してmountをかけるといきなりpassword聞かれて、
答えると難なくmount出来たもののそれはgrubとかkernelとか
置かれているboot領域。
root領域本体はsdb5の中。
[いますぐ実践! Linux システム管理 / Vol.186 暗号化ファイルシステムを使ってみる ～ LUKS 編](http://www.usupi.org/sysad/186.html)を参考に、
`sudo apt install cryptsetup`して、

```
$ sudo cryptsetup luksOpen /dev/sdb5 luks
Enter passphrase for /dev/sdb5: *******
```

とすると、`/dev/mapper/luks`が出来ます。
それをmountすればいいようなんですが、

```
$ sudo mount /dev/mapper/luks /tmp/luks/
mount: /tmp/luks: unknown filesystem type 'LVM2_member'.
```

と言われて出来ません。
[HDDからのデータサルベージ](http://a5.hatenablog.jp/entry/2011/11/01/000000)を参考に、

```
$ sudo vgscan
  Reading volume groups from cache.
  Found volume group "ubuntu-vg" using metadata type lvm2
```

確かに何かあるようです。
[LVMの使い方](http://d.hatena.ne.jp/kt_hiro/20120117/1326764520)を参考に、

```
$ sudo pvscan
  PV /dev/mapper/luks
  Total: 1 [<232.17 GiB] / in use: 0 [0   ] / in no VG: 1 [<232.17 GiB]
$ sudo pvdisplay -v
    Wiping internal VG cache
    Wiping cache of LVM-capable devices
  "/dev/mapper/luks" is a new physical volume of "<232.17 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/mapper/luks
  VG Name
  PV Size               <232.17 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               w34D6Y-8iVC-zIWW-0WHB-ltuf-FADa-sPMT7k
```

```
$ sudo vgchange -ay
  2 logical volume(s) in volume group "ubuntu-vg" now active
```

これで、`/dev/ubuntu-vg/`が出来ます。

```
$ ls -alrt /dev/ubuntu-vg/
total 0
lrwxrwxrwx  1 root root    7  9月 19 04:44 root -> ../dm-1
drwxr-xr-x 22 root root 4760  9月 19 04:44 ..
lrwxrwxrwx  1 root root    7  9月 19 04:44 swap_1 -> ../dm-2
drwxr-xr-x  2 root root   80  9月 19 04:44 .
```

これをmountすればよいですね。

```
$ sudo mount /dev/ubuntu-vg/root /tmp/luks
```

### unmount / detach

これを外すのに右往左往してしまいました。
`sudo vgremove ubuntu-vg`とかってやってしまいました。
これは、volume groupを消してしまう行為でした。
やってはいけません。

結局、外す時はこれでいいわけですね。

```
$ sudo umount /tmp/luks
$ sudo vgchange -an ubuntu-vg
  0 logical volume(s) in volume group "ubuntu-vg" now active
$ sudo cryptsetup luksClose luks
```

これで`/dev/ubuntu-vg/`も`/dev/mapper/luks`もなくなり、
deviceを外せます。

`vgremove`しちゃったからもうダメかな、と思ったら、戻せるんですね。
[CentOS / RHEL : How to restore/recover a deleted volume group in LVM](https://www.thegeekdiary.com/centos-rhel-how-to-restore-recover-a-delete-volume-group-on-lvm/)を参考に、

```
$ sudo vgcfgrestore --list ubuntu-vg

  File:         /etc/lvm/archive/ubuntu-vg_00000-434080001.vg
  Couldn't find device with uuid w34D6Y-8iVC-zIWW-0WHB-ltuf-FADa-sPMT7k.
  VG name:      ubuntu-vg
  Description:  Created *before* executing 'vgremove ubuntu-vg'
  Backup Time:  Tue Sep 18 21:04:48 2018


  File:         /etc/lvm/archive/ubuntu-vg_00001-841595159.vg
  VG name:      ubuntu-vg
  Description:  Created *before* executing 'vgcreate ubuntu-vg /dev/mapper/luks-1144f2c9-43f4-c433-66a7-81aa5e0f703a'
  Backup Time:  Wed Sep 19 04:31:30 2018


  File:         /etc/lvm/backup/ubuntu-vg
  VG name:      ubuntu-vg
  Description:  Created *after* executing 'vgcreate ubuntu-vg /dev/mapper/luks-1144f2c9-43f4-c433-66a7-81aa5e0f703a'
  Backup Time:  Wed Sep 19 04:31:30 2018
```

このように出て来るので、
身に覚えのあるものを選んで、

```
$ sudo vgcfgrestore -f /etc/lvm/archive/ubuntu-vg_00000-434080001.vg ubuntu-vg
  Restored volume group ubuntu-vg
```

とするとVolume Groupが戻るんですね。すごいー。
助かりました。

あと、`cryptsetup`と`lvm`が予め入っていると、
encrypted diskをUSBで繋げた時点でkeyを聞かれて、
`luksOpen`まで裏でやっちゃうんですね。
自分で`luksOpen`しようとして、

```
$ sudo cryptsetup luksOpen /dev/sdb5 luks2
Enter passphrase for /dev/sdb5: 
Cannot use device /dev/sdb5 which is in use (already mapped or mounted).
```

といわれて「?!」とか思ってしまいました。
この時点で`/dev/mapper/luks-1144f2c9-43f4-c433-66a7-81aa5e0f703a`が出来ています。
