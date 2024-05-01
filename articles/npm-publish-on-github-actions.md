---
title: "GitHub Actionsでnpm publishをする"
emoji: "🎁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["npm","githubactions"]
published: true
publication_name: "iret"
---

# これは

GitHubでリリースタグを切ったら`npm publish`するワークフローの作り方。

# ワークフローの定義

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

jobs:
  deploy:
    name: Node.js ${{ matrix.python-version }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    timeout-minutes: 10
    strategy:
      matrix:
        node-version: [20.x]
        os: [ubuntu-latest]

    permissions:
      contents: read
      id-token: write

    steps:
      - name: Checkout 🔔
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }} 🔧
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies 🧹
        run: npm ci

      - name: Build 🔨
        run: npm run build

      - name: Publish 🎁
        run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

# リポジトリシークレットの設定

リポジトリシークレットとして`NPM_TOKEN`にnpmのアクセストークンを設定するためにトークンを発行する。

https://www.npmjs.com/settings/[ユーザー名]/tokens

Generate New Token -> Classic Token

![](https://storage.googleapis.com/zenn-user-upload/f861dad251f4-20240501.png)

トークンのタイプはAutomationにする。
