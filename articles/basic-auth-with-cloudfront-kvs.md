---
title: "CloudFront FunctionsのBasic認証で認証情報をCloudFront KeyValueStoreで管理する"
emoji: "🔓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "cloudfront", "cloudfrontfunction", "cicd", "githubactions"]
published: true
---

# はじめに
先日リリースされたばかりの「CloudFront KeyValueStore」。

https://aws.amazon.com/jp/blogs/aws/introducing-amazon-cloudfront-keyvaluestore-a-low-latency-datastore-for-cloudfront-functions/

これにより、CloudFront Functions間で共有できる**Key-valueストア**が使えるようになりました。

用途としては、URLの書き換えパターンやリダイレクトの遷移先を保存したり、オリジンを切り替えたり、アクセス拒否の設定を保存したりといったことに使えそうです。
最大5MBのデータを保存でき、キーの最大サイズは512バイト、値の最大サイズは1KBでJSON文字列も扱えるのでちょっとしたDynamoDBみたいな使い方もできそうです。

これまでCloudFront Functionsにハードコードするしかなかったような情報も外出しできるので用途の幅が広がりそうです。


本記事では、古くはLambda@Edge。最近ではCloudFront Functionsで実装されることが多い、CloudFrontディストリビューションへのBasic認証付与の関数。
その**認証情報をCloudFront KeyValueStoreで管理**してみます。

# KeyValueStoreを作成

![](https://storage.googleapis.com/zenn-user-upload/30311bca2ae5-20231122.png)

始めに認証情報を保存しておくKeyValueStore（KVS）を作成します。

![](https://storage.googleapis.com/zenn-user-upload/1a0bb0fb4c0e-20231122.png)

任意の名前で作成します。初期の格納データをS3に置いたファイルからインポートして作成もできます。

![](https://storage.googleapis.com/zenn-user-upload/9e4d7478ce5b-20231122.png)

しばらく待つとKVSができるのでキーと値のペアを登録します。この後に紹介するコードを使うならキー名を合わせておいて下さい。
この画面からKVSとCloudFront Functionsの紐づけも確認できます。

![](https://storage.googleapis.com/zenn-user-upload/fc2b98c2cdb4-20231122.png)

保存すると、確定されひとつ前の画面に反映されます。

# 関数を作成しKVSに紐づける

次はBasic認証のためのCloudFront Functionsを作成します。

![](https://storage.googleapis.com/zenn-user-upload/15043eefd20c-20231122.png)

関数ランタイムは`cloudfront-js-2.0`である必要があります。

![](https://storage.googleapis.com/zenn-user-upload/c8e731a4a166-20231122.png)

作成した関数に先ほど作ったKVSを紐づけます。

![](https://storage.googleapis.com/zenn-user-upload/3c011a1ef7eb-20231122.png)

紐づけができるとこのようになります。１つの関数に紐づけられるKVSは１つだけです。
赤枠部分にKVSのIDが表示されるのでこれを関数のコードで使います。
また、サンプルコードも出てくるので迷ったらこちらを参考にしましょう。

関数に記述するコードはこちらです。

@[gist](https://gist.github.com/danishi/968cd415536bd2f7d4c84cd04e05bc96)

`<KEY_VALUE_STORE_ID>`を取得したKVSのIDに差し替えましょう。

コードを写して保存したら、関数のテストを実行します。
特にエラーなく401になれば問題ないでしょう。

最後に関数を発行して、任意のCloudFrontディストリビューションに紐づけます。
イベントタイプはViewer request。
キャッシュビヘイビアは発火させたいものを選びましょう。例えば`/api/*`のビヘイビアで発火させたくないなどあれば選ばないように。

![](https://storage.googleapis.com/zenn-user-upload/6e0d67e50d4b-20231122.png)

紐づけ後、アクセスしたディストリビューションでBasic認証ダイアログが表示され、KVSに保存した認証情報の入力でアクセスができれば完成です。
