---
title: "Google Cloudのリリース情報がいち早く知りたいんじゃ！"
emoji: "🔔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud"]
published: true

---

という方に。

# リリースノートをチェック

https://cloud.google.com/release-notes

RSSフィードも用意されています。

https://cloud.google.com/feeds/gcp-release-notes.xml

## サービスごとのインデックス

一部のサービスを選んでチェックしたいときはここから。

https://cloud.google.com/release-notes/all

こちらも各サービスごとのリリースノートにRSSフィードのURLへのリンクが用意されてます。

https://cloud.google.com/vertex-ai/generative-ai/docs/release-notes?hl=en

## RSSフィードを収集
上記のRSSフィードのURLを活用すれば、例えば[SlackでRSSフィードを購読](https://slack.com/intl/ja-jp/help/articles/218688467-Slack-%E3%81%AB-RSS-%E3%83%95%E3%82%A3%E3%83%BC%E3%83%89%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)していち早くリリースに気づくこともできます。

https://github.com/douglasdollars/gcp-release-notes-feeds

RSSフィードのURLをまとめてるリポジトリもあります。

# BigQuery データセット

リリースノートが格納されたBigQueryのデータセットもあります。

https://console.cloud.google.com/marketplace/product/bigquery-public-datasets/google_cloud_release_notes

若干遅れて更新されるので即時性はないですが。

```sql
SELECT 
  product_id,
  product_name,
  product_version_name,
  description,
  release_note_type,
  published_at
FROM `bigquery-public-data.google_cloud_release_notes.release_notes` 
WHERE
      product_name = 'Vertex AI'
ORDER BY
  published_at DESC
;
```

![](https://storage.googleapis.com/zenn-user-upload/a8a0dcd8bee5-20241129.png)

このようにリリース情報を取得できるので、結果をGeminiに読み込ませて月間リリースサマリなどを作ってみるのも面白いかもしれません。
