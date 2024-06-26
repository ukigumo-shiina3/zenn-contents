---
title: "[Github]デフォルトブランチを変える方法"
emoji: "🔧"
type: "tech"
topics: [Github, Git]
published: false
---

通常作成したリポジトリのデフォルトのブランチは main になります。
develop をデフォルトに変えることでメリットがあるので設定フローを作成しました。
前提として、develop ブランチから作業ブランチを切ってチーム開発形式を取っている人向けに需要があると思います。

## デフォルトブランチを変えようと思った理由

- Github 上で真っ先に確認したいブランチは統合ブランチである develop を見たい。
- 作業ブランチからプルリクを送る際デフォルトが main だとプルリク先が main になってしまうため、ブランチ先を develop に選択する一手間がかかる。
- develop にマージしてから挙動を確認したかったのに、main にプルリクを送ってしまっていることに気づかず、マージしてしまうというミスをしてしまいデプロイ環境に影響してしまった過去がありました。

## 設定方法

① 対象リポジトリの上部のバーから「Settings」を選択後、サイドバーの「Branches」を選択。
![](https://storage.googleapis.com/zenn-user-upload/f9d293b1ac40-20220505.png)

② 画面中央の「Default branch」の下に書かれているブランチが現在のデフォルトブランチ(現段階ではデフォルトを develop に私が指定していますが、特に設定していなかった場合は main が表示されているはずです)。
ペンアイコンをクリックして、ブランチを変更できる。
![](https://storage.googleapis.com/zenn-user-upload/ead22e5c6edd-20220505.png)

③ ペンアイコンをクリックすると、ブランチの一覧が表示されるのでデフォルトブランチに設定したいブランチを選択する。
これで設定自体の変更は終了です。
![](https://storage.googleapis.com/zenn-user-upload/cf0ea848ac1d-20220505.png)

## プルリク先のブランチ

設定後、プルリク先のベースが develop に変わっているはずです。
![](https://storage.googleapis.com/zenn-user-upload/76c2f9c77286-20220505.png)

## さいごに

設定自体はすぐでき、試す価値はあると思いますのでぜひ参考にして頂ければ幸いです！
