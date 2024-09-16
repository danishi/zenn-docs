---
title: "スタンプ1つでURLを要約！Geminiを活用したSlackbotを作りました"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud","python","cloudfunctions","slack","gemini"]
published: true
publication_name: "iret"
---

# TL;DR

こんなものを作りました。

![demo](https://github.com/user-attachments/assets/d7009994-3f27-4c8b-af33-2efe1f46846c)

所定のスタンプを押されたメッセージにURLがあると内容を読み込んでGeminiで要約してくれます。
SlackbotとCloud Run 関数 第1世代（旧名：Cloud Functions）のバックエンドで実現しています。

日々RSSやXで大量に上がってくる様々なニュースブログ記事の概要を素早く掴んで、ついでにSlackで簡単にシェアできるようにしたかったので作りました。
英語の記事も日本語に訳して要約してくれるので助かります。

ゼロスケールするのでお財布にも優しくできています。

# 作り方

ソースコードはGitHubで公開しています。

https://github.com/danishi/slack-url-summarize-gemini

作り方はまず、こちらのマニュフェストファイルを使いまわしてSlackbot用にSlackアプリを作りワークスペースにインストールします。

https://github.com/danishi/slack-url-summarize-gemini/blob/main/manifest.json

botの呼び出しエンドポイントURLに設定するためのCloud Run 関数 第1世代を作成します。

https://github.com/danishi/slack-url-summarize-gemini/blob/main/main.py

スラッシュコマンドも使えるようにしたかったのですが、Slackbotの3秒ルールを突破するためのlazy listenersが対応していなかったので断念。

https://github.com/slackapi/bolt-python/issues/687

必要なライブラリはこちら。

https://github.com/danishi/slack-url-summarize-gemini/blob/main/requirements.txt

また設定ファイルとして下記を参考に`.env`ファイルを一緒にデプロイしてください。

https://github.com/danishi/slack-url-summarize-gemini/blob/main/.env.example

設定としては、Slackbotのトークンやシークレット情報と、処理のトリガー、処理中の目印となるSlack絵文字の名前（絵文字は[こういうの](https://emoji-gen.ninja/result?text=URL%0A%E8%A6%81%E7%B4%84&color=3AB0C7FF&back_color=00000000&font=notosans-mono-bold&size_fixed=false&align=center&stretch=true&public_fg=true&locale=ja)を使って作るとよいでしょう）。
あとはGoogle CloudのプロジェクトIDなどです。

Vertex AIを利用するので関数のサービスアカウントに適切に権限を設定してください。

これらがデプロイできていれば動くはずです。

# 応用として

作ってみて、この仕組みはプロンプトを変えればURLの要約以外にも役立ちそうだなと思いました。

例えば、[RSSを拾って集めるようなチャンネル](https://slack.com/intl/ja-jp/help/articles/218688467-Slack-%E3%81%AB-RSS-%E3%83%95%E3%82%A3%E3%83%BC%E3%83%89%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)で、自社のブログフィードを集めて記事のレビュー自動化に使うとか。

Slack上で文書の誤字脱字チェックのレビューに使うとか。

https://github.com/danishi/slack-url-summarize-gemini/blob/main/main.py#L67-L75

Vertex AIならではの機能としてGoogle検索や自前のデータストアへ関単にグラウンディングができるので、文書のソースチェックにも使えそうです。

生成AIはユーモアも出せるので失敗や悩みの書き込みに対してのアドバイスとかにも使えるかもしれません。

スタンプを押すだけという手軽さも相まっていいものが作れればワークスペースで利用が広がって行きそうな気がします！

OSSなので、是非ご自由に改変して使ってみてください！

最後にこれを作るヒントをくださった[@kaz_goto](https://twitter.com/kaz_goto)ありがとうございます！
