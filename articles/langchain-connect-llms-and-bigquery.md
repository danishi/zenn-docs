---
title: "LangChainでBigQueryのデータをLLMに利用させてみる"
emoji: "⛓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecolaboratory", "langchain", "googlecloud", "gc24", "bigquery"]
published: true
publication_name: "iret"
---

https://cloud.google.com/blog/ja/products/ai-machine-learning/open-source-framework-for-connecting-llms-to-your-data

先月公開されたこちらのブログを参考に、LangChainを使ってBigQueryのデータをLLMに動的に参照させて問いかけに答えられるか試してみました。

# データセットのスキーマ情報を読み込む
```python
from langchain.document_loaders import BigQueryLoader

query = f"""
SELECT table_name, ddl
FROM `bigquery-public-data.google_trends.INFORMATION_SCHEMA.TABLES`
WHERE table_type = 'BASE TABLE'
ORDER BY table_name;
"""

loader = BigQueryLoader(query, project=PROJECT_ID, metadata_columns="table_name", page_content_columns="ddl")
schema_data = loader.load()
```

LLMにプロンプトと合わせて渡すためのデータセットのスキーマ情報を取得するコードです。
`schema_data`にはCREATE TABLE文が入るので、精度を上げるにはカラムのコメント文も重要になりそうです。

今回はBigQueryの一般公開データセットである[Googleトレンド](https://console.cloud.google.com/marketplace/product/bigquery-public-datasets/google-search-trends?hl=ja)のデータセットを利用しました。

# プロンプトとスキーマ情報を基にクエリを生成する
```python
# プロンプト
PROMPT = "2024年2月13日更新日の「日本」のトレンドワード（急上昇ではない）と「ランク」を３位まで教えて。"
SYSTEM_PROMPT = "DISTINCTを付与すること。"

# Vertex AI SDK を初期化する
import vertexai
vertexai.init(project=PROJECT_ID, location=LOCATION)

# コード生成モデルを使用する
from langchain.llms import VertexAI
llm = VertexAI(model_name="code-bison@002", max_output_tokens=2048)

# チェーンを定義する
from langchain.prompts import PromptTemplate
from langchain.schema import format_document

fixed_prompt = PROMPT + "\n\nこちらの問いに答えるクエリを提案してください:\n\n{content}\n\n" + SYSTEM_PROMPT

content = lambda docs: "\n\n".join(
    format_document(doc, PromptTemplate.from_template("{page_content}")) for doc in docs
)
chain = (
   {
       "content": content
   }
   | PromptTemplate.from_template(fixed_prompt)
   | llm
)

# ドキュメントを使用してチェーンを呼び出し、コードのバッククォートなどを削除する
gen_query = chain.invoke(schema_data).strip('```').strip('sql')
print(gen_query) # 生成されたクエリ
```

テーブルにコメントがないのでかなり詳細にプロンプトを作らないと意図したクエリが出ませんでしたが、下記のクエリが生成できました。

```sql
SELECT DISTINCT term, rank
FROM `bigquery-public-data.google_trends.international_top_terms`
WHERE country_name = "Japan"
AND refresh_date = "2024-02-13"
ORDER BY rank
LIMIT 3;
```

# BigQueryへのクエリ実行
```python
import google.cloud.bigquery as bq

client = bq.Client(project=PROJECT_ID)
query_result = client.query(gen_query).result().to_dataframe()
query_result
```

実際に実行した結果がこちら。

| term | rank |
| --- | --- |
| 女子バスケ日本代表 | 1 |
| 佐賀記念 | 2 |
| 北鯖江駅 火災 | 3 |

# クエリ結果を使って回答
```python
llm = VertexAI(model_name="gemini-pro", temperature=0.5)

result_prompt = f"""
{gen_query}
上記のSQLを実行し
{query_result}
上記の実行結果が返ってきました。
これを踏まえ下記の質問に回答してください。
{PROMPT}
"""

result = llm(result_prompt)
result
```

クエリの実行結果を付与してプロンプトに回答させた結果がこちらです。

>1. 女子バスケ日本代表 (ランク: 1)
>2. 佐賀記念 (ランク: 2)
>3. 北鯖江駅 火災 (ランク: 3)

# まとめ
参考ブログのようにスキーマ情報を参照してクエリを実行し、**さらに最終的に当初のプロンプトに回答させる**ところまで実現することができました。

今回はデータセットの都合でプロンプトを作りこまないと狙ったクエリが生成されませんでしたが、分かりやすいデータベーススキーマ設計でテーブル名や列名、列コメントがしっかりしてれば割と良しなにクエリを生成してくれそうな雰囲気がします！

---

https://colab.research.google.com/drive/1nAgH2CaWUqidcDOf--tTj0p0fzZUf3MZ

Google Colabノートブックも公開してあるので、ご自身のプロジェクトで試すことも可能です。
