---
layout: post
title: "Hybrid MBR/GPT for dual booting MBR/UEFI"
date: "2015-11-04 02:24"
author: 'u-ryo'
categories: [Linux, MBR, UEFI, boot]
comments: true
published: true
---
BIOSブートとUEFIブートの両対応を求められました。

UEFIブートなので、GPT(GUID Partition Table)は必須です。
が、GPTだと旧来のMBRではないので、BIOSブートがこけます。
そこで、Hybridなるものがありますと。
これは、`gdisk`でしか作れません。
`r`で「recovery and transformation options (experts only)」に入って、
`h`(make hybrid MBR)で作ります。

```
$ sudo gdisk /dev/sdb

GPT fdisk (gdisk) version 0.8.8

Partition table scan:
   MBR: protective
   BSD: not present
   APM: not present
   GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): p
Disk /dev/sdb: 7831552 sectors, 3.7 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 3BF2DA63-FB2F-4616-ACE6-694968477678
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 7831518
Partitions will be aligned on 2048-sector boundaries
Total free space is 4029 sectors (2.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
    1            2048         5734399   2.7 GiB     0700
    2         5734400         7829503   1023.0 MiB  EF00

Command (? for help): r

Recovery/transformation command (? for help): p
Disk /dev/sdb: 7831552 sectors, 3.7 GiB
Logical sector size: 512 bytes
    1            2048         5734399   2.7 GiB     0700
    2         5734400         7829503   1023.0 MiB  EF00

Recovery/transformation command (? for help): o

Disk size is 7831552 sectors (3.7 GiB)
MBR disk identifier: 0x00000000
MBR partitions:

Number  Boot  Start Sector   End Sector   Status      Code
    1                     1      7831551   primary     0xEE

Recovery/transformation command (? for help): h

WARNING! Hybrid MBRs are flaky and dangerous! If you decide not to use one,
just hit the Enter key at the below prompt and your MBR partition table will
be untouched.

Type from one to three GPT partition numbers, separated by spaces, to be
Place EFI GPT (0xEE) partition first in MBR (good for GRUB)? (Y/N): n

Creating entry for GPT partition #1 (MBR partition #1)
Enter an MBR hex code (default 07):
Set the bootable flag? (Y/N): n

Creating entry for GPT partition #2 (MBR partition #2)
Enter an MBR hex code (default EF):
Set the bootable flag? (Y/N): y

Unused partition space(s) found. Use one to protect more partitions? (Y/N): y
Note: Default is 0xEE, but this may confuse Mac OS X.
Enter an MBR hex code (default EE):

Recovery/transformation command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.

Caution: More than one 0xEE MBR partition found. This can cause problems
in some OSes.
The operation has completed successfully.

$ sudo gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.8

Partition table scan:
   MBR: hybrid
   BSD: not present
   APM: not present
   GPT: present

Found valid GPT with hybrid MBR; using GPT.

Command (? for help): q
```

その上で、grub-installします。
MBR用には、

```
sudo mount /dev/sdb2 /mnt
sudo grub-install --force --no-floppy --boot-directory=/mnt/boot /dev/sdb
sudo vi /mnt/boot/grub/grub.cfg

set timeout=3
set default=0

menuentry "Run Ubuntu Live ISO" {
       set gfxpayload=keep
       loopback loop (hd0,gpt2)/airscope2.iso
       linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=/airscope2.iso splash --- debian-installer/language=ja keyboard-configuration/layoutcode?=jp keyboard-configuration/modelcode?=jp109
       initrd (loop)/casper/initrd.lz
}
```

UEFI用には、

```
sudo mount /dev/sdb2 /mnt
sudo mkdir -p /mnt/EFI/BOOT/
grub-mkimage -d /usr/lib/grub/x86_64-efi -o /mnt/EFI/BOOT/BOOTx64.EFI -O x86_64-efi -p "" part_gpt part_msdos ntfs ntfscomp hfsplus fat ext2 normal chain boot configfile linux multiboot efi_gop
sudo cp -rp /usr/lib/grub/x86_64-efi /mnt/EFI/BOOT/
sudo vi /mnt/EFI/BOOT/grub.cfg

configfile (hd0,GPT2)/boot/grub/grub.cfg
```