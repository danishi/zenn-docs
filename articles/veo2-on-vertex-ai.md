---
title: "Vertex AIで動画生成モデルVeo 2を使ってみる"
emoji: "🎭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud","veo","veo2","vertexai","生成ai"]
published: true
publication_name: "iret"
---

# Veo 2とは

https://deepmind.google/technologies/veo/veo-2/

GoogleのAI研究部門であるDeepMind社が開発した、リアルな動きと最大4Kの高品質出力を備えた動画生成モデルです。

以下は公式サイトで紹介されているVeo 2によって生成された動画です。

https://www.youtube.com/watch?v=gztv6XYUzTM

https://www.youtube.com/watch?v=e-uf510bXH0

https://www.youtube.com/watch?v=oMZ7YNaSfn8

# Vertex AIでVeo 2を使う

:::message
Veo 2は一般提供されていますがホワイトリストに登録された承認ユーザーのみが利用できます。利用にはGoogle Cloud アカウント担当者にお問い合わせください。
:::

## Vertex AI StudioでVeo 2を使う

Vertex AI Studioを利用すればブラウザでVeo 2を試すことができます。
動画を説明するプロンプトを入力するだけで30秒ほどで動画が生成されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/54c8dc52-7813-40b0-bde7-d6fdee29eab4.png)

https://www.youtube.com/watch?v=CeWpOxBbSck

プロンプトは日本語でも通りますが英語のほうがより意図通りのものが生成されやすいように感じました。

### 画像から動画を生成する

指定のアスペクト比の画像から動画を生成することも可能です。
画像生成モデルのImagen 3で作成したこちらの画像を動かしてみました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/b5fc036d-13ff-47d9-913c-f16919a485a0.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96507/b400bfb5-de07-4ae6-8478-fe7a7edd7d13.png)

https://www.youtube.com/watch?v=d84Z90S6j38

試した感じでは元画像をダイナミックにコントロールするにはコツがいりそうでした。

生成した動画の最終カットを次のようにffmpegコマンドで取り出して

`ffmpeg -sseof -1 -i MyMovie1.mp4 -update 1 -q:v 1 last_frame.png`

続きの動画を生成し、最終的にffmpegコマンドで結合すれば長い動画も作ることができますね。

`ffmpeg -f concat -safe 0 -i videos.txt -c copy output.mp4`

```txt:videos.txt
file 'MyMovie1.mp4'
file 'MyMovie2.mp4'
```

## Google Gen AI SDKでVeo 2を使う
PythonライブラリのGoogle Gen AI SDKでもVeo 2を使えるのでGoogle Colabで試してみました。

@[gist](https://gist.github.com/danishi/5e72b580ab3b7464a6db20ff647c0f12)

## 価格
1秒の動画生成あたり **$0.50** のコストが発生します。
現在の最大の8秒動画生成では$4かかる計算になります。

https://cloud.google.com/vertex-ai/generative-ai/pricing?hl=en

# 参考

https://cloud.google.com/vertex-ai/generative-ai/docs/video/generate-videos?hl=en

https://cloud.google.com/vertex-ai/generative-ai/docs/video/video-gen-prompt-guide?hl=en

https://cloud.google.com/vertex-ai/generative-ai/docs/model-reference/veo-video-generation?hl=en
