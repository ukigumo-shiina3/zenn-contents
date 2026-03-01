---
title: "良いコード／悪いコード要点まとめ"
emoji: "📘"
type: "tech"
topics: ["設計", "技術書", "リファクタリング", "レビュー", "コーディング"]
published: false
---

## はじめに
コード設計を正しくすることで、可読性の向上、潜在的なバグの解消を行い、より良いコードを書いていきたいなと思ったことがこの記事を書いたきっかけになります。
名著である[良いコード／悪いコード](https://www.amazon.co.jp/%E8%89%AF%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89%EF%BC%8F%E6%82%AA%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89%E3%81%A7%E5%AD%A6%E3%81%B6%E8%A8%AD%E8%A8%88%E5%85%A5%E9%96%80%E2%80%95%E4%BF%9D%E5%AE%88%E3%81%97%E3%82%84%E3%81%99%E3%81%84-%E6%88%90%E9%95%B7%E3%81%97%E7%B6%9A%E3%81%91%E3%82%8B%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E6%9B%B8%E3%81%8D%E6%96%B9-%E4%BB%99%E5%A1%B2-%E5%A4%A7%E4%B9%9F-ebook/dp/B09Y1MWK9N/ref=sr_1_3?adgrpid=139536761235&dib=eyJ2IjoiMSJ9.wCudvM1lJDPbGRKCI7Wao58F6SURKTl5Pn90JP9I3rYXq4BUfymQQca7sW8VhlqzY8-vkF3M1Rkx8lxbq3ElCQAeFEzG_dkpFsbHtMFh5Xw.Yi3Ll353QR66VFiNlApEuExlAEjglEd5HH3-PxWn_Qg&dib_tag=se&hvadid=651300985102&hvdev=c&hvexpln=0&hvlocphy=1009298&hvnetw=g&hvocijid=2470014822369119926--&hvqmt=e&hvrand=2470014822369119926&hvtargid=kwd-1718532919576&hydadcr=13646_13521649&jp-ad-ap=0&keywords=%E8%89%AF%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89%E6%82%AA%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89&mcid=88de0e203dfc3599bfd79ba5e1ceb568&qid=1772342668&sr=8-3)を読み、要点を整理してコード品質の向上に繋げたいと思っています。

この記事を読むことで、良いコード／悪いコードの要点の把握と実際の活用例を学ぶことができます。
また例として載せているコードは、普段実務で使用しているRubyのコードで書き直しています。

## 対象読者

- コード品質を向上させたい方
- 良いコード／悪いコードを読んでない方、または読み直したい方

## クラス設計

### コンストラクトで確実に正常値を設定

```
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency)
    // 不正な値をチェック
    raise ArgumentError, "金額には0以上を指定してください。" if amount < 0
    raise ArgumentError, "通貨単位を指定してください。" if currency.nil?

    @amount = amount
    @currency = currency
  end

  def add(other)
    unless currency == other.currency
      raise ArgumentError, "通貨単位が違います。"
    end

    Money.new(amount + other.amount, currency)
  end
end
```

不正値を持ったままプログラムが作動していくと、バグが発生します。
不正値の混入を防ぐたみにバリデーションをコンストラクタ内に定義します。
不正値の場合は例外をスローするようにします。

まず正常値のルールを定義します。
次の定義に当てはまらない値が不正値となります。
これらをバリデーションとして下記をコンストラクタに実装します。

- 金額 amount: 0以上の整数
- 通過 currency: null以外