---
title: "Pollyを使ってSlackで回答できる出社アンケートを運用してみた"
emoji: "💼"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["slack", "polly", "コミュニケーション", "リモートワーク"]
published: true
publication_name: "iret"
---

# やりたかったこと
私のチームでは出社とリモートを組み合わせたハイブリットワークで業務を行っています（リモートの比率の方が多いですが）。

たまに出社したときにコミュニケーションの機会になるように、また誰もいなくて寂しくなったりしないように、チームの誰が出社しようとしてるか簡単にわかるような仕組みが作れないかと考えていました。

# やってみたこと
うちではGoogleカレンダーを使っているので、[勤務場所を設定する機能](https://support.google.com/calendar/answer/7638168?hl=ja&co=GENIE.Platform%3DDesktop)を使ってみましたが、このためにみんなの予定を見に行くのはちょっと億劫です。

簡単に出社の意思表示ができ、全員の回答がすぐに見られる場所はどこかなと考えると、普段から使っているSlackがいいと考えました。

Pollyというアンケートを取れるSlackアプリがあり、これを利用することにしました。

https://japangooglecl-xxv7878.slack.com/apps/A04E6JX41-polly

というわけで、Pollyで作った出社アンケートがこんな感じ。
![](https://storage.googleapis.com/zenn-user-upload/924ec4c0a7e5-20240120.png)

曜日のボタンを押すとリアルタイムで自分の名前が投票結果に反映されるので、誰が何曜日に出社しようとしているのか一目瞭然です。
チームのチャンネルがあるので毎週木曜投稿するようにしました。

別に強制というわけではないですが、わりかしみんな出社する曜日を押してくれるようになりました。

# 作り方
最後に作り方を簡単に紹介して終わります。

送りたいチャンネルでスラッシュコマンドを実行してアンケートを作成開始します。
`/polly "アンケートのタイトル"`

基本的な設定を行います。
![](https://storage.googleapis.com/zenn-user-upload/5eb317ef6018-20240120.png)
* `Question type`は複数選択なので`Multiple choice`
* `Enter choices below`は選択肢を改行で区切る
* `Allow comments`は選択肢を選んだあとさらにコメントが残せるようになる
* `Allow multiple vote`は複数投票を可能にする
* `Audience can add choices`は投票に参加する人が選択肢を足せるようにする

![](https://storage.googleapis.com/zenn-user-upload/24de4de8b101-20240120.png)
投稿先のチャンネルが間違っていないことを確認したら、その他の設定を行います。

![](https://storage.googleapis.com/zenn-user-upload/a1902cf48ea6-20240120.png)
* `Non-anonymoous`で記名投票になる
* `In realtime`で投票結果がリアルタイム反映になる
さらに送信スケジュールを設定。

![](https://storage.googleapis.com/zenn-user-upload/405fd2e02a82-20240120.png)
* `Send On`でいつ送るか
* `Time`で何時に送るか
* `Frequency`で頻度を設定
* `Allow Voting for`でいつまで投票できるかを設定

Submitで戻っていき、Sendボタンを押せばスケジューリング完了です。

# 注意点

無料版だと応答回数に制限があるので大規模に使うのは向いてないかも。

https://www.polly.ai/help/slack/response-limits-explained
