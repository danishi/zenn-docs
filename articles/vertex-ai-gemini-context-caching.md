---
title: "Vertex AI GeminiのContext cachingでプロンプトのコストとパフォーマンスを改善"
emoji: "🤑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud", "vertexai", "gemini", "生成ai", "python"]
published: true
publication_name: "iret"
---

# はじめに

昨日Vertex AIのアップデートがいくつか発表されました！

https://cloud.google.com/blog/ja/products/ai-machine-learning/vertex-ai-offers-enterprise-ready-generative-ai/

その中で「[コンテキスト キャッシュ](https://cloud.google.com/vertex-ai/generative-ai/docs/context-cache/context-cache-overview)」が個人的に気になったので調べてみました！
>本日より、Gemini 1.5 Pro モデルと Gemini 1.5 Flash モデルでコンテキスト キャッシュをパブリック プレビューで展開し、お客様は Gemini の大規模なコンテキスト ウィンドウを効率的に活用できるようになります。コンテキストが長くなると、コンテキストの長いアプリケーションの応答を取得するのにコストがかかるだけでなく、パフォーマンスが遅くなり、運用環境へのデプロイが困難になる場合もあります。Vertex AI コンテキスト キャッシュは、頻繁に使用されるコンテキストのキャッシュ データを活用することで、コストを 76% 削減させるのに役立ちます。現在、Google はコンテキスト キャッシュ API を提供する唯一のプロバイダです。

# 使ってみる

こちらのコードをベースに試していきます。

https://ai.google.dev/gemini-api/docs/caching?hl=ja&lang=python

## 事前準備

[Google AI Studio](https://aistudio.google.com/app/apikey?hl=ja)でAPIキーを発行して環境変数にセット、SDKをインストールします。

```bash
export API_KEY=<YOUR_API_KEY>

pip install -q -U google-generativeai
```

## コンテキスト キャッシュを使わない問い合わせ

まずは普通に動画を使ったマルチモーダル問い合わせをやってみます。

動画データは[こちら](https://storage.googleapis.com/generativeai-downloads/data/Sherlock_Jr_FullMovie.mp4)で45分ほどある[無声映画](https://ja.wikipedia.org/wiki/%E3%82%AD%E3%83%BC%E3%83%88%E3%83%B3%E3%81%AE%E6%8E%A2%E5%81%B5%E5%AD%A6%E5%85%A5%E9%96%80)です。

```python
import google.generativeai as genai
import os
import datetime
import time

# モデルの初期化
genai.configure(api_key=os.environ["API_KEY"])
model = genai.GenerativeModel(
    "gemini-1.5-flash-001",
    system_instruction="あなたは動画分析の専門家であり、動画ファイルに基づいて回答する役割を持っています。"
)

# 動画ファイルは事前にローカルにダウンロードしておく
# !wget https://storage.googleapis.com/generativeai-downloads/data/Sherlock_Jr_FullMovie.mp4
video_file_name = "./Sherlock_Jr_FullMovie.mp4"

# Files APIを使用してアップロード
video_file = genai.upload_file(
    path=video_file_name,
    display_name="sherlock jr movie"
)

# ファイルの処理が完了するのを待つ
while video_file.state.name == "PROCESSING":
    print('アップロード中...')
    time.sleep(5)
    video_file = genai.get_file(video_file.name)

print(f'アップロード完了: {video_file.uri}')

# 処理時間の計測開始
start_time = datetime.datetime.now()

# マルチモーダルでモデルへ問い合わせ
response = model.generate_content(
    [    
        "映画の中で登場するキャラクターの性格、外見、名前を紹介し、初登場のタイムスタンプもリストアップしてください。", 
        video_file
    ]
)

print(response.text)
print(response.usage_metadata)

end_time = datetime.datetime.now()
elapsed_time = end_time - start_time
print(f'処理時間: {elapsed_time}')
```

```markdown
The characters in the movie are:

- **Sherlock Jr.** (Played by Buster Keaton) is a young man who works as a projectionist at a theater in a small town. He also dreams of being a detective. Sherlock is depicted as a goofy, clumsy, but well-meaning character who is constantly trying to prove his abilities as a detective. He wears a suit, vest, tie, and a sailor hat. He first appears at 01:26. 
- **The Girl** (Played by Kathryn McGuire) is the object of Sherlock Jr.'s affections. She is a young woman who is initially skeptical of Sherlock's detective skills. She is sweet and kind but also a bit naive. She wears a dress with a gingham pattern. She first appears at 02:12.
- **The Girl's Father** is a serious and stern man who is unimpressed by Sherlock Jr.'s attempts to solve a mystery. He is depicted as a hard-working, practical individual. He wears a suit and a bow tie. He first appears at 01:51.
- **The Sheik** (Played by Ward Crane) is a mysterious character who seems to be involved in a scheme to steal the girl's jewels. He is a charming and cunning individual who is adept at fooling people. He wears a suit with pinstripes, a tie, and a bowler hat. He first appears at 03:45.
- **The Hired Man** is a rugged and tough-looking individual who works for the girl's father. He seems to be somewhat gruff but is ultimately a decent person. He wears a flannel shirt, a vest, and a cap. He first appears at 02:25.

prompt_token_count: 696206
candidates_token_count: 367
total_token_count: 696573

処理時間: 0:00:57.122914
```

## コンテキスト キャッシュを使った問い合わせ

次は同じ動画データでコンテキスト キャッシュを使ってやってみます。
コンテキスト キャッシュ自体は動画以外のテキスト、画像、音声データにも対応しています。

```python
import google.generativeai as genai
from google.generativeai import caching
import os
import datetime
import time

genai.configure(api_key=os.environ["API_KEY"])

# 動画ファイルは事前にローカルにダウンロードしておく
# !wget https://storage.googleapis.com/generativeai-downloads/data/Sherlock_Jr_FullMovie.mp4
video_file_name = "./Sherlock_Jr_FullMovie.mp4"

# Files APIを使用してアップロード
video_file = genai.upload_file(
    path=video_file_name,
    display_name="sherlock jr movie"
)

# ファイルの処理が完了するのを待つ
while video_file.state.name == "PROCESSING":
    print('アップロード中...')
    time.sleep(5)
    video_file = genai.get_file(video_file.name)

print(f'アップロード完了: {video_file.uri}')

# 10分間利用できるキャッシュを作成
cache = caching.CachedContent.create(
    model="models/gemini-1.5-flash-001",
    display_name="sherlock jr movie", # キャッシュを識別するための表示名
    system_instruction="あなたは動画分析の専門家であり、動画ファイルに基づいて回答する役割を持っています。",
    contents=[video_file],
    ttl=datetime.timedelta(minutes=10),
)

# 処理時間の計測開始
start_time = datetime.datetime.now()

# 作成したキャッシュを使用する
model = genai.GenerativeModel.from_cached_content(cached_content=cache)

# モデルへ問い合わせ
response = model.generate_content(
    ["映画の中で登場するキャラクターの性格、外見、名前を紹介し、初登場のタイムスタンプもリストアップしてください。"]
)

print(response.text)
print(response.usage_metadata)

end_time = datetime.datetime.now()
elapsed_time = end_time - start_time
print(f'処理時間: {elapsed_time}')
```

```
この映画には以下のキャラクターが登場します。

* **バスター・キートン**: 映画の主人公。彼は1:26に初登場します。バスターは、まだ若くて、少しお調子者で、探偵になりたいと夢見ています。彼は、映画のほとんどのシーンで、帽子、シャツ、ベスト、ズボン、ネクタイを着用しています。
* **キャサリン・マックガイア**: バスターが働いている劇場の若い女性です。彼女は2:13に初登場します。キャサリンは魅力的で、優しい心を持つ、若い女性です。彼女は、映画のほとんどのシーンで、格子柄のドレスと帽子を着用しています。
* **ジョー・キートン**: バスターの父。彼は2:17に初登場します。ジョーは、少しおかしなところがあり、バスターの探偵の夢を理解していません。彼は、映画のほとんどのシーンで、帽子、シャツ、ベスト、ズボンを着用しています。
* **ウォード・クレーン**: キャサリンの父。彼は3:41に初登場します。ウォードは、少し厳しく、保守的な人物ですが、娘に対しては優しいです。彼は、映画のほとんどのシーンで、帽子、スーツ、ネクタイを着用しています。

映画のタイトルは「シャーロック・ジュニア」です。

prompt_token_count: 696207
cached_content_token_count: 696181
candidates_token_count: 284
total_token_count: 696491

処理時間: 0:00:48.994498
```

回答内容のブレは誤差として、`cached_content_token_count`からキャッシュしたコンテンツが使われていることがわかります。
回答の生成にかかった時間も10秒ほど速くなりました。

コンテキスト キャッシュへのその他の操作については[こちら](https://cloud.google.com/vertex-ai/generative-ai/docs/context-cache/context-cache-overview)のドキュメントに載っています。


# 料金

料金については下記に記載があります。

https://cloud.google.com/vertex-ai/generative-ai/pricing?hl=en#context-caching

`Cached Input`はキャッシュから取得した入力量に関しての課金を表しているようで。
今回のプロンプトは128Kのコンテキストウィンドウは超えており、動画は44分なので。
ざっくり2640×0.00006575で **$0.17358** 。

`Context Cache Storage`はキャッシュしたデータ量とキャッシュ時間への課金なので。
ざっくり600×0.000263で **$0.1578** 。

よって合計 **$0.33138** かかっていそうです。

通常のマルチモーダルの[料金](https://cloud.google.com/vertex-ai/generative-ai/pricing?hl=en#google_models)と比較すると。
2640×0.0001315で **$0.34716** となります。

単発の問い合わせはあまり違いがありませんが100回問い合わせたと考えると。

マルチモーダルだけだと、 **$33.138** ですが、10分のコンテキスト キャッシュをうまく再利用できたケースを考えると、17.358＋0.1578の **$17.5158** となり半額近くコストが減らせそうです！

更にいうと双方プラスで出力トークンの費用も同額かかってそうですが割愛しています。

# まとめ

Geminiの最大の特徴である巨大なコンテキストウィンドウをより効率よく、より安く扱えるようになるアップデートでした！
これで、大きなコンテキストをプロンプトに渡して何度も問い合わせをするようなユースケースの実装にもある程度踏み切れるようになったのではないでしょうか。
