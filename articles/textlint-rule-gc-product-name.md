---
title: "Google Cloudプロダクト名をチェックするtextlintルールを公開しました！"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["textlint","googlecloud","文章校正","個人開発","OSS"]
published: true
publication_name: "iret"
---

# tl;dr

GW期間を利用してGoogle Cloudに関連するプロダクトやAPIの表記を校正するためのtextlintルールをOSSで作成、公開しました！

https://www.npmjs.com/package/textlint-rule-gc-product-name

Google Cloudに関する記事を書くときに、是非使ってみてください！

利用方法は一般的なtextlintルールと同じなので割愛します。
textlintを使えばコマンドラインやVSCode拡張機能でルールのチェックができます。

![](https://storage.googleapis.com/zenn-user-upload/eb68330d0615-20240502.png)

このようにスペースを入れるところやキャピタリゼーションが異なるとエラーにしてくれます。
スペルミスなどのtypoは検出できませんが、他のスペルチェックのルールと組み合わせることである程度実現できると思います。

# 作り方解説

ソースコードはOSSとして公開していますが、せっかくなのでどのように作っているか解説します。

https://github.com/danishi/textlint-rule-gc-product-name

## プロダクト情報の取得

まず、このルールを作る前にどのようにマスタとなるGoogle Cloudのプロダクト名一覧を取得しようか考えました。
手動で全てリストアップするのは避けたいですし、サービス名は追加、更新、削除されるので自動化を行いたいです。

AWSだとこういった[ズバリなAPI](https://aws.amazon.com/api/dirs/items/search?item.directoryId=aws-products)が用意されているみたいですが、探した限り見つからなかったので以下3点を情報のリソース源としました。

* [手動でリストアップしたプロダクト名一覧](https://github.com/danishi/gc-service-list-api/blob/main/data/products.json)
* [Google Cloudの製品ページ](https://cloud.google.com/products?hl=en)
* [GoogleのAPI一覧を取得できるAPI](https://www.googleapis.com/discovery/v1/apis)

これらを統合し1つのJSONファイルとして返却できるAPIをまずは作成しました。それがこちらです。

https://danishi.github.io/gc-service-list-api/products.json

リソースから必要な情報を収集し統合する処理をGitHub Actionsで行い、GitHub PagesでなんちゃってAPIとして公開しています。
こちらもOSSで作られているので興味があれば。

https://github.com/danishi/gc-service-list-api

## textlintルールの作成

textlintルールを作って公開するのは初めての経験だったのですが、こちらの記事を参考にし作ることができました！

https://efcl.info/2016/12/14/create-textlint-rule/

ルールの元となる[ファイル](https://github.com/danishi/textlint-rule-gc-product-name/blob/main/src/terms.json)は前述の[API](https://danishi.github.io/gc-service-list-api/products.json)をGitHub Actionsから呼び出すことで生成できるようにしました。
ルールを適用する部分は[Gemini](https://gemini.google.com/app)と相談しながら作りました。

---

https://dev.classmethod.jp/articles/create_aws_textlint_rule/

https://aadojo.alterbooth.com/entry/2023/12/23/090000

AWSやAzureのルールは既に有志で作られていますがGoogle Cloudは調べた限りなさそうでした。
パートナーとしてプロダクト名を間違えるような失礼のないように、弊社CTO([@kaz_goto](https://twitter.com/kaz_goto)）からのリクエストで作成しました😤

https://github.com/danishi/textlint-rule-gc-product-name

よければスターやコントリビュートいただけると励みになります。
