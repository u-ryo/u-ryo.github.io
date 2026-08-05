---
layout: post
title: "Secure Boot 3 (using signed grub in the official package)"
date: "2015-11-05 06:27"
author: 'u-ryo'
categories: [Linux, Secure Boot, grub]
comments: true
published: true
---
### ubuntu公式パッケージのsigned grubを使う方法

USBメモリに直接ubuntuをインストールするために、
8GBのUSBメモリを用意しました。
それさえあれば、至極簡単です。

1. VirtualBox等で、USBメモリにubuntuを入れる(そのままだと2GBスワップになるので、適宜調整。インストール後は4GB程の使用量なので、最初から5GB程でもよいかも。VMをEFIにしておくのを忘れないように。あと先頭にvfat32のEFI用パーティションを)
2. リブートした後、`/etc/efi/ubuntu/`を`/etc/efi/BOOT`にリネーム
3. `linux-signed-image-generic`も`grub2-efi-amd86-signed`も入っているので、
```
$ sudo cp -rpi /usr/lib/grub/x86_64-efi-signed/grubx64.efi.signed /etc/efi/BOOT/BOOTx64.EFI
```
4. これでSecure BootなPCに挿してリブート