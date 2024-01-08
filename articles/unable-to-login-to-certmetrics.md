---
title: "CertMetricsにアカウント重複でログインできなくなった対処メモ"
emoji: "🚫"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud", "gcp", "資格"]
published: true
---
https://cp.certmetrics.com/google/en/login

最近できたGoogle Cloudの認定資格ポータルの[CertMetrics](https://support.google.com/cloud-certification/answer/14093796?hl=JA)ですが、ある日以下のようなエラーでアカウントにログインできなくなってしまいました。

`the specified email is used by multiple certmetrics accounts`

以前はログインできていたので原因は何かと考えたところ、英語版の試験を受けるために[webassessor](https://www.webassessor.com/)の英語版アカウントを作成したのが原因ではと考えました（エラーメッセージも登録メールアドレスが重複してそうな感じなので）。

とはいえ、原因にあたりが付いたところでこちらでは何もできることはないので大人しく[サポート](https://support.google.com/cloud-certification/gethelp?hl=en)に問合せします。

今の状況にあわせて選択肢を進めて、最終的な問合せの詳細欄に「アカウントにログインできない、メールアドレスはこれで、日本語版と英語版のアカウントを持ってるのが原因だと思うんだけど～」的なことをGoogle翻訳で問い合わせたところ、速攻で返信がきて無事ログインできるようになりました。

同じことで困ってる人の助けになれば幸いです。
