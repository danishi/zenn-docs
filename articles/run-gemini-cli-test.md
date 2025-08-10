---
title: "Google Cloud Next Tokyo '25で発表された、Gemini CLI GitHub Actionsを試してみた"
emoji: "☁️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["githubactions","googlecloudnext","gemini","geminicli","生成ai"]
published: true
publication_name: "iret"
---

先日開催された、[Google Cloud Next Tokyo '25](https://www.googlecloudevents.com/next-tokyo/)に **Next Tokyoアンバサダー**として参加してきました！

https://iret.media/163135

本記事では、2日目の基調講演で発表された、Gemini CLI GitHub Actionsを実際に試してみた結果を共有します。

![](https://storage.googleapis.com/zenn-user-upload/775c777bc2e8-20250810.png)

# Gemini CLI GitHub Actionsとは

https://cloud.google.com/blog/ja/topics/developers-practitioners/introducing-gemini-cli-github-actions

Gemini CLI GitHub Actionsは、[Gemini CLI](https://cloud.google.com/blog/ja/topics/developers-practitioners/introducing-gemini-cli)をGitHub Actionsで利用できるようにしたものです。
これにより、CI/CDパイプラインでGemini CLIの活用がより簡単になりました！

Gemini CLIと同じく、OSSとして公開されており、GitHub上でソースコードを確認できます。

https://github.com/google-github-actions/run-gemini-cli

アクションの中身としては、Gemini CLIのインストールと、呼び出し時に渡されたパラメータで認証を行い、同じく渡されたプロンプトをYOLO（You Only Look Once）モードで実行するというものになっています。

# 試してみた
実際に個人のGitHubリポジトリで試してみました。

https://github.com/danishi/run-gemini-cli-test

公式で提供されている[サンプル](https://github.com/google-github-actions/run-gemini-cli/blob/9da55331df5ce66d7b676a3a134e3abe13b7d00c/.github/workflows/gemini-cli.yml)を参考に、以下のようなワークフローを作成しました。

https://github.com/danishi/run-gemini-cli-test/blob/main/.github/workflows/gemini-cli.yml

次のようなイシューを作り、`@gemini`でメンションをすると、ワークフローが実行され、Gemini CLIが呼び出されます。

https://github.com/danishi/run-gemini-cli-test/issues/2

うまくいくと、イシューへのコメントを行い、命令に応じてプルリクエストが作成されます。

https://github.com/danishi/run-gemini-cli-test/pull/3

プルリクエスト内でも追加の指示が可能です。

# 認証

https://github.com/danishi/run-gemini-cli-test/blob/main/.github/workflows/gemini-cli.yml#L192-L209

サンプルリポジトリでは、[Google AI Studio](https://aistudio.google.com/)で発行したAPIキーをリポジトリシークレットに入れて認証していますが、他にも認証方法が用意されています。

:::message
APIキーを利用することでGoogle AI Studioでもトークン利用料に応じて課金が発生する可能性があるため、ご注意ください。
:::

エンタープライズで利用する場合は、Vertex AIを利用したほうがより安全でスケーラブルです。

こちらの記事を参考に、Workload Identityプールとサービスアカウントを作成。

https://qiita.com/danishi/items/796042cfca5b1df4d1ca#google-cloud%E3%81%AE%E8%A8%AD%E5%AE%9A

```yaml
        with:
          # gemini_api_key: '${{ secrets.GEMINI_API_KEY }}'
          gcp_workload_identity_provider: '${{ secrets.GCP_WIF_PROVIDER }}'
          gcp_project_id: '${{ secrets.GOOGLE_CLOUD_PROJECT }}'
          gcp_location: '${{ vars.GOOGLE_CLOUD_LOCATION }}' # ex: 'global'
          gcp_service_account: '${{ secrets.SERVICE_ACCOUNT_EMAIL }}'
          use_vertex_ai: 'true'
          # use_gemini_code_assist: '${{ vars.GOOGLE_GENAI_USE_GCA }}'
```

例えば、このように変数に設定することでVertex AIでの認証に切り替えることが可能です。

# まとめ
Gemini CLIでは個人のローカル環境でターミナルを経由してペアプログラマーのようにGeminiを実行することができますが、Gemini CLI GitHub Actionsを利用することで、GitHub上でまるで**チームメイトのように**Geminiと共同作業を行うことができます！

Geminiを使ったバイブコーディングがさらに加速しますね！
