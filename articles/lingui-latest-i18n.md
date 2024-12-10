---
title: "Linguiで解決するi18n対応の最適解"
emoji: "🌐"
type: "tech"
topics: [nextjs, react, typescript, i18n, tech]
published: false
---

## はじめに

この記事は、[PLEX Advent Calendar 2024](https://qiita.com/advent-calendar/2024/plex)の 12 日目の記事です。
現在、弊社では i18n 対応をしているプロダクトはありませんが、非日本語ネイティブユーザーへの対応で、ターゲット層を広げられる可能性を感じ、i18n 対応に興味を持ったので、この記事を執筆しました。
今回はインストールやセットアップ周りでなく、Lingui でできることや実装に重点を置き解説していきます。

## i18n 対応とは

ソフトウェアやウェブサイトを国際化する対応のことを指します。
具体的には、異なる言語や文化圏のユーザーに対応するために、テキストの翻訳、日付や数値、通貨のフォーマット対応などフトウェア適用させるプロセスを含みます。
i18n という略称は、「Internationalization」の最初の「I」と最後の「n」の間に 18 文字があることに由来します。

## Lingui とは

[Lingui](https://lingui.dev/)は 、国際化（i18n）に対応するためのライブラリになります。
[Next.js 公式](https://nextjs.org/docs/app/building-your-application/routing/internationalization)の Internationalization ページ下部にも Lingui がおすすめされています。
また記事執筆の 2024 年 12 月 12 日時点でも[Github のスター数](https://github.com/lingui/js-lingui)は 4.7k のスター数を獲得しており、多くの開発者から支持されているライブラリです。
インストールやセットアップについては、 Lingui の[インストール](https://lingui.dev/installation)と Next.js の[Internationalization](https://nextjs.org/docs/app/building-your-application/routing/internationalization)[Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)に関する公式ドキュメントを参考にしてください。

## Lingui の主な特徴と Next-Intl との違い

大きな特徴としては、自動的に未翻訳の文字列を検出することができたり、翻訳ファイルを生成・更新する CLI コマンドを提供してくれています。
有名な i18n ライブラリである[Next-Intl](https://next-intl-docs.vercel.app/)と比較すると 、Lingui の方が翻訳作業の効率化、自動化がしやすく、未翻訳の文字列を検出する仕組みがデフォルトで用意されており、翻訳の漏れを防ぎやすいといったメリットがあります。また Next.js、React、Vue.js、TypeScript、Gatsby など、幅広いフレームワークに対応しています。
Next-Intl では翻訳対象が膨大になると、自身で設定した対象のキーが抜けているとその分翻訳の漏れが起きます。
また、Next-Intl は Next.js に特化しているため、使用できるフレームワークが限定的というデメリットがあります。

## Lingui を使った i18n 対応の実装フロー

1. ロケール（言語）の定義
2. パスの設定
3. 翻訳文字列の管理
4. 翻訳ファイルの生成と管理

それぞれ通貨を題材に日本語から英語の翻訳を解説していきます。

## 1. ロケール（言語）の定義

- ロケール（言語）の設定ファイルを追加します。

```js:/lingui.config.ts
import type { LinguiConfig } from '@lingui/conf'
import { formatter } from '@lingui/format-json'

export default {
  // 日本語と英語のため「ja」と「en」を設定
  locales: ['ja', 'en'],
  // 翻訳ファイルがなかったときに最終的に日本語にするため、 default は「ja」で設定
  fallbackLocales: { default: 'ja' },
  catalogs: [
    {
      // 翻訳ファイルの保存場所を設定
      path: '<rootDir>/src/i18n/locales/{locale}/messages',
      // 翻訳文字列を抽出するフォルダを設定
      include: ['src'],
    },
  ],
   //  JSON 形式でォーマットを設定
  format: formatter({ style: 'lingui' }),
} as const satisfies LinguiConfig
```

## 2. パスの設定

- ユーザーからのリクエストを処理する前に、ミドルウェアを使って前処理を行い、パスを書き換える。
- URL の末尾に「/ja」の場合は、日本語のページに遷移させる。
- URL の末尾に「/en」の場合は、英語のページに遷移させる。

```js:/src/middleware.ts
import Negotiator from 'negotiator'
import { type NextRequest, NextResponse } from 'next/server'
import { defaultLocale, locales } from './i18n/util/locales'

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  const pathnameHasLocale = locales.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`,
  )
  if (pathnameHasLocale) {
    return
  }

  const locale = getRequestLocale(request.headers)
  request.nextUrl.pathname = `/${locale}${pathname}`
   // ユーザーからリクエストがあったpathnameに応じてリダイレクトさせる
  return NextResponse.redirect(request.nextUrl)
}

function getRequestLocale(requestHeaders: Headers): string {
  // accept-languageでブラウザの言語を読み込む
  const langHeader = requestHeaders.get('accept-language') || undefined
  //localsで設定している言語を読み取る
  const languages = new Negotiator({ headers: { 'accept-language': langHeader } }).languages(locales.slice()) (今回は日本語と英語)
  const activeLocale = languages[0] || locales[0] || defaultLocale
  return activeLocale
}
```

## 3.翻訳文字列の管理

### Trans コンポーネント

- @lingui/react から提供されるコンポーネントで、JSX 内での翻訳に使用されます。
- 動的な値（変数）も埋め込めむことができる。

```js:/src/component/Currency.tsx
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

### t マクロ

- @lingui/macro からインポートされるユーティリティで、プレーンな翻訳文字列を扱う際に使用します。
- JSX 外部や動的な変数を翻訳文字列に組み込みたい場合に使用します。

```js:/src/component/Currency.tsx
function handleClick() {
   alert(t(i18n)`翻訳完了！`)}
}
```

## 4.翻訳ファイルの生成と管理

- `npm run lingui:extract` を実行することで、翻訳可能な文字列をコードから自動抽出できる。
- ソースコードをスキャンして、Lingui マクロ（例: t、Trans）で囲まれた文字列を抽出。
- lingui.config.js に指定されたロケール（例: en, ja）ごとに翻訳ファイルを更新。
- 新しく検出された翻訳可能な文字列を翻訳ファイル(message.json)に追加。

```js:/src/i18n/locales/en/messages.json
{
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
  },
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

}

```

### JSON ファイルの translation フィールドに対応する英語を追記する

- `npm run lingui:extract` を実行し、messages.json に　先ほどの Trans コンポーネントで囲った翻訳対象が書き出される。
- それぞれの translation のフィールドの値には何も入っていない状態なので、翻訳したい文字を埋める。
- src/i18n/locales/ja/messages.json の json ファイルと src/i18n/locales/en/messages.json の両方のファイルがあるが、日本語から英語にするので、src/i18n/locales/en/messages.json に翻訳したい文字列を埋める。
- 空白のままだと何も翻訳されずに、日本語のまま表示されるので翻訳したい文字列の追記を忘れないように注意が必要です。

```js:/src/i18n/locales/en/messages.json
{
  "hRIVGz": {
    "translation": "Yen" // Yenを追記する
    "message": "円",
    "comments": [],
    "origin": [
      [
        "src/component/Currency.tsx",
        16
      ]
    ]
  },
}
```

### 翻訳ファイルのコンパイルとアプリへの反映

- `npm run lingui:compile` を実行し、翻訳がアプリケーションで利用可能な形式に変換する。
- この実装によって日本語と英語を URL のパスを切り替えることによって、多言語翻訳が完了します。
- 実際のデモ動画からイメージを掴んで頂けたらと思います。

## さいごに

Lingui を使った i18n 対応の基本的な流れを解説しました。
今回初めて i18n 対応に触れてみましたが、Lingui を利用することで、翻訳文字列の抽出や管理が自動化され、効率的に多言語対応を実現できます。
Lingui は初めて i18n 対応に取り組む方にもおすすめです。まずは小さなプロジェクトで試し、翻訳管理や多言語対応の便利さを体感してみてください。
最後まで読んでいただきありがとうございました！
