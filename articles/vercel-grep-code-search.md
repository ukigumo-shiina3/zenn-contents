---
title: "VercelのGrepで爆速コード検索！GitHub検索は時代遅れ！？"
emoji: "🔍"
type: "tech"
topics: ["vercel", "github"]
published: true
---

## はじめに

[PLEX Advent Calendar 2024](https://qiita.com/advent-calendar/2024/plex)の 17 日目の記事です。
この記事では、[Vercel](https://vercel.com/home) が [買収](https://vercel.com/blog/vercel-acquires-grep)した高速コード検索サービス[Grep](https://grep.app/) を、実際に使ってみた感想とともとに、その特徴を解説します。
Vercel の CEO である Guillermo Rauch が [ 買収を発表した投稿](https://x.com/rauchg/status/1859365672444363037)からも、かなり期待が高まっています。
従来のコード検索といえば GitHub が一般的ですが、検索速度の遅さや UI/UX の課題に不満を感じる方も多いのではないでしょうか。
そこで今回は、GitHub に代わる選択肢として注目される Grep のサービス内容にフォーカスして記事を執筆しました！

## 特徴

Grep の主な特徴は以下の通りです。

**高速検索** : 50 万以上の公開 Git リポジトリを瞬時に検索可能。
**再利用性向上** : 再利用可能なコードや、特定のライブラリ・アルゴリズムの実装例を簡単に見つけられる。
**統合性** : Vercel のプラットフォームと統合され、より強力な検索体験を提供。
**成長性** : Grep の創設者 Dan Fox が Vercel の AI チームに加わり、検索機能のさらなる進化が期待される。

## コード検索

今回は useActionState を例に、Grep でのコード検索を試してみます。
ドキュメントを読むだけでは、API の概要はわかるものの、実際のユースケースが気になることも多いでしょう。Grep を使えば、数百の実例を瞬時に見つけることができます。
GitHub と Grep の検索結果の速度をデモ動画で比較してみましょう。

https://www.youtube.com/watch?v=gHOR4CxjmeM

https://youtu.be/S5thNtChrmI

最初の動画が GitHub でその次が Grep になります。
GitHub では、検索結果が表示されるまで数秒かかり、ややもどかしさを感じます。
一方 Grep では、1 文字入力するごとに検索結果が即座に更新されます。
この高速さは、事前にコードをインデックス化しておくことで実現しているのではないかと私は考えています。GitHub と比較すると、その速度の差が一目瞭然です。

## 絞り込み

絞り込み機能を活用すると、目当てのユースケースに効率よくたどり着けます。以下に、主な絞り込み方法を紹介します。

**Repository** : 特定のリポジトリで絞り込み可能。

**Path** : ファイルパスを指定して検索。

**Programming language** : プログラミング言語で絞り込み可能。

**Match case** : 大文字小文字を区別して検索。例: `Transition` で検索した場合、`transition `や `TRANSITION` は除外。

**Match whole words** : 単語単位で検索し、誤った部分一致を防止。`Transitions`や`TransitionCode` は除外される。

**Use regular expression** : 正規表現で高度な検索が可能。例: `const\s+use[A-Z][a-zA-Z]\*` でカスタムフックを検索。

https://www.youtube.com/watch?v=IRktS-ma3iY

https://www.youtube.com/watch?v=QifA-pdn0E4

これらの絞り込みを活用すれば、より正確かつ効率的に目的の情報を見つけることができます。

## さいごに

Vercel との統合により、Grep は今後 AI 機能を取り入れ、検索意図を的確に理解したり、結果を分析する機能が追加されると期待されています。これにより、開発者の生産性向上にさらに貢献していくでしょう。

Grep の機能が気になった方は、ぜひ試してみてください！
最後までお読みいただきありがとうございました！
