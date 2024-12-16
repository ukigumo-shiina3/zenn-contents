---
title: "VercelのGrepで爆速コード検索！Github検索は時代遅れ？"
emoji: "🔍"
type: "tech"
topics: ["vercel", "github", "tech"]
published: false
---

## はじめに

この記事では、[Vercel](https://vercel.com/home) が 高速コード検索サービス[Grep](https://grep.app/) を[買収](https://vercel.com/blog/vercel-acquires-grep)し、実際に使ってみた感想をもとに、その特徴を解説します。
Vercel の CEO である Guillermo Rauch が [ 買収を発表した投稿](https://x.com/rauchg/status/1859365672444363037)からも、かなり期待が高まっています。
従来のコード検索といえば GitHub が一般的ですが、検索速度が遅かったり、UI/UX の観点から使い勝手に課題を感じる方も多いのではないでしょうか。
そこで今回は、GitHub に代わる Grep のサービス内容にフォーカスして記事を執筆しました！

## 特徴

- 50 万以上の公開 Git リポジトリを高速に検索可能。
- 再利用可能なコードや特定のライブラリ・アルゴリズムの実装例を簡単に見つけられる。
- Vercel の開発者向けツールと統合され、より強力な検索体験を提供。
- Grep の創設者 Dan Fox が Vercel の AI チームに参加し、コード検索機能のさらなる向上が期待される。

## コード検索

今回は useActionState を例に、Grep でのコード検索を試してみます。
ドキュメントを読むだけでは、API の概要はわかるものの、実際のユースケースが気になることも多いでしょう。Grep を使えば、数百の実例を瞬時に見つけることができます。

デモ動画を見ると、1 文字入力するごとに検索結果が即座に更新される様子がわかります。
おそらく、事前にコードをインデックス化しておくことで、高速な検索が可能になっていると思っています。

github と比較したら、どれだけ高速なのかよりわかると思います。

## 絞り込み

絞り込みを有効的に使うことで、より目当てのユースケースにたどり着くことができます。
デモ動画を例に、絞り込みの方法を解説します。

**Repository** : 特定のリポジトリで絞り込み可能。

**Path** : ファイルパスを指定して検索。

**Programming language** : プログラミング言語で絞り込み可能。

**Match case** : 大文字小文字を区別して検索。例: Transition で検索した場合、transition や TRANSITION は除外。

- 大文字小文字を区別し、検索クエリが完全に一致する大文字小文字の組み合わせの結果だけが返されます。
- Transition で検索した場合は、Transition で検索した場合は、transition, TRANSITION, TraNSITIon は一致しない。

**Match whole words** : 単語単位で検索し、誤った部分一致を防止。例: TransitionCode は除外される。

**Use regular expression** : 正規表現で高度な検索が可能。例: const\s+use[A-Z][a-zA-Z]\* でカスタムフックを検索。

## さいごに

Vercel との統合により、Grep は今後 AI 機能を取り入れ、検索意図を的確に理解したり、結果を分析する機能が追加されると期待されています。これにより、開発者の生産性向上にさらに貢献していくでしょう。

最後までお読みいただきありがとうございました！
