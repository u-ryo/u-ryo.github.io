---
layout: post
title: "Secure Boot 2 (using self-signed cert)"
date: "2015-11-05 01:52"
author: 'u-ryo'
categories: [Linux, Secure Boot, self-signed cert]
comments: true
published: true
---
###訂正
/EFI/ubuntu/... ではブート時に読み込まない、
と前回下記ましたが、efibootmgrで変更出来そうです。
が、今回はUSBメモリからのブートが要件なので、
efibootmgrは使えません。

###手法2
「loopbackモジュールを含めたgrubを自力で署名する」
手法が漸くわかりました

1. `efitools`をapt-get installしておきます
1. 自己署名証明書(pem形式とder形式)を作成します
2. 必要なモジュールを含めた`grub`を作ります
(ここまでモジュール入れないとうまく行きませんでした。
`iso9660`と`cpio`くらいでいいような気はしますが下限は試してません)
```
$ grub-mkimage -d /usr/lib/grub/x86_64-efi -o BOOTx64.EFI -O x86_64-efi -p "" part_gpt part_msdos ntfs ntfscomp hfsplus fat ext2 normal chain boot configfile linux multiboot efi_gop loopback linuxefi affs afs bfs btrfs cbfs cpio cpio_be exfat hfs iso9660 jf
```
1. 出来たBOOTx64に自己署名証明書で署名します
```
$ sbsign --key /etc/secureboot/key-material/test-key.rsa --cert /etc/secureboot/key-material/test-cert.pem BOOTx64.EFI
```
1. 上記で出来る`BOOTx64.EFI.signed`を、USBメモリの`/EFI/BOOT/BOOTx64.EFI`としてコピーします
2. 自己署名証明書(der形式)をUSBメモにコピーします
3. PCでUSBブートし、BIOSメニューから、自己署名証明書を読み込ませます

これでUSBメモリ内のisoファイルから立ち上がるようになりました。

ただ、BIOSメニューに自己署名証明書を読み込ませるUIのない場合、
実機が今はないので試せないこともあり、どうするのかはよくわかりません。

cf.

* [技術者見習いの独り言 SecureBootとLinux](http://qiita.com/kunichiko/items/12cbccaadcbf41c72735)
* [SecureBoot](https://wiki.ubuntu.com/SecurityTeam/SecureBoot)
* [The rEFInd Boot Manager: Managing Secure Boot](http://www.rodsbooks.com/refind/secureboot.html)
* [SUSE document セキュアブート](https://www.suse.com/ja-jp/documentation/sles11/book_sle_admin/data/sec_uefi_secboot.html)