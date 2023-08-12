---
title: "VS CodeでAWS SAMのローカルデバッグをやってみる"
emoji: "🐛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","python","lambda","sam","vscode"]
published: true
---

# 準備
既にAWS SAMアプリがある場合は不要ですが、ない場合はまずは事前準備。

## 適当なAWS SAMアプリを用意する

こちらのHello Worldアプリを作りましょう（デプロイは必要ありません）。
https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html

手順に従って`sam init`が終わったら、ターミナルで以下を実行。

```bash
sam local start-api
```

別のターミナルから呼び出せれば準備完了。

```bash
curl http://127.0.0.1:3000/hello
{"message": "hello world"}
```

# VS Codeでデバッグ

## AWS Toolkitのインストール
AWS SAMをVS Codeでデバッグするために必須の拡張機能となります。
https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-toolkit-vscode

## デバッグ構成の追加
デバッグ構成を追加します。
![](https://storage.googleapis.com/zenn-user-upload/880d29a98e80-20230812.png)

すると下記のように構成ファイルが追加されます。

```json:.vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "aws-sam",
            "request": "direct-invoke",
            "name": "sam-app:HelloWorldFunction (python3.9)",
            "invokeTarget": {
                "target": "template",
                "templatePath": "${workspaceFolder}/template.yaml",
                "logicalId": "HelloWorldFunction"
            },
            "lambda": {
                "payload": {},
                "environmentVariables": {}
            }
        },
        {
            "type": "aws-sam",
            "request": "direct-invoke",
            "name": "API sam-app:HelloWorldFunction (python3.9)",
            "invokeTarget": {
                "target": "api",
                "templatePath": "${workspaceFolder}/template.yaml",
                "logicalId": "HelloWorldFunction"
            },
            "api": {
                "path": "/hello",
                "httpMethod": "get",
                "payload": {
                    "json": {}
                }
            },
            "lambda": {
                "runtime": "python3.9"
            }
        }
    ]
}
```

単なるLambda関数として実行するパターンとAPIとして実行するパターンの二通りの構成が用意されています。

直接ファイルを作成してもかまいません。  
各パラメータの詳細な説明はこちら。
https://docs.aws.amazon.com/ja_jp/toolkit-for-vscode/latest/userguide/serverless-apps-run-debug-config-ref.html

## デバッグの実行

それではブレークポイントを置いてデバッグを実行します。２つあるうちのどちらでも構いません。  
事前準備の時に試したように`sam local start-api`をやっておく必要もありません。

![](https://storage.googleapis.com/zenn-user-upload/8224de196190-20230812.png)

ブレークポイントで止まれば成功です。
