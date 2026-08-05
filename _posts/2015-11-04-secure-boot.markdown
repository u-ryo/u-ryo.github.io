---
layout: post
title: "Secure Boot"
date: "2015-11-04 02:00"
author: 'u-ryo'
categories: [Linux, Secure Boot]
comments: true
published: true
---
Linux (ubuntu)でSecure Bootを求められました。
ubuntuには、Linux-signed-imageとかgrub-efi-amd64-signedとかっていう
パッケージがあるので、何か出来そうなんですけど、
じゃ具体的にどうやるの?
というのは意外と書いてないので、苦労してます。

取り敢えず、grub-installで「--secure-boot」というオプションがあるので、

```
$ sudo grub-install --force --no-floppy --efi-directory=/mnt/ --target=x86_64-efi --uefi-secure-boot /dev/sdb
```

とすると、USBメモリには、/mnt/EFI/ubuntu/ の下にsignedなgrubx64.efiやshimx64.efiが入るんですが、
/EFI/ubuntu/... ではブート時に読み込まないので、
/EFI/BOOT/ にリネームし、
また、/EFI/BOOT/grubx64.efi(又はshimx64.efi) を/EFI/BOOT/BOOTX64.efi にリネームし、
/boot/grub/grub.cfg を、

```
set timeout=3
set default=0

menuentry "Run Ubuntu Live ISO" {
       set gfxpayload=keep
       loopback loop (hd0,gpt1)/airscope3.iso
       linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=/airscope3.iso splash --- debian-installer/language=ja keyboard-configuration/layoutcode?=jp keyboard-configuration/modelcode?=jp109
       initrd (loop)/casper/initrd.lz
}
```

とすると、grubまでは立ち上がりました。
更にブートするには、loopbackモジュールが必要になり、
それはunsignedなので、そこで引っ掛かりますが、
仕組み上、signed grubでモジュールは使えない(使えてしまうとillegalなモジュールが入り込めてしまう)ので、

1. loopbackモジュールを使わない
1. loopbackモジュールを含めたgrubを自力で署名する

前者はUSBメモリをHDとして直接インストールすれば良さそうですが、
8GB以上の容量が必要になります。
現状手元にそんな容量のUSBメモリがありません。
後者は、かなり面倒そうです。Micro$osftに署名してもらうにはお金かかりそうですし、
自分で署名してそのキーをMOK?を使って云々という手法は、
まだ読み込んでません...