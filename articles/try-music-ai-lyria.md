---
title: "Googleの音楽生成AI Lyriaを試してみた！"
emoji: "🎶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud","lyria","Lyria2","vertexai","生成ai"]
published: true
publication_name: "iret"
---

# Lyria（リリア）とは

https://deepmind.google/models/lyria/

今年4月のGoogle Cloud Next '25の基調講演で発表された音楽生成のためのAIモデルです。

https://www.youtube.com/live/Md4Fs-Zc3tg?si=pZXFbGjSV8Kq1AXq&t=1865

私は現地ラスベガスで基調講演を聞いていたのですが、Lyriaの作ったミュージックデモが流れたあとのなんともいえない間が今も記憶に強く残っています。

公開からしばらくは申請が必要なプライベートプレビューだったのですが、Google I/O 2025の開催と同時に**Lyria 2**となって**GA**されました！

# Lyria 2で音楽を生成してみる

https://cloud.google.com/vertex-ai/generative-ai/docs/music/generate-music

Lyria 2モデルは現在のところはGoogle CloudのVertex AI StudioかAPIを直接呼び出すことで実行できます。

[プロンプトガイドライン](https://cloud.google.com/vertex-ai/generative-ai/docs/music/music-gen-prompt-guide)にあったこちらのプロンプトをVertex AI Studioで試してみます。

>An energetic (mood) electronic dance track (genre) with a fast tempo (tempo) and a driving beat (rhythm), featuring prominent synthesizers (instrumentation) and electronic drums (instrumentation). High-quality production (production quality).	
>訳：シンセサイザー（楽器）とエレクトロニックドラム（楽器）をフィーチャーした、速いテンポ（テンポ）と力強いビート（リズム）を備えた、エネルギッシュな（ムード）エレクトロニックダンストラック（ジャンル）。高品質なプロダクション（プロダクションクオリティ）。

![](https://storage.googleapis.com/zenn-user-upload/375866c2e2fb-20250522.png)

1、2分で30秒の曲が生成されます。一度のプロンプトで最大4つを生成できるUIになっています。作成した曲はダウンロードできます。
このような曲ができました。

https://soundcloud.com/danishi-91070654/lyria-2-sample?si=47a6acfcc2bf4ff5a4d14055a4030c99&utm_source=clipboard&utm_medium=text&utm_campaign=social_sharing

他にレトロゲーム風の8ビットミュージックも試してみました。

https://soundcloud.com/danishi-91070654/lyria-2-sample-8-bit-boss-bgm

このようにプロンプトからインストゥルメンタルミュージックを生成できますが、歌やコーラス、効果音の類は試した限り難しそうでした。

公式から出てるサンプルもありました。

https://www.youtube.com/watch?v=kSmy1iPmaC8

https://www.youtube.com/watch?v=Dza25xkWEYc

# 価格
モデルの利用料に関する情報は今のところ見つけることができませんでした。

https://cloud.google.com/vertex-ai/generative-ai/pricing?hl=en

# 感想
30秒までしか生成できず続きを作ることもできないので現状は使いどころが難しいですが、同じくGoogle I/O 2025で発表された動画生成AIモデルのVeo 3やAI映像制作ツールのFlowと組み合わさることで強力な進化をしそうです。

また、[Lyria RealTime](https://deepmind.google/models/lyria/realtime/)というインタラクティブに音楽を生成し続けることができるものもすでに存在するので、今後のアップデートに期待ですね！
