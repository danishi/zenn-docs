---
title: "5年かけてAWSとGoogle Cloudの認定資格を全冠したので振り返る"
emoji: "👑"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["aws", "googlecloud", "資格", "aws認定試験", "googlecloud認定資格"]
published: true
publication_name: "iret"
---

# はじめに

オンプレでの開発しか経験のなかった私が、2019年の転職を機にAWSやGoogle Cloudを利用するようになり、勉強のために認定資格を取り始めてだいたい5年。
ついに**AWSとGoogle Cloudの両認定資格のコンプリート**に至ったので振り返ろうと思います。

# 受験履歴

| 日付 | プラットフォーム | 試験 | 備考 |
| :---: | :-----------: | ---- | ------ | 
| 2024/02/11 | Google Cloud | Professional Machine Learning Engineer | **Google Cloud全冠達成👑** | 
| 2024/01/14 | Google Cloud | Professional Cloud Network Engineer | |
| 2024/01/06 | AWS | Database - Specialty | 再認定 |
| 2024/01/05 | Google Cloud | Professional Cloud Database Engineer | |
| 2023/12/21 | Google Cloud | Professional Cloud Security Engineer ※2回落ちた | |
| 2023/11/12 | Google Cloud | Professional Cloud DevOps Engineer ※1回落ちた | |
| 2023/11/02 | Google Cloud | Professional Google Workspace Administrator ※1回落ちた | |
| 2023/10/22 | Google Cloud | Professional Cloud Developer | |
| 2023/04/27 | AWS | DevOps Engineer - Professional | 再認定 |
| 2022/11/09 | Google Cloud | Professional Data Engineer | |
| 2022/09/10 | Google Cloud | Professional Cloud Architect | |
| 2022/08/06 | Google Cloud | Associate Cloud Engineer | 再認定 |
| 2022/05/08 | AWS | SAP on AWS - Specialty | |
| 2021/10/16 | Google Cloud | Cloud Digital Leader | |
| 2021/09/04 | AWS | Advanced Networking - Specialty ※1回落ちた | **AWS全冠達成👑** |
| 2021/07/04 | AWS | Solutions Architect - Professional ※1回落ちた | |
| 2021/06/04 | AWS | Machine Learning - Specialty | |
| 2021/05/07 | AWS | Security - Specialty ※1回落ちた| |
| 2021/05/07 | AWS | Data Analytics - Specialty ※1回落ちた | |
| 2021/03/06 | AWS | Database - Specialty | |
| 2020/12/14 | AWS | Alexa Skill Builder - Specialty | |
| 2020/08/22 | Google Cloud | Associate Cloud Engineer | |
| 2020/08/15 | AWS | DevOps Engineer - Professional | |
| 2020/02/15 | AWS | SysOps Administrator - Associate | |
| 2019/08/26 | AWS | Developer - Associate | |
| 2019/07/14 | AWS | Solutions Architect - Associate | |
| 2019/06/29 | AWS | Cloud Practitioner | |

# 資格取得のモチベ

もともと資格を取るのは好きで、何か新しいことを学ぶときは勉強の目標のために資格を取るようにしていました。
しかし、それだけではなく会社の支援制度が充実していたことが大きな理由です。

https://www.iret.co.jp/recruit/career/

ベンダー系の資格は受験料が高く、個人でいくつも受けるのは大変なので会社に感謝です。

## 全冠へのモチベ

最初はアソシエイトあたりをコンプリートして終わりでいいかと考えていたのですが、全冠達成するとAWSから表彰してもらえる（APN参加企業に所属している必要があります）ことを知り、どうせなら全部取ってやろうと思うようになりました。

https://aws.amazon.com/jp/blogs/psa/2023-japan-aws-all-certifications-engineers/

時間はかかりましたが、2023年は表彰してもらうことができました！
（本当は2022年も要件満たしていたのに応募が必要なことを知らなくて逃しました…）

https://aws.amazon.com/jp/blogs/psa/2024-japan-aws-all-certifications-engineers-criteria/

2024年も募集はまだですがクライテリアを満たしているので応募します！


Google Cloudの方にチャレンジした理由は、AWS全冠取得者は割とありふれてきたのでもういっちょ箔をつけようと思い頑張りました。

https://cloud.google.com/blog/ja/topics/partners/2024-google-cloud-partner-top-engineer-award-program

Google Cloudは全取得の表彰プログラムこそありませんが、Partner Top Engineerの審査基準に認定資格数の言及があるので資格を沢山持っていると有利に働くはずです。

# 試験対策について
両試験に共通してやっていた対策について。

ありきたりですが、**試験範囲のサービスを覚えること**、ドキュメントやブログ、書籍で概要レベルでもいいのでとにかく覚えました。
気になるサービスやイマイチ概要がつかみきれないものは自分で触ってみたりもしました。
廃止になったAlexaの試験では自分でAlexaアプリを作ってリリースするところまでやりました。

とはいえ、膨大なサービスを利用料を気にせず触りまくるのは大変すぎるので（個人で触るハードルが高いものもありますし）、ある程度は割り切って問題集を解きまくって突破したこともあります。

もちろんハンズオンや仕事で実際にそのサービスに触れていた方が理解はしていると言えるでしょうが、所詮はテストなので試されてるのは**知識量と問題文を読解して正しい正解にたどり着けるか**だと思っています。

全部とは言いませんが、どの試験の問題もとどのつまり、**問題のシチュエーションに応じて、要件を天秤にかけてベストプラクティスに沿った最適なサービス活用の選択肢**を選べれば正解にたどり着くことができると思いました。

下記のようなワードに気を配り、この問題が何の要件を大切にしているか判断し、適する選択肢を選ぶことが大事です。

* 費用対効果 ⇒ コスト（費用）の考慮
* 短時間で、迅速に ⇒ より手間やステップの少ない手段
* DR、高可用性 ⇒ マルチリージョン、IaC、高可用な設計
* 最小限の管理 ⇒ CI/CD、マネージドサービス
* セキュリティ ⇒ よりセキュアな手法
* 性能、パフォーマンス ⇒ より高い性能を出せる方法
* 最小限の開発 ⇒ ノーコード・ローコードサービス、FaaS

# 全冠することで得たもの
全冠することで何か自分が変わったかといえば別に変わりはしませんが、取得の過程で得た知識は普段の仕事にもしっかり活かされているなと感じています。

突然のトラブルシュートでも原因に思い当ったり、他のエンジニアやお客様とのディスカッションで使ったことのないサービスが出てきても概要を知っていれば話についていくこともできます。
現在のアーキテクチャでは解決できないこともこのサービスと組み合わせれば解決できるのではないかなどの**引き出し**も多くなりました。

# さいごに

次はAzure全冠とまでは考えていませんが、これからもアップデートされていくAWSやGoogle Cloudにキャッチアップしながら再認定で資格を維持していく所存です。
