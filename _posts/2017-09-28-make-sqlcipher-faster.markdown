---
layout: post
title: "Make SqlCipher Faster"
date: "2017-09-28 14:23"
author: 'u-ryo'
categories: [android, sqlite, sqlcipher]
comments: true
published: true
---
他に言及しているsourceが全く無かったのでまさかと思っていたのですが、[keyの形式を変えて爆速に](http://qiita.com/atr-toru/items/d98a434eecf9f58c443d#keyの形式を変えて爆速に)は本当でした。
試しに、Allcaridaから`c.bin`だけ持ってきてAndroid sample applicationを作って計測した所、従来のkey(4文字)だと約0.4秒、上記ページ例の64字だと0.02秒と顕著な差がありました。

従いまして、「SqlCipherの`key`を64文字の16進数にすれば速くなる」が結論です。

```
従来
09-27 16:25:59.010 30173-30173/sqlcipher.test.jmtech.co.jp.sqlciphertest D/Open: /data/user/0/sqlcipher.test.jmtech.co.jp.sqlciphertest/databases/c.bin
09-27 16:25:59.400 30173-30173/sqlcipher.test.jmtech.co.jp.sqlciphertest D/After open: /data/user/0/sqlcipher.test.jmtech.co.jp.sqlciphertest/databases/c.bin

64字key
09-27 16:24:26.060 28517-28517/sqlcipher.test.jmtech.co.jp.sqlciphertest D/Open: /data/user/0/sqlcipher.test.jmtech.co.jp.sqlciphertest/databases/c2.bin
09-27 16:24:26.080 28517-28517/sqlcipher.test.jmtech.co.jp.sqlciphertest D/After open: /data/user/0/sqlcipher.test.jmtech.co.jp.sqlciphertest/databases/c2.bin
```

そこで、実際にAllcaridaで`c.bin`,`r.bin`だけ64字key版を作って「履歴一覧」画面表示を比較してみると、従来約3.5秒のところ約0.2秒で開けることを確認しました。

```
従来 3.46秒(約0.6秒×5+α)
09-27 17:53:03.670 9054-9054/jp.ideacross.allcardia.main D/After: <<HistoryMainActivity#activityStart:49>> 履歴一覧
09-27 17:53:03.680 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> c.bin Open
09-27 17:53:04.360 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:53:04.380 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> r.bin Open
09-27 17:53:05.010 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:53:05.050 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> c.bin Open
09-27 17:53:05.710 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:53:05.720 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> r.bin Open
09-27 17:53:06.370 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:53:06.390 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> r.bin Open
09-27 17:53:07.030 9054-10071/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open

64字key 0.21秒(0.0+0.01+0.01+0.02+0.03+α)
09-27 17:50:57.350 4493-4493/jp.ideacross.allcardia.main D/After: <<HistoryMainActivity#activityStart:49>> 履歴一覧
09-27 17:50:57.360 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> c2.bin Open
09-27 17:50:57.360 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:50:57.400 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> r2.bin Open
09-27 17:50:57.410 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:50:57.460 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> c2.bin Open
09-27 17:50:57.470 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:50:57.480 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> r2.bin Open
09-27 17:50:57.500 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
09-27 17:50:57.530 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:964>> r2.bin Open
09-27 17:50:57.560 4493-5754/jp.ideacross.allcardia.main D/After: <<DatabaseHelper#open:970>> After Database Open
```

More concretely,

`SQLiteDatabase database = SQLiteDatabase.openOrCreateDatabase(databaseFile, "x\'2DD29CA851E7B56E4697B0E1F08507293D761A05CE4D1B628663F411A8086D99\'", null);`

in Java.

To get a rekeyed file,

```
$ sudo apt install sqlcipher
$ sqlcipher /tmp/c.bin
SQLCipher version 3.15.2 2016-11-28 19:13:37
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> PRAGMA key = '7824';
sqlite> ATTACH DATABASE 'c2.bin' AS c KEY "x'2DD29CA851E7B56E4697B0E1F08507293D761A05CE4D1B628663F411A8086D99'";
sqlite> SELECT sqlcipher_export('c');

sqlite> DETACH DATABASE c;
sqlite>
```

You'll get `c2.bin` with the new 64bit key.

```
$ sqlcipher r.bin
SQLCipher version 3.8.6 2014-08-15 11:46:33
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> PRAGMA key = "7824";
sqlite> ATTACH DATABASE 'r2.bin' AS r KEY "x'2DD29CA851E7B56E4697B0E1F08507293D761A05CE4D1B628663F411A8086D99'";
sqlite> SELECT sqlcipher_export('r');

sqlite> DETACH DATABASE r;sqlite> PRAGMA user_version;
0
sqlite> PRAGMA user_version = 6;
sqlite> PRAGMA user_version;
6
sqlite>
```

You'll get `r2.bin` with the new 64bit key and version 6.

`PRAGMA user_version` is needed because in SQLiteOpenHelper class judges whether it calls `onCreate`(table creation) by `db.getVersion()`.
[[Android]データベースをアップグレードする時](http://d.hatena.ne.jp/isher/20091108/1257684508)

そもそも画面遷移に4秒も掛かるようなAndroidアプリをリリースするなんていうのもunbelievableですが、そういう人達なので...

まぁ、暗号化するにせよ自分なら[Realm](https://realm.io/)使うので、こんな知識不要ですけど、SqlCipher使うなら最初から64字16進code使うべきなんですね。
