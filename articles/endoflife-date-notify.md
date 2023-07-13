---
title: "endoflife.dateとLambdaを使ってSlackに自動でプロダクトのEOLを通知する"
emoji: "⌛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["eol","python","lambda","aws"]
published: true
---

# やりたいこと

[endoflife.date](https://endoflife.date/)にはさまざまなプロダクトのEOL情報がまとめられています。
また、その情報を取得できる[API](https://endoflife.date/docs/api)も用意されています。

普段よく使うプロダクトのEOLが気づいたら間近に迫って大慌てで対応することにならないように、毎日自動チェックを行いその結果をSlack通知できるようにしてみようと思いました。

# 手順

## 事前準備
通知用のSlackチャンネルとSlack Appを作ってIncoming Webhooks URLを作っておきます。

![](https://storage.googleapis.com/zenn-user-upload/bfb914e11c8c-20230608.png)

## Lambdaを作成

以下のコードでPython Lambdaを作ります。

```python:lambda_function.py
import requests
from http import HTTPStatus
import json
import sys
from datetime import datetime, timedelta
import logging
import os

logger = logging.getLogger()
logger.setLevel(logging.INFO)

EOL_URL = 'https://endoflife.date/'
EOL_API_BASE_URL = 'https://endoflife.date/api/'

WEB_HOOK_URL = os.getenv('WEB_HOOK_URL')

# EOLを確認したいプロダクトとバージョン
NOTIFICATION_PRODUCTS_VERSION = {
    'vue': '2',
    'nuxt': '2',
    'php': '8.0',
    'python': '3.7',
    'laravel': '9',
    'amazon-rds-mysql': '5.7',
    'amazon-linux': '2',
}

# EOLの何日前に通知する
NOTIFICATION_BEFORE_DEADLINE_DAYS = [
    1,
    30,
    60,
    180,
    365,
]


def send_slack(text):
    # 指定されたテキストをSlackに送信
    headers = {
        'Accept': 'application/json',
    }
    response = requests.post(WEB_HOOK_URL, headers=headers, data= json.dumps({
        'text': text
    }))
    logger.info(response.text)


def fetch_end_of_life_date(product, version = None):
    # EOL日付を取得する。
    # 指定された製品とバージョンに応じて、エンドポイントのURLを構築し、リクエストを送信します。
    url = EOL_API_BASE_URL + product
    if version != None :
        url = url + '/' + version + '.json'
    else:
        url = url + '.json'
    response = requests.get(url)
    
    if response.status_code == HTTPStatus.OK:
        logger.info(response.text)
        return response.text
    else:
        return None


def notify_product_version_deadline_for_slack():
    # 指定された製品とバージョンのEOL日付と期限を取得し、Slackに通知  
    today = datetime.now()
    logger.info(today)
    
    for product,version in NOTIFICATION_PRODUCTS_VERSION.items():
        res = fetch_end_of_life_date(product, version)
        if res != None:
            product_json = json.loads(res)

            slack_notify = None
            eol = product_json["eol"]
        try:
            deadline_date = datetime.strptime(eol, '%Y-%m-%d')
        except:
            continue
        
        # 設定したEOLまでの日数に達したらメッセージを通知
        for day in NOTIFICATION_BEFORE_DEADLINE_DAYS:
            notify_date = deadline_date - timedelta(days=day)
            
            if today.date() == notify_date.date():
                slack_notify = {
                  'product': product,
                  'version': version,
                  'day': day,
                  'support_term': (deadline_date - datetime.now()).days
                }
                break
        
        if slack_notify != None and deadline_date >= today:
            slack_text = create_notify_message(product_json, slack_notify)
            send_slack(slack_text)


def create_notify_message(product_json, slack_notify):
    # 通知メッセージを作成
    slack_text = ':warning:*' + slack_notify["product"] + slack_notify["version"] + '*\n'
    support_term = slack_notify["support_term"] + 1
    slack_text += 'EOL(' + product_json["eol"] + ')まであと' + str(support_term)  + '日\n'
    slack_text += EOL_URL + slack_notify["product"]
    
    logger.info(slack_text)
    
    return slack_text


def lambda_handler(event, context):
    logger.info(json.dumps(event))

    notify_product_version_deadline_for_slack()

    return
```

タイムアウトは30秒ほどに伸ばしておいた方がいいでしょう。

`NOTIFICATION_PRODUCTS_VERSION`と`NOTIFICATION_BEFORE_DEADLINE_DAYS`変数は任意のリテラルをセットしてください。

環境変数にはIncoming Webhooks URLを設定します。

![](https://storage.googleapis.com/zenn-user-upload/7509c54a6eb8-20230608.png)

また、Lambdaにデフォルトで入ってないライブラリの`requests`を使うのでレイヤーを追加します。
`arn:aws:lambda:ap-northeast-1:770693421928:layer:Klayers-p310-requests:2`

![](https://storage.googleapis.com/zenn-user-upload/3a5362ddf8b8-20230608.png)

## テスト実行

通知日付を調節して、このように通知が飛べば成功です。
![](https://storage.googleapis.com/zenn-user-upload/b14cc208b165-20230608.png)

## 定期実行
Amazon EventBridgeルールを設定して、1日1回所定の時間に実行されれば完成です。

# 参考記事

https://blog.k-bushi.com/post/tech/programming/python/eol-notify-python/
