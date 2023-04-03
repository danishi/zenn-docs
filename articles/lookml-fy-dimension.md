---
title: "【小ネタ】LookMLで年度のディメンションを定義する"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["looker", "lookml", "bigquery"]
published: true
---

# やりたかったこと
今年度の売上と前年度の売上の比較などを行う際に、データベース（BigQuery）には年度のカラムがなく、LookML側で年度のディメンションが欲しかった。

# ディメンションの定義

ここでは例として売上テーブルの請求日（`billing_date`）から請求年度を求めます。
salesビューの定義に次のディメンションを定義します。

```:views/sales.view.lkml
  dimension: billing_fy {
    label: "請求年度"
    type: string
    description: "会計年度は４月から翌３月"
    sql: CONCAT(
           CAST(
             IF(EXTRACT(MONTH FROM ${TABLE}.billing_date) >= 4
              , EXTRACT(YEAR FROM ${TABLE}.billing_date)
              , EXTRACT(YEAR FROM ${TABLE}.billing_date) -1
           ) as STRING)
         , '年度'
      )
    ;;
  }
```

この組織は会計年度が４月から翌３月なので、EXTRACTを使って請求月が４～１２月ならば年をそのまま採用し、１～３月ならば年からマイナス1して得られた年をそのデータの発生年度とします。

# 結果
![](https://storage.googleapis.com/zenn-user-upload/5538f7b17e13-20230403.png)
このように請求日から請求年度が得られます。

![](https://storage.googleapis.com/zenn-user-upload/ff423e1072b7-20230403.png)
こんな感じで、フィルター条件に使ったりもできます。

同じようなやり方で半期や四半期のディメンションも定義できます。
