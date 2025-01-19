---
title: "Media CDNを試してみる"
emoji: "🎥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mediacdn", "googlecloud", "cdn"]
published: true
publication_name: "iret"
---
# Media CDNとは

https://cloud.google.com/media-cdn/docs/overview?hl=ja

Cloud CDNはウェブの高速化に最適化されたCDNサービス。
一方でMedia CDNはメディア配信に最適化されています。
ストリーミング再生や長い動画の配信ユースケースに最適で、世界最大級の動画配信プラットフォームであるYouTubeと同じインフラストラクチャを活用できます。

https://cloud.google.com/blog/products/networking/deploy-streaming-service-with-media-cdn?hl=en
>With Google Cloud’s Media CDN, customers can now leverage the same infrastructure that YouTube uses to serve its global users.

但し利用にはGoogleの担当者に[アクセスをリクエスト](https://cloud.google.com/media-cdn/docs/overview?hl=ja#request-access)する必要があり、利用料も公開されていないため現状は担当者に確認をとる必要があります。

# クイックスタートを試してみる
https://cloud.google.com/media-cdn/docs/quickstart?hl=ja

今回利用申請が通ったのでクイックスタートを試してみました。

## Edge Cache Originを作成する
テスト用の動画ファイルをCloud Storageバケットに配置して、公開状態にします。
コンテンツの送信元は、Cloud Storageバケット、サードパーティのストレージサービス、ロードバランサなどが選択できます。

Cloud Storageバケットを指定してEdge Cache Originを作成します。
![](https://storage.googleapis.com/zenn-user-upload/e20043f095d9-20250119.png)

## Edge Cache Serviceを作成する
次にオリジンを配信するEdge Cache Serviceを作成します。
クイックスタートにしたがってキャッシュステータスを返すレスポンスヘッダも設定しました。

![](https://storage.googleapis.com/zenn-user-upload/a8e078526ba8-20250119.png)

この時設定するドメインは動作検証目的であれば自分の所有するものでなくても構いません。
5分ほど待つとグローバルIPアドレスが設定されるのでレスポンスを確認できます。

![](https://storage.googleapis.com/zenn-user-upload/3912f9e3ebcf-20250119.png)

## レスポンスがキャッシュに保存されているかどうかテスト

次のように割り当てたドメインを名前解決してcurlコマンドを実行します。

```bash
curl -svo /dev/null --resolve media-cdn.example.com:80:203.0.113.1 "http://media-cdn.example.com/sample.mp4"
```

レスポンスを一部抜粋。

```bash
> GET /sample.mp4 HTTP/1.1
> Host: media-cdn.example.com
> User-Agent: curl/8.5.0
> Accept: */*
< HTTP/1.1 200 OK
< server: Google-Edge-Cache
< x-cache-status: nrt-202b782; hit, nrt;uncacheable
```

リクエストを繰り返すとキャッシュヒットするようになりました。

[こちらの記事](https://zenn.dev/ma_me/articles/e90c996a768b77)を参考にCloud Storageに直接リクエストしたときとのパフォーマンス比較も行ってみました。

Cloud Storageに対して実行。

```bash
curl -w "http_code: %{http_code}\n\
time_total: %{time_total}\n\
time_namelookup: %{time_namelookup}\n\
time_connect: %{time_connect}\n\
time_appconnect: %{time_appconnect}\n\
time_starttransfer: %{time_starttransfer}\n\
" -svo /dev/null "https://storage.googleapis.com/hoge-media-cdn-test/sample.mp4"
```

```bash
http_code: 200
time_total: 0.959250
time_namelookup: 0.002889
time_connect: 0.009426
time_appconnect: 0.022765
time_starttransfer: 0.212505
```

Media CDNに対して実行。

```
curl -w "http_code: %{http_code}\n\
time_total: %{time_total}\n\
time_namelookup: %{time_namelookup}\n\
time_connect: %{time_connect}\n\
time_appconnect: %{time_appconnect}\n\
time_starttransfer: %{time_starttransfer}\n\
" -svo /dev/null --resolve media-cdn.example.com:80:203.0.113.1 "http://media-cdn.example.com/sample.mp4"
```

```
http_code: 200
time_total: 0.199635
time_namelookup: 0.000020
time_connect: 0.010368
time_appconnect: 0.000000
time_starttransfer: 0.048489
```

Media CDNを使った方が圧倒的にレイテンシが低いことが分かります。

## モニタリング情報を確認する
また、Edge Cache Serviceの詳細からモニタリング情報を確認できます。
![](https://storage.googleapis.com/zenn-user-upload/93dd4e3c8adc-20250119.png)
![](https://storage.googleapis.com/zenn-user-upload/e9b46ac4615c-20250119.png)
![](https://storage.googleapis.com/zenn-user-upload/ed43c3f2b51b-20250119.png)
![](https://storage.googleapis.com/zenn-user-upload/76dc530d683e-20250119.png)
