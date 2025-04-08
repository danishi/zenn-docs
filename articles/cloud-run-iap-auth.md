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
この機能は2024年11月3日現在、まだPrivate Previewのため、利用申請が必要です。
申請方法はGoogle担当者にご相談ください。
:::

:::message
2025年4月8日追記。Pre-GAされました！
https://cloud.google.com/run/docs/securing/identity-aware-proxy-cloud-run?hl=en
コンソールからも設定可能になっています！
![](https://storage.googleapis.com/zenn-user-upload/44d936a84131-20250408.png)
:::

# IAPとは
Identity-Aware Proxy（IAP）はGoogle Cloudが提供するセキュリティ機能で、特定のユーザーのみがリソースにアクセスできるように制御するプロキシサービスです。

これまではCloud RunでIAPによる認証を利用するには**Cloud Load Balancerを構築する必要がありました**が、今回のアップデートにより**Cloud Run単体でIAPを設定できる**ようになりました！

# 手順
今のところはgcloudコマンドからしか設定できないようなので、Cloud Shellで実行します。

## IAP用サービスアカウントの作成と権限付与
まず、IAPに必要なサービスアカウントを作成し、「run.invoker」権限を付与します。

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

## Cloud RunへのデプロイとIAPの有効化

次に、コンテナをCloud Runにデプロイし、IAPを有効にします。

```bash
gcloud alpha run deploy my-iap-service \
--image=us-docker.pkg.dev/cloudrun/container/hello \
--region asia-northeast1 \
--iap
・・・
Service URL: https://my-iap-service-XXXXXXXXX.asia-northeast1.run.app
```

URLにアクセスすると、権限エラーが発生するはずです。

![](https://storage.googleapis.com/zenn-user-upload/13910205255f-20241104.png)

## IAPアクセス権の設定

特定のメールアドレスによるアクセスを許可するために、IAPにアクセス権を付与します。

```bash
gcloud alpha iap web add-iam-policy-binding \
--member=user:<your_email> \
--role=roles/iap.httpsResourceAccessor \
--region asia-northeast1 \
--resource-type=cloud-run \
--service=my-iap-service
```

数秒待ってから再度アクセスすると、以下のようにIAPを通過してCloud Runサービスへアクセスできるようになります。

![](https://storage.googleapis.com/zenn-user-upload/3631f5926841-20241104.png)

IAPの認証を通してCloud Runサービスにアクセスできました！

## アクセス権の削除

アクセス権を削除する場合は、以下のコマンドを実行します。

```bash
gcloud alpha iap web remove-iam-policy-binding \
--member=user:<your_email> \
--role=roles/iap.httpsResourceAccessor \
--region asia-northeast1 \
--resource-type=cloud-run \
--service=my-iap-service
```

## 既存のCloud RunサービスへのIAP設定

既存のCloud Runサービスに対しても、次のコマンドでIAPを有効化または無効化できます。

```bash
gcloud alpha run services update <your_service> --[no-]iap
```

---

今回のアップデートにより、Cloud Load Balancerの構築が不要になり、コスト削減と手軽さが大きく向上しました！
GAが待ち遠しいですね！
