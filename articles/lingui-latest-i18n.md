---
title: "Linguiのi18n対応"
emoji: "🌐"
type: "tech"
topics: [nextjs, react, typescript, i18n, tech]
published: false
---

## はじめに

この記事は、[PLEX Advent Calendar 2024](https://qiita.com/advent-calendar/2024/plex)の 12 日目の記事です。
現在当社では i18n 対応をしているプロダクトはありませんが、国外にサービスを展開せずとも、国内で非日本語ネイティブの人にとっては国際化対応をされることで、ターゲットの幅も広がり、導入の需要もあるかと思い、記事を執筆しました。

## i18n 対応とは

ソフトウェアやウェブサイトを国際化する対応のことを指します。
具体的には、異なる言語や文化圏のユーザーに対応するために、テキストの翻訳、日付や数値、通貨のフォーマット対応などフトウェアを適応させるプロセスを含みます。
i18n という略称は、「Internationalization」の最初の「I」と最後の「n」の間に 18 文字があることに由来します。

## Lingui とは

[Lingui](https://lingui.dev/)は JavaScript/TypeScript アプリケーション、特に Next.js を使用した国際化（i18n）に対応するためのライブラリになります。
[Next.js 公式](https://nextjs.org/docs/app/building-your-application/routing/internationalization)の Internationalization ページの下部にも Lingui がおすすめされています。
また記事執筆時点の 2024 年 12 月 12 日時点でも[Github のスター数](https://github.com/lingui/js-lingui)は 4.7k なのである程度の支持を受けているライブラリになります。

## 特徴

大きな特徴としては、自動的に未翻訳の文字列を検出することができたり、翻訳ファイルを生成・更新する CLI コマンドを提供してくれています。
有名な i18n 対応のライブラリの[Next-Intl](https://next-intl-docs.vercel.app/)と比較すると 、Lingui の方が翻訳作業の効率化、自動化が可能であり、未翻訳の文字列を検出する仕組みがデフォルトで用意されており、翻訳の漏れを防ぎやすいといったメリットがあります。
また、Next-Intl は Next.js に特化しているため、Next.js アプリケーション以外には使いにくかったり、翻訳文字列の管理については基本的なファイルベースのサポートに留まってしまうことから、Lingui の方がフレームワークの多様性があります。

## 基本的な実装の流れ

1. Trans コンポーネントで追加した日本語を囲む
2. npm run lingui:extract で追加した日本語が message.json に書き出される
3. json の translation に対応する英語を追記する
4. npm run lingui:compile でコンパイルして変更を反映する

## Trans コンポーネント

```
import {  Trans } from '@lingui/macro'
import { useLingui } from '@lingui/react'

export function Currency() {

  return (
    <div className="p-8 text-4xl font-bold">
      <div>
        <Trans>
          円
        </Trans>
      </div>

      <div>
        <Trans>
        ドル
        </Trans>
      </div>

      <div>
        <Trans>
        ユーロ
        </Trans>
      </div>
    </div>
  )
}
```

- 今回は通貨を例に Trans コンポーネントで翻訳対象を囲む

## npm run lingui:extract の実行

基本的な動作

- ソースコードをスキャンして、Lingui マクロ（例: t、Trans）で記述された文字列を抽出。
- lingui.config.js に指定されたロケール（例: en, ja）ごとに翻訳ファイルを更新。
- 新しく検出された翻訳可能な文字列を翻訳ファイルに追加。

src/i18n/locales/en/messages.json

```
{
  "hRIVGz": {
    "translation": "",
    "message": "ドル",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        16
      ]
    ]
  },
  "E1t6Eh": {
    "translation": "",
    "message": "ユーロ",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        22
      ]
    ]
  },
  "3KWAVh": {
    "translation": "",
    "message": "円",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        10
      ]
    ]
  }
}

```

## json の translation に対応する英語を追記する

- npm run lingui:extract を実行し、messages.json に　先ほどの Trans コンポーネントで囲った翻訳対象が書き出される
- それぞれの translation のフィールドの値には何も入っていない状態なので、翻訳したい文字を埋める

```
{
  "hRIVGz": {
    "translation": "Dollar",
    "message": "ドル",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        16
      ]
    ]
  },
  "E1t6Eh": {
    "translation": "Euro",
    "message": "ユーロ",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        22
      ]
    ]
  },
  "3KWAVh": {
    "translation": "Yen",
    "message": "円",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        10
      ]
    ]
  }
}
```

## npm run lingui:compile でコンパイルして変更を反映する
