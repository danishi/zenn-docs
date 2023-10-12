---
title: "htmltestでHTMLファイルのリンク切れを確認する"
emoji: "🧪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html", "テスト", "自動化", "cicd", "githubactions"]
published: true
---

# htmltestとは
https://github.com/wjdp/htmltest

[htmltest](https://github.com/wjdp/htmltest)はOSSのGo言語で作られたコマンドラインツールです。  
HTML形式のファイルに対して、タグ内のリンクをたどってアクセスできるかを検証できます。  
  
静的サイトジェネレーター等で生成したHTMLファイルのリンク切れを確認することで、自動テストに使うことができます。
  
例えば次のようなHTMLをチェックしてみます。

```html:index.html
<!DOCTYPE html>
<html>

<body>

  <h1>htmltest</h1>

  <a href="https://www.example.com">This is a link</a>
  <a href="https://hoge.example.com">This is a non-existent link</a>

</body>

</html>
```

```bash
$ htmltest index.html 
htmltest started at 02:51:28 on .
========================================================================
index.html
  Get "https://hoge.example.com": dial tcp: lookup hoge.example.com on 192.168.65.5:53: no such host --- index.html --> https://hoge.example.com
========================================================================
✘✘✘ failed in 10.1932048s
1 errors
```

存在しないドメインのリンクが１つあるためエラーとなりました。  
  
aタグだけでなくimgタグやscriptタグ、metaタグなどリンクを参照できるタグはチェックできます。
  
ファイル指定ではなく、ディレクトリやその中の拡張子を指定してチェックもでき、コンフィグを定義すればもっと細かい制御もできるようです。  
  
Curlでインストールできたり、Dockerイメージの提供があったり、[GitHub Actions](https://github.com/wjdp/htmltest-action)でも使えるのでCI/CDにも組み込みやすくなってます。
  
サイトのリンク切れはSEO的にも不利になります。  
リンク切れのチェックという面倒な作業を自動化できるのは便利ですが、静的なHTMLが吐き出せるものにしか使えないので、動的なサイトには別のツールを使う必要がありますね。
