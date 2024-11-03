---
title: "Cloud RunがIAPでよりお手軽に認証できるようになりました！"
emoji: "🔓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud","cloudrun","iap"]
published: true
publication_name: "iret"
---

:::message
注意
この機能は2024/11/03現在まだPrivate Previewのため利用申請が必要です。
:::
# IAPとは
Identity-Aware Proxy（IAP）は、Google Cloudで提供されているセキュリティ機能で、特定のユーザーだけが特定のリソースへアクセスできるようにするプロキシサービスです。

これを利用することでCloud Runの前段でIAPによる認証を付与することは以前からできましたが、そのためには別途Cloud Load Balancingを構成する必要がありました。
それが、Cloud Runサービス単体に対してできるようになりました！

# 手順
今のところはgcloudコマンドからしか設定できないようなので、Cloud Shellで実行します。

## IAPのサービスアカウントを作成、「run.invoker」権限を付与

```bash
export PROJECT_ID=<your_project_id>
gcloud beta services identity create \
--service=iap.googleapis.com \
--project=$PROJECT_ID
```

```bash
export PROJECT_NUMBER=<your_project_number>
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member="serviceAccount:service-$PROJECT_NUMBER@gcp-sa-iap.iam.gserviceaccount.com" \
--role="roles/run.invoker"
```

## IAPを有効にしてコンテナをCloud Runにデプロイ

```bash
gcloud alpha run deploy my-iap-service \
--image=us-docker.pkg.dev/cloudrun/container/hello \
--region asia-northeast1 \
--iap
・・・
Service URL: https://my-iap-service-XXXXXXXXX.asia-northeast1.run.app
```

そのままアクセスすると権限エラーになります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/2fe689e0-1505-5ace-a336-0756fb5c7b32.png)

## IAPにアクセス権を付与

```bash
gcloud alpha iap web add-iam-policy-binding \
--member=user:<your_email> \
--role=roles/iap.httpsResourceAccessor \
--region asia-northeast1 \
--resource-type=cloud-run \
--service=my-iap-service
```

実行後少し待って再アクセスすると。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/f7206670-0b76-696c-381f-e1bd892aee60.png)

IAPの認証を通してCloud Runサービスにアクセスできました！

アクセスを削除するにはこのようにします。

```bash
gcloud alpha iap web remove-iam-policy-binding \
--member=user:<your_email> \
--role=roles/iap.httpsResourceAccessor \
--region asia-northeast1 \
--resource-type=cloud-run \
--service=my-iap-service
```

## 既存のCloud RunサービスをIAP有効化/無効化
```bash
gcloud alpha run services update <your-service> --[no-]iap
```

---

ロードバランサーを構築しなくてよい分コスト削減と手軽さにつながる嬉しいアップデートですね！
GAが待ち遠しいです！
