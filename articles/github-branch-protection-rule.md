---
title: "[Github]ブランチの保護ルール設定"
emoji: "🔧"
type: "tech"
topics: [Github, Git]
published: true
---

Github ではリポジトリごとにブランチの保護ルール設定ができます。

- 他のメンバーからレビューを承認されなければマージできない
- 特定のメンバーはマージできない
- CI が通らなければマージできない

上記のようなマージのルールを定めることができます。

## ブランチの保護ルール設定をしようと思った理由

- ブランチにマージしたらテストに失敗してしまう。
- プルリク時コードを確認し、レビューをするという習慣をつけたい。

## 設定方法

① 対象リポジトリの「settings」を選択後、サイドバーの「Branches」を選択。
![](https://storage.googleapis.com/zenn-user-upload/99dfd87cd31b-20220512.png)

②「Branch protection rule」から「Branch name pattern」で名称を付けるが、慣例的には対象とするブランチ名をつける。(main のリポジトリに設定するのであれば main と命名)

③ 設定したい項目に合わせてチェックを付けて保護ルールを設定する。
今回は「Require a pull request before merging」と「Require approvals」にチェックを付け、マージ前にプルリクを要求し、承認(Approve)されなければマージできないというルールを選択。
![](https://storage.googleapis.com/zenn-user-upload/6dffd3e6abee-20220512.png)

④ 最後に「Create」のボタンをクリックすることでルールが作成される。

## ルールの各項目の日本語訳と説明

![](https://storage.googleapis.com/zenn-user-upload/8995bc630799-20220512.png)
![](https://storage.googleapis.com/zenn-user-upload/edfb8edb9547-20220512.png)

## プルリク時の表示

「Review required」の項目が追加され、自分以外の誰かに承認(Approve)されないとマージできないという保護ルールが追加されました。
![](https://storage.googleapis.com/zenn-user-upload/87d6e26a09bf-20220512.png)

## さいごに

チーム開発では必須の Github の設定知識になりますので、ぜひ参考にしていただければと思います！
