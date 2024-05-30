---
title: "Cloud DLP（Data Loss Prevention）で機密情報を検出し置き換え、マスキングしてみる"
emoji: "👀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud", "clouddlp", "python"]
published: true
---

# はじめに

[Cloud DLP](https://cloud.google.com/security/products/dlp?hl=ja)を使うと個人情報やカード番号などのセンシティブデータを検出し、それを指定文字列に置き換えたりマスキングできます。

何をセンシティブデータとして検出するかはinfoType検出器で指定できます。
多種多様な組み込みの[infoType検出器](https://cloud.google.com/sensitive-data-protection/docs/infotypes-reference?hl=ja)が用意されており、それだけでなくカスタマイズしたものも指定できます。

また、Cloud Storageのデータにバッチジョブで適用したり、BigQueryと組み合わせて特定の列のデータに適用するのも可能です。

本記事ではCloud DLPのAPIを直接呼び出して、センシティブデータの入った文字列に変換をかけてみます。

# Cloud DLPによる検出、マスキングと置換

Cloud DLPのAPIを有効にしたプロジェクトで、Cloud Shellエディタを使ってソースコードの作成、実行します。

```python:dlp_test.py
from google.cloud import dlp_v2
from google.cloud.dlp_v2 import types


def deidentify_with_mask(project, text, info_types, masking_character='*'):
    """文字列中の機密情報をマスキング"""
    
    dlp = dlp_v2.DlpServiceClient()

    # 検査対象
    item = {'value': text}

    # 非識別化設定
    deidentify_config = {
        'info_type_transformations': {
            'transformations': [
                {
                    'primitive_transformation': {
                        'character_mask_config': {
                            'masking_character': masking_character,
                        }
                    }
                }
            ]
        }
    }

    # 検査設定
    inspect_config = {
        'info_types': info_types,
        'include_quote': True
    }

    # DLP APIを呼び出して文字列を非識別化
    response = dlp.deidentify_content(
        request={
            'parent': f'projects/{project}',
            'inspect_config': inspect_config,
            'deidentify_config': deidentify_config,
            'item': item
        }
    )

    # 非識別化された文字列を表示
    print(response.item.value)


def deidentify_with_replace(project, text, info_types, replace_string='[REDACTED]'):
    """文字列内の機密情報を置換"""
    
    dlp = dlp_v2.DlpServiceClient()

    # 検査対象
    item = {'value': text}

    # 非識別化設定
    deidentify_config = {
        'info_type_transformations': {
            'transformations': [
                {
                    'primitive_transformation': {
                        'replace_config': {
                            'new_value': {
                                'string_value': replace_string
                            }
                        }
                    }
                }
            ]
        }
    }

    # 検査設定
    inspect_config = {
        'info_types': info_types,
        'include_quote': True
    }

    # DLP APIを呼び出して文字列を非識別化
    response = dlp.deidentify_content(
        request={
            'parent': f'projects/{project}',
            'inspect_config': inspect_config,
            'deidentify_config': deidentify_config,
            'item': item
        }
    )

    # 非識別化された文字列を表示
    print(response.item.value)


if __name__ == '__main__':
    project_id = 'my-project'

    text_to_deidentify = """
My name is John Connor. I'm 35 years old.
My email is foobar@example.com.
My credit card number is 4242424242424242.
"""

    # 適用するinfoType検出器
    info_types = [
        'FIRST_NAME',
        'AGE',
        'EMAIL_ADDRESS',
        'CREDIT_CARD_NUMBER',
    ]
    # infoType検出器を辞書のリストに変換
    info_types = [{'name': info_type} for info_type in info_types]

    deidentify_with_mask(project_id, text_to_deidentify, info_types, '#')
    deidentify_with_replace(project_id, text_to_deidentify, info_types)
```

![](https://storage.googleapis.com/zenn-user-upload/981413fbe5e2-20240530.png)

```bash
$ python3 dlp_test.py

My name is #### Connor. I'm ############.
My email is ##################.
My credit card number is ################.


My name is [REDACTED] Connor. I'm [REDACTED].
My email is [REDACTED].
My credit card number is [REDACTED].
```

センシティブデータがマスキング及び置換されたことを確認できました。
