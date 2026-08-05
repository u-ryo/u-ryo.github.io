---
layout: post
title: "google photo api"
date: "2019-06-02 03:49"
author: 'u-ryo'
categories: [google photos, curl]
comments: true
published: true
---
一日千枚とか写真撮る人だと写真がすぐ溜まっちゃうんですよね。
backupは無限の[Google Photos](https://photos.google.com)に、ということで、前はPicasaのAPI、`upload_gphots`を使ってたんですけど、もう無くなっちゃっていて。どうしよう、途方に暮れていました。暫くぶりに探すと、丁度1年程前からGoogle Photo APIが整備されたようで、良かったです。ずっと待っていました。
[[追記あり] Google Photos APIsでアルバム作成と写真のアップロード](https://qiita.com/zaki-lknr/items/97c363c12ede4c1f25d2)と[Google Photoを業務システムのクラウドストレージとして使った結果](https://qiita.com/wo_k_harada/items/7327971a2414040e5a86)、[本家API Document](https://developers.google.com/photos/library/guides/list)を参考に早速使ってみます。

## ACCESS_TOKENの取得
- [APIの有効化](https://developers.google.com/photos/library/guides/get-started)
- [Google Developer Console](https://console.developers.google.com/)から「認証情報」→「OAuth2.0クライアントID」無ければ上の「認証情報を作成」pulldown menuから「OAuthクライアントID」(「ウェブアプリケーションの種類」は「その他」)で作成
- 上記「クライアントID」「クライアント シークレット」をメモ
- 次のURLに`$CLIENT_ID`を入れてbrowserでaccess、AUTHORIZATION_CODEを取得 (`https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=$CLIENT_ID&redirect_uri=urn:ietf:wg:oauth:2.0:oob&scope=https://www.googleapis.com/auth/photoslibrary&access_type=offline` (SCOPEはGoogle PhotoでのR/W accessの場合は`https://www.googleapis.com/auth/photoslibrary`)
- 以下のようにして、`ACCESS_TOKEN`及び`REFRESH_TOKEN`を得る
```sh
$ AUTHORIZATION_CODE=4/wnmGpTh__1zdrgdjmPWyetUI7C1mvsjRrA_IyZmwY7aSeYppD9X_9iB
$ CLIENT_ID=952391557281-s8b8ditnocfu590fi0ntsfk76rbmkm80.apps.googleusercontent.com
$ CLIENT_SECRET=k6XPLuryMWUtKDKmS1cYgW0r
$ REDIRECT_URI=urn:ietf:wg:oauth:2.0:oob

$ curl --data "code=$AUTHORIZATION_CODE" --data "client_id=$CLIENT_ID" --data "client_secret=$CLIENT_SECRET" --data "redirect_uri=$REDIRECT_URI" --data "grant_type=authorization_code" --data "access_type=offline" https://www.googleapis.com/oauth2/v4/token
{
  "access_token": "ya29.GlsOB-ebr6NrI78UemOPHcm1-jdw0XkxD8iiSqE-Bh5xB_Sx8bhKsRhRyz7gqJy45A-HIF6s6GF0j5wz0dmNppVqEMhtUurAwfbe-xgEsR5MZFjoIY3ONOx8zd4Q",
  "expires_in": 3600,
  "refresh_token": "1/8LrGRLdBaFJYHlOr0rEAyZcgC9yDl2PcZZyrbqoxc7c",
  "scope": "https://www.googleapis.com/auth/photoslibrary",
  "token_type": "Bearer"
}
```
- `ACCESS_TOKEN`は1時間しか有効でないので、適宜`REFRESH_TOKEN`を使って更新
```sh
$ REFRESH_TOKEN=1/8LrGRLdBaFJYHlOr0rEAyZcgC9yDl2PcZZyrbqoxc7c
$ CLIENT_ID=952391557281-s8b8ditnocfu590fi0ntsfk76rbmkm80.apps.googleusercontent.com
$ CLIENT_SECRET=k6XPLuryMWUtKDKmS1cYgW0r

$ ACCESS_TOKEN=`curl -s --data "refresh_token=$REFRESH_TOKEN" --data "client_id=$CLIENT_ID" --data "client_secret=$CLIENT_SECRET" --data "grant_type=refresh_token" https://www.googleapis.com/oauth2/v4/token|jq .access_token -r`
```

`REFRESH_TOKEN`を取得すれば、あと`CLIENT_ID`と`CLIENT_SECRET`が分かれば`ACCESS_TOKEN`は更新できます。

## ALBUMの作成
- 既存のAlbumの確認
```sh
$ curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://photoslibrary.googleapis.com/v1/albums?pageSize=50
```
- 既存のAlbumの確認(`nextPageToken`がある場合)
```sh
$ curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://photoslibrary.googleapis.com/v1/albums?pageSize=50&pageToken=...
```
- 新規Albumの作成
```sh
$ DIR=20190428
$ curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{ "album": { "title":"'$DIR'" } }' https://photoslibrary.googleapis.com/v1/albums
{
  "id": "ADIlBkAOcfB64a_Opnwdjgxeq6jhQv4GQ1pZQ-wse2o2hiBIofuhefmFycfTtIcLAG0inLt0FlZn",
  "title": "20190428",
  "productUrl": "https://photos.google.com/lr/album/ADIlBkAOcfB64a_Opnwdjgxeq6jhQv4GQ1pZQ-wse2o2hiBIofuhefmFycfTtIcLAG0inLt0FlZn",
  "isWriteable": true
}
```

## UPLOAD and adding to Album
2段階になっていて、

1. binary fileをuploadして`UPLOAD_TOKEN`を得る
1. `UPLOAD_TOKEN`を元に`mediaItems:batchCreate`する(ALBUM名はここで渡す。batch処理なので複数の`UPLOAD_TOKEN`を渡せる)

```sh
FILENAME=20190428/img_0699.jpg

$ curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H 'Content-type: application/octet-stream' -H 'X-Goog-Upload-Protocol: raw' -H "X-Goog-Upload-File-Name: $FILENAME" --data-binary "@$FILENAME" https://photoslibrary.googleapis.com/v1/uploads
CAIS+QIASsy4zFbu3IKGgbDXA5XshOGvHPOTLuqbqTN9MQxKRVCsxp3YHbus+qsDgA0GCjuqXdmGpv1uWFxKvf8GYa/8VJQ1S6FUcmGWgw6Hdj14QNYtBRVbXU/cdq/Jkx3ZblG5co3hnY6+yMxih26kB0vTWfWp9GwIE904y5yXEE1pm/V0bFduzA/CZvdlAU9EvWfqKnNO7c3nozWUalm5WUZHHatVQZT+H5+jD0Bq3YwMUdfC5KF048AxFa9auW1HpQGdboalYyXBCJksfzteWtU53wZ8rFnZgHwrui9uA2ptnTuDlin2m+WXU+HqaVRuKX1ou5BzalI4P0gVfWql41Af6nuvvEdMNZ39tEvK2EARUX0CUd8veDznZiWjtPcRqpJnvjDRCxaSgr/cn+JXf9k7SnD0DYVWOdM64lngcAuXxsKk6RJJOVxQBUi6XAG04dHnKxDndqjl+fcH9qWAmpXejPx8Kgn6GX7TgatiKHEG4ybvWjStWg1JPg

$ UPLOAD_TOKEN=CAIS+...
$ ALBUM_ID=ADIlBkAOcfB64a_Opnwdjgxeq6jhQv4GQ1pZQ-wse2o2hiBIofuhefmFycfTtIcLAG0inLt0FlZn
$ curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{ "albumId": "'$ALBUM_ID'", "newMediaItems":[ { "simpleMediaItem": { "uploadToken": "'$UPLOAD_TOKEN'" }} ] }' https://photoslibrary.googleapis.com/v1/mediaItems:batchCreate
{
  "newMediaItemResults": [
    {
      "uploadToken": "CAIS+...",
      "status": {
        "message": "OK"
      },
      "mediaItem": {
        "id": "ADIl...",
        "productUrl": "https://photos.google.com/lr/album/ADIl...",
        "mimeType": "image/jpeg",
        "mediaMetadata": {
          "creationTime": "2019-04-28T02:40:35Z",
          "width": "5184",
          "height": "3456"
        },
        "filename": "20190428/img_0699.jpg"
      }
    }
  ]
}
```

## folderまるっとupload
- 事前準備
```sh
REFRESH_TOKEN=...
CLIENT_ID=...
CLIENT_SECRET=...
ACCESS_TOKEN=`curl -s --data "refresh_token=$REFRESH_TOKEN" --data "client_id=$CLIENT_ID" --data "client_secret=$CLIENT_SECRET" --data "grant_type=refresh_token" https://www.googleapis.com/oauth2/v4/token|jq .access_token -r`
```
- Album作成
```sh
DIR=...
ALBUM_ID=`curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"album":{"title":"'$DIR'"}}' https://photoslibrary.googleapis.com/v1/albums|jq -r .id`
```
- `~/photo/$DIR`以下の`img_*.jpg` filesのuploadとalbum登録(約100 files毎に`ACCESS_TOKEN`のrefresh)

```sh
for i in ~/photo/$DIR/img_*.jpg; do if [ ! ${i##*00.jpg} ];then ACCESS_TOKEN=`curl -s --data "refresh_token=$REFRESH_TOKEN" --data "client_id=$CLIENT_ID" --data "client_secret=$CLIENT_SECRET" --data "grant_type=refresh_token" https://www.googleapis.com/oauth2/v4/token|jq .access_token -r`;fi;UPLOAD_TOKEN=`FILENAME=$DIR/${i##*/}; curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H 'Content-type: application/octet-stream' -H 'X-Goog-Upload-Protocol: raw' -H "X-Goog-Upload-File-Name: $FILENAME" --data-binary "@$i" https://photoslibrary.googleapis.com/v1/uploads`;curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"albumId":"'$ALBUM_ID'","newMediaItems":[{"simpleMediaItem":{"uploadToken":"'$UPLOAD_TOKEN'"}}]}' https://photoslibrary.googleapis.com/v1/mediaItems:batchCreate|tee -a /tmp/upload.log|grep -q error&&echo $i;done|tee /tmp/upload_failed.log
```

uploadに失敗したfile namesが標準出力と`/tmp/upload_failed.log`に出てくるので、後刻それらをretry。

```sh
for i in `cat /tmp/upload_failed.log`; do if [ ! ${i##*00.jpg} ];then ACCESS_TOKEN=`curl -s --data "refresh_token=$REFRESH_TOKEN" --data "client_id=$CLIENT_ID" --data "client_secret=$CLIENT_SECRET" --data "grant_type=refresh_token" https://www.googleapis.com/oauth2/v4/token|jq .access_token -r`;fi;UPLOAD_TOKEN=`FILENAME=$DIR/${i##*/}; curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H 'Content-type: application/octet-stream' -H 'X-Goog-Upload-Protocol: raw' -H "X-Goog-Upload-File-Name: $FILENAME" --data-binary "@$i" https://photoslibrary.googleapis.com/v1/uploads`;curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"albumId":"'$ALBUM_ID'","newMediaItems":[{"simpleMediaItem":{"uploadToken":"'$UPLOAD_TOKEN'"}}]}' https://photoslibrary.googleapis.com/v1/mediaItems:batchCreate|tee -a /tmp/upload.log|grep -q error&&echo $i;done
```

これではbatch処理を活かしていない(複数の`UPLOAD_TOKEN`をbatchCreateしていない)のですが、Googleだけに割とすぐ終わること、error handlingがあまりにも複雑になることから、都度batchCreateすることにしました。

私の場合、1000 filesで約3GB弱、を目処に分割してuploadしています。
uploadしたfilesは全て「元のサイズ」で保存されてしまい、Google Driveの容量を消費してしまうので、[設定](https://photos.google.com/settings)から「容量を解放」しなければなりません。これが「1日1回」となっているものの、だからといって24時間後に再度実行しても「ファイルを圧縮できませんでした。ストレージを復元できるのは 1 日 1 回だけです。」と言われて出来ず、困っています。実際に再度実行できるまでには1.5日〜2日かかるようで、これが最大のneckになっています。

## 新規Albumへの既存files追加
これはダメでした。
何度試してもダメだったので、調べてみると、公式Documentに、
[Note that you can only add media items that have been uploaded by your application to albums that your application has created.](https://developers.google.com/photos/library/guides/manage-albums)とあります。
なんでやねん!
何で既存の画像とAPI経由の画像とを区別するのか、わけわかりません。
それじゃぁ、っていうんで、既にGoogle Photos上にある写真も改めてuploadしてalbumにaddしたら、それは出来ました。しかし、「元のサイズ」になってしまって容量を食ってしまいます。これについても「容量を解放」しなければなりません。
全く七面倒臭いものです。

ちなみに、以下のようにやりました。paginationが発生しない程度のAlbum限定で、
```sh
$ DIR=20171005
$ ALBUM_ID=`curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{ "album": { "title":"'$DIR'" } }' https://photoslibrary.googleapis.com/v1/albums|jq -r .id`

$ MEDIA_ITEMS=`curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"pageSize":"100","filters":{"dateFilter":{"dates":[{"year":2017,"month":10,"day":5}]}}}' https://photoslibrary.googleapis.com/v1/mediaItems:search|jq .mediaItems[].id|sed -z 's/\n/,/g'`

$ curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"mediaItemIds":['$MEDIA_ITEMS']}' https://photoslibrary.googleapis.com/v1/albums/${ALBUM_ID}:batchAddMediaItems
{
  "error": {
    "code": 400,
    "message": "Request contains an invalid media item id.",
    "status": "INVALID_ARGUMENT"
  }
}
```
なら既存のものは手でやればいいではないか?
やってみたのですが、微妙に手元のfilesと数が合わなかったりするので、困難です。手元に3282枚あってGoogle PhotosのAlbumに3281枚あった時、どうやって差分をあぶり出したらいいのですか?! 全downloadはなしで。もう一つでは、手元に5221枚、Google Photosに5230枚と増えてます! Manuallyでは限界を感じました。

## Album中の全file名取得

自己解決しました。
`NEXT_PAGE_TOKEN`あると面倒くさいんですけど、これで何とか。

### Album探し

最初のpageに目的のalbumがあるかをこれ↓で探す

```sh
curl -s -H "Authorization: Bearer $ACCESS_TOKEN" 'https://photoslibrary.googleapis.com/v1/albums?pageSize=50'|jq -r .albums[].title,.nextPageToken
```
なければ次のpageへ。

```sh
curl -s -H "Authorization: Bearer $ACCESS_TOKEN" 'https://photoslibrary.googleapis.com/v1/albums?pageSize=50&pageToken=Ck...'|jq -r .albums[].title,.nextPageToken
```

見付かれば、`ALBUM_ID`を同定。

```sh
curl -s -H "Authorization: Bearer $ACCESS_TOKEN" 'https://photoslibrary.googleapis.com/v1/albums?pageSize=50&pageToken=Ck...'|grep -1 20080318
      "id": "ADI...",
      "title": "20080318",
      "productUrl": "https://photos.google.com/lr/album/ADI...",
```

そうしてから徐に、
```sh
ALBUM_ID=...
NEXT_PAGE_TOKEN=`curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"pageSize":"100","albumId":"'$ALBUM_ID'"}' https://photoslibrary.googleapis.com/v1/mediaItems:search|jq -r '.mediaItems[].filename,.nextPageToken'|tee /tmp/files.txt|tail -1`;while [ "$NEXT_PAGE_TOKEN" != null ];do NEXT_PAGE_TOKEN=`curl -s -X POST -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-type: application/json" -d '{"pageSize":"100","albumId":"'$ALBUM_ID'","pageToken":"'$NEXT_PAGE_TOKEN'"}' https://photoslibrary.googleapis.com/v1/mediaItems:search|jq -r '.mediaItems[].filename,.nextPageToken'|tee -a /tmp/files.txt|tail -1`;done
```

その後、

```
grep -vE -e '.{300,}' -e null /tmp/files.txt
```

で取り出せます。

それで比較(`diff -y --suppress-common-lines <(cd ~/photo;ls .../img_*|sort) <(grep -vE -e '.{300,}' -e null /tmp/files.txt|sort)`)したところ、足りないものはわかりました。
ですが、何故かAPIで取ると3282個なのに
「コンテンツ 3283個」と表示されていたり...
よく精査すると、なるほど、Google Photosが勝手に?作った、
[アシスタント](https://photos.google.com/assistant)にある
`MOVIE.mp4`(ムービー)や`...-EFFECTS.jpg`(スタイルを適用した写真)、
`...-PANO.jpg`(パノラマ)が含まれているから?
いや、一つ(5230個と表示)はそれが原因で9個多く数が表示されていたのですが、
もう一つ、「コンテンツ 3283個」は、APIで取得するといくら見ても
3282個しかない、自動生成物もない、です。謎です。

それと、EnrichmentとかいってTextやLocationとMapを入れられるんですけど、
それらを取得する術がなく、Textに書いたことを検索するとかも出来ず。
GUIから入れてみましたが、要素を追加する度にいちいち先頭に戻される、
移動すると他の要素もどこかに行ってしまうことがある、
等何だかなぁ、というものでした。
Googleならこんなもんじゃないだろー!
