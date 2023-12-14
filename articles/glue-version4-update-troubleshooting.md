---
title: "AWS Glueのバージョン4へのアップデートでのエラー対処"
emoji: "💣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "glue", "pyspark"]
published: true
publication_name: "iret"
---

# エラー内容
https://docs.aws.amazon.com/glue/latest/dg/glue-version-support-policy.html

既存のバージョン2のGlueジョブが2024年1月末でサポート終了になるのでアップデートしたところ見慣れないエラーで失敗してしまった。

```
An error occurred while calling o165.save. File already exists:s3://my-bucket/data/part-00051-50c0f3a2-eba1-521b-1b0a-c6652c129376-a000.csv
```

CSVの書き込みエラーっぽいが、CloudWatch Logsを見ると前段で別のエラーが出ていることが分かった。

```
23/12/13 15:41:51 ERROR Utils: Aborting task
org.apache.spark.SparkUpgradeException: You may get a different result due to the upgrading to Spark >= 3.0:  Fail to format it to '2019-11-29 00:00:00' in the new formatter. You can set spark.sql.legacy.timeParserPolicy to LEGACY to restore the behavior before Spark 3.0, or set to CORRECTED and treat it as an invalid datetime string.        
	at org.apache.spark.sql.errors.QueryExecutionErrors$.failToFormatDateTimeInNewFormatterError(QueryExecutionErrors.scala:1076) ~[spark-catalyst_2.12-3.3.0-amzn-1.jar:3.3.0-amzn-1]
以下略
```

# 対処

CSV出力時のオプション指定でこのように（`.option("dateFormat", "yyyy-MM-dd HH:mm:ss")`）日付の書式設定していたジョブだけがエラーになっていた。
エラー文にもあるように3.0以降はこの書式がそのままでは使えないようなので、下記のように古いパーサーを有効化することで解消された。

```python
spark.conf.set("spark.sql.legacy.timeParserPolicy", "LEGACY")
```

# 参考

https://stackoverflow.com/questions/62602720/string-to-date-migration-from-spark-2-0-to-3-0-gives-fail-to-recognize-eee-mmm
