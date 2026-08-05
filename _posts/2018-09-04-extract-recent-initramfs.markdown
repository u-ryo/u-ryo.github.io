---
layout: post
title: "Extract recent initramfs"
date: "2018-09-04 11:19"
author: 'u-ryo'
categories: [linux, ubuntu]
comments: true
published: true
---
よその会社に用意して貰ったLinux(ubuntu)のSSDがあって、
ちょっとhappy hackingしてみようと。
`initramfs`を展開しようと`zcat initrd.img|cpio -di`とかってやってみたところ、
「not in gzip format」と。
`file initrd.img`とすると「ASCII cpio archive (SVR4 with no CRC)」です。
圧縮fileじゃないのかとそのまま`cpio -id`しても、
何か`kernel/x86/microcode/AuthenticAMD.bin`しか出て来ません。
えー!? どうなってんのー?!
調べてみると、`lsinitramfs`というのでlistは出るらしい、です。
試してみると、確かに色々入ってそうです。
なのに出て来ません。えー。
[FedoraやCentOS 6/7、RHEL 6/7のinitramfsを展開する](http://neocat.hatenablog.com/entry/20150730/1438273548)や[RHEL7 initramfsの展開方法](http://nopipi.hatenablog.com/entry/2015/05/06/235615)を見ると、
きょうびの`initramfs`は違うんですねー。
びっくりです。

ちなみに、`skipcpio`は`apt install dracut`で入ります。
使うには、`/usr/lib/dracut/skipcpio`とfull path指定が必要です。
`binwalk`は`apt install binwalk`でした。

早速試したところ、上手く行くもの(`/boot/initrd.img-4.15.0-33-generic`)もありましたが、
下記のように上手く行かないものもありました。
そして、目的のものは上手く行かない方のものでした。

```
$ /usr/lib/dracut/skipcpio ~/initrd.img-4.15.0-33-generic|file -
/dev/stdin: ASCII cpio archive (SVR4 with no CRC)
```

`binwalk`で見てみると、全く同じ位置にあるのに。

```
$ binwalk ~/initrd.img-4.15.0-33-generic

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             ASCII cpio archive (SVR4 with no CRC), file name: ".", file name length: "0x00000002", file size: "0x00000000"
112           0x70            ASCII cpio archive (SVR4 with no CRC), file name: "kernel", file name length: "0x00000007", file size: "0x00000000"
232           0xE8            ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86", file name length: "0x0000000B", file size: "0x00000000"
356           0x164           ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode", file name length: "0x00000015", file size: "0x00000000"
488           0x1E8           ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/AuthenticAMD.bin", file name length: "0x00000026", file size: "0x00006B2A"
28072         0x6DA8          ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
28672         0x7000          ASCII cpio archive (SVR4 with no CRC), file name: "kernel", file name length: "0x00000007", file size: "0x00000000"
28792         0x7078          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86", file name length: "0x0000000B", file size: "0x00000000"
28916         0x70F4          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode", file name length: "0x00000015", file size: "0x00000000"
29048         0x7178          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/.enuineIntel.align.0123456789abc", file name length: "0x00000036", file size: "0x00000000"
29212         0x721C          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/GenuineIntel.bin", file name length: "0x00000026", file size: "0x00180C00"
1605296       0x187EB0        ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
1605632       0x188000        gzip compressed data, from Unix, last modified: 2018-08-30 06:15:36

$ binwalk /boot/initrd.img-4.15.0-33-generic

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             ASCII cpio archive (SVR4 with no CRC), file name: ".", file name length: "0x00000002", file size: "0x00000000"
112           0x70            ASCII cpio archive (SVR4 with no CRC), file name: "kernel", file name length: "0x00000007", file size: "0x00000000"
232           0xE8            ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86", file name length: "0x0000000B", file size: "0x00000000"
356           0x164           ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode", file name length: "0x00000015", file size: "0x00000000"
488           0x1E8           ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/AuthenticAMD.bin", file name length: "0x00000026", file size: "0x00006B2A"
28072         0x6DA8          ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
28672         0x7000          ASCII cpio archive (SVR4 with no CRC), file name: "kernel", file name length: "0x00000007", file size: "0x00000000"
28792         0x7078          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86", file name length: "0x0000000B", file size: "0x00000000"
28916         0x70F4          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode", file name length: "0x00000015", file size: "0x00000000"
29048         0x7178          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/.enuineIntel.align.0123456789abc", file name length: "0x00000036", file size: "0x00000000"
29212         0x721C          ASCII cpio archive (SVR4 with no CRC), file name: "kernel/x86/microcode/GenuineIntel.bin", file name length: "0x00000026", file size: "0x00180C00"
1605296       0x187EB0        ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
1605632       0x188000        gzip compressed data, from Unix, last modified: 2018-08-28 05:01:31
```

`TRAILER`が2つあるので、`skipcpio`を2回かけてみてもダメでした。

```
$ /usr/lib/dracut/skipcpio ~/initrd.img-4.15.0-33-generic|/usr/lib/dracut/skipcpio /dev/stdin|file -
/dev/stdin: data
```

仕方ないので、`binwalk`で得られたbyte数を自分でskipしてやる手法で試すと、上手く解凍されました。

```
$ tail -c +1605632 ~/initrd.img-4.15.0-33-generic|file -
/dev/stdin: data
$ tail -c +1605633 ~/initrd.img-4.15.0-33-generic|file -
/dev/stdin: gzip compressed data, last modified: Thu Aug 30 06:15:36 2018, from Unix
$ tail -c +1605633 ~/initrd.img-4.15.0-33-generic|zcat|cpio -id
```

`binwalk`で得られた1605632ではダメで、1605633でOKでした。
上記サイトには「得られたbyte数以降を`dd`で書き出せ」とかありましたが、
別にそんなことしなくても`tail -c +NNN`で十分です。
