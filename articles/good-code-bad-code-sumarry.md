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

### コンストラクタで確実に正常値を設定

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
end
```

コンストラクタとは、クラスからインスタンスを生成する際に自動的に実行される初期化メソッドです。
Rubyでは `initialize` がコンストラクタにあたります。

コンストラクタ内で値の検証（バリデーション）を行うことで、不正な状態のオブジェクトが生成されるのを防ぐことができます。

もし不正な値を持ったままプログラムが動作すると、後続の処理でバグの原因になります。
そのため、不正値の混入を防ぐためにコンストラクタ内でバリデーションを行い、不正な値の場合は例外を発生させます。

今回は次のルールを正常値とします。

- 金額 `amount` : 0以上の整数  
- 通貨 `currency` : null以外


### インスタンス変数を不変にする

```
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency)
    if amount < 0
      raise ArgumentError.new('金額には0以上を指定してください。')
    end
    if currency.nil? || currency.empty?
      raise ArgumentError.new('通貨単位を指定してください。')
    end
    @amount = amount
    @currency = currency
    self.freeze  # 不変にする
  end
end
```

変数の値が変わる前提だと、いつ変更されたのか、今の値がどう変わっているのかをいちいち気にしなければなりません。
仕様変更で処理が変わった時、意図しない値に変わる、いわゆる「思わぬ副作用」が容易に発生します。
これを防止するためには、[freeze](https://docs.ruby-lang.org/ja/latest/method/Object/i/freeze.html)を使ってインスタンス変数を不変(イミュータブル)にします。


```
money = Money.new(100, "JPY")

money.instance_variable_set(:@amount, 200)
```

例えばこのように代入すると、`FrozenError`となります。

`freeze`を使用した場合のインスタンス変数は、一度しか代入できません。
インスタンス変数に不正値を直接代入できなくなるので、不正に強い構造と言えます。

### 変更したい場合は新しいインスタンスを作成する

```
class Money
// 省略
  def add(other)
    added = @amount + other.amount
    Money.new(added, @currency)
  end
```

値を変更したい場合でも、インスタンス変数の値を書き換えることはしません。
代わりに、変更後の値を持つ新しいMoneyインスタンスを生成して返します。
このようにすることで、既存のオブジェクトを変更することなく値の計算ができ、不変性を保つことができます。

## 不変の活用

### 不変にして再代入を避ける

```
def damage(member, enemy)
  tmp = member.power + member.weapon_attack
  tmp = tmp * (member.speed / 100.0)
  tmp = tmp - (enemy.defence / 2)
  tmp = [0, tmp].max

  tmp
end
```

計算の際にローカル変数tmpが使いまわされています。
変数tmpは、代入される値の意味がコロコロ変わっています。
途中で意味が変わると読み手が混乱し、誤解してバグを埋め込む可能性もあります。
そのため再代入を避けることが重要です。

### 変数を分ける

def damage(member, enemy)
  basic_attack_power = member.power + member.weapon_attack
  final_attack_power = basic_attack_power * (member.speed / 100.0)
  reduction = enemy.defence / 2
  damage = [0, final_attack_power - reduction].max

  damage
end

変数を分けて、再代入を避けることで以下のメリットを受けることができます。

- 処理の意味が読みやすくなる
- 変数の役割が明確になる
- バグが減る

## 低凝集

```
class GiftPoint {
  private static final int MIN_POINT = 0;
  final int value;

  GiftPoint(final int point) {
    if (point < MIN_POINT) {
      throw new IllegalArgumentException("ポイントが0以上ではありません。");
    }

    value = point;
  }

  /**
   * ポイントを加算する。
   *
   * @param other 加算ポイント
   * @return 加算後の残余ポイント
   */
  GiftPoint add(final GiftPoint other) {
    return new GiftPoint(value + other.value);
  }

  /**
   * @return 残余ポイントが消費ポイント以上であればtrue
   */
  boolean isEnough(final ConsumptionPoint point) {
    return point.value <= value;
  }

  /**
   * ポイントを消費する。
   *
   * @param point 消費ポイント
   * @return 消費後の残余ポイント
   */
  GiftPoint consume(final ConsumptionPoint point) {
    if (!isEnough(point)) {
      throw new IllegalArgumentException("ポイントが不足しています。");
    }

    return new GiftPoint(value - point.value);
  }
}

```

一見このGiftPointクラスにはポイントの加算メソッドや消費メソッドが定義されており、ギフトポイントに関するロジックがかなりしっかり凝集されていると思われます。

しかし、外部から自由に`new GiftPoint(5000)`や`new GiftPoint(999999)`のように自由に生成ができてしまいます。
コンストラクタを公開すると、様々な用途に使われがちです。
結果、関連ロジックが分散しがちになり、メンテナンスが困難になります。

### コンストラクタを外部から呼べないようにする


```
class GiftPoint
  MIN_POINT = 0
  STANDARD_MEMBERSHIP_POINT = 3000
  PREMIUM_MEMBERSHIP_POINT = 10000

  attr_reader :value

  private_class_method :new

  def initialize(point)
    raise ArgumentError, "ポイントが0以上ではありません。" if point < MIN_POINT

    @value = point
    freeze
  end

  def self.for_standard_membership
    new(STANDARD_MEMBERSHIP_POINT)
  end

  def self.for_premium_membership
    new(PREMIUM_MEMBERSHIP_POINT)
  end
end
```

[private_class_method](https://docs.ruby-lang.org/ja/latest/method/Module/i/private_class_method.html) :newにより、GiftPoint.new(5000)が外部から呼べなくなり、NoMethodErrorになります。
`GiftPoint.for_standard_membership`または`GiftPoint.for_premium_membership`しか作れなくなります。
つまり、GiftPoint = 入会ギフトポイントというドメインルールが保証されます。
また将来的に標準会員のポイントが変わっても`STANDARD_MEMBERSHIP_POINT = 4000`の値を変更するだけで良くなりました。

### 横断的関心事

さまざまなユースケースに広く横断する事柄を**横断的関心事**と呼びます。
**横断的関心事**に関する処理であれば共通処理としてまとめ上げても良い。
代表的なものとしては以下が挙げられます。

- ログ出力
- エラー検出
- デバッグ
- 例外処理
- キャッシュ
- 同期処理
- 分岐処理

## 条件分岐

### 条件分岐のネストによる可読性低下

```
# 生存しているか判定
if member.hit_point > 0
  # 行動可能かを判定
  if member.can_act?
    # 魔法力が残存しているかを判定
    if member.cost_magic_point <= member.magic_point
      member.consume_magic_point(member.cost_magic_point)
      member.chant(magic)
    end
  end
end
```

ネストが深くなることでコードの見通しが悪くなります。
どこからどこまでがif文の処理ブロックなのか、読み解くのが難しくなります。

```
return if member.hit_point <= 0
return unless member.can_act?
return if member.magic_point < member.cost_magic_point

member.consume_magic_point(member.cost_magic_point)
member.chant(magic)
```

こうしたネストが深い場合の対処法としては、早期リターンがあります。
早期リターンとは、条件を満たしていない場合に、直ちにreturnで抜けてしまうという手法です。
早期リターンを適用することで、ネストが解消され、ロジックの見通しが良くなりました。

### case文の重複
```
class MagicManager
  def get_name(type)
    name = ""

    case type
    when "fire"
      name = "ファイア"
    when "shiden"
      name = "紫電"
    end

    name
  end

  def cost_magic_point(type, member)
    magic_point = 0

    case type
    when "fire"
      magic_point = 2
    when "shiden"
      magic_point = 5 + (member.level * 0.2)
    end

    magic_point
  end
end
```

種類ごとに処理を切り替える場合はcase文を使うことが多いと思います。
今回で言うとtypeで分岐しています。
例えば、何十個もcase文が存在している場合は、注意深く対応すれば大丈夫とは言えなくなります。
仕様変更のたびにに、膨大なcase文から仕様変更に関する箇所を探し出せねばならず、可読性の経過が懸念されます。

```
class Magic
  attr_reader :name, :cost_magic_point, :attack_power, :cost_technical_point

  def initialize(magic_type, member)
    case magic_type
    when :fire
      @name = "ファイア"
      @cost_magic_point = 2
      @attack_power = 20 + (member.level * 0.5).to_i
      @cost_technical_point = 0

    when :shiden
      @name = "紫電"
      @cost_magic_point = 5 + (member.level * 0.2).to_i
      @attack_power = 50 + (member.agility * 1.5).to_i
      @cost_technical_point = 5

    when :hell_fire
      @name = "地獄の業火"
      @cost_magic_point = 16
      @attack_power = 200 + (member.magic_attack * 0.5 + member.vitality * 2).to_i
      @cost_technical_point = 20 + (member.level * 0.4).to_i

    else
      raise ArgumentError
    end
  end
end
```

case文の重複コードが存在している場合は、**単一責任選択の原則**の考え方が重要です。
この原則は端的に言うと、同じ条件式の条件式を書かず、一箇所にまとめようとする原則です。
この原則に従って修正すると魔法の仕様が1箇所にまとまり、可読性も向上します。

## コレクション
```
members.each do |member|
  if member.hit_point > 0
    if member.contains_state?(StateType::POISON)
      member.hit_point -= 10

      if member.hit_point <= 0
        member.hit_point = 0
        member.add_state(StateType::DEAD)
        member.remove_state(StateType::POISON)
      end
    end
  end
end
```

コレクションとは、複数のデータ（要素）をまとめて扱うためのデータ構造の総称になります。
今回のコレクション操作はmembersになります。
これもif文のネストが深く見通しが悪くなっています。

```
members.each do |member|
  next if member.hit_point == 0
  next unless member.contains_state?(StateType::POISON)

  member.hit_point -= 10

  next if member.hit_point > 0

  member.hit_point = 0
  member.add_state(StateType::DEAD)
  member.remove_state(StateType::POISON)
end
```

今回はループの中の処理になるので、next ifを使って早期リターンをする事で、ネストの解消を行うことで、可読性が向上します。

## 密結合
結合度とは、「モジュール間の依存の度合いを表す指標」です。
凝集度と同様に、モジュールの粒度をクラスとします。
また密結合とは、「あるクラスが、他の多くのクラスに依存」していることです。
結合度の低い、疎結合な構造へ改善すると、コードの変更が楽になります。

### 継承より委譲

下記のような継承は一見良さそうに見えます。

```
class PhysicalAttack
  # 単体攻撃
  def single_attack
    10
  end

  # 2回攻撃
  def double_attack
    20
  end
end


# 武闘家クラス それぞれの攻撃に少しダメージを上乗せする
class FighterPhysicalAttack < PhysicalAttack
  # オーバーライド
  def single_attack
    super + 20
  end

  def double_attack
    super + 20
  end
end

fighter_physical_attack = FighterPhysicalAttack.new
puts fighter_physical_attack.single_attack  # 30
puts fighter_physical_attack.double_attack  # 40
```

PhysicalAttack.doubleAttackのロジックがsingleAttackを2回実行する修正が行われると意図していない動作になる

```
class PhysicalAttack
  # 単体攻撃
  def single_attack
    10
  end

  # 2回攻撃
  def double_attack
    single_attack + single_attack
  end
end

# 武闘家クラス
class FighterPhysicalAttack < PhysicalAttack
  def single_attack
    super + 20
  end
end

fighter_physical_attack = FighterPhysicalAttack.new

puts fighter_physical_attack.single_attack  # 30
puts fighter_physical_attack.double_attack  # 60（意図しない結果になる例）
```

このように継承関係にあるクラスはサブクラスがスーパークラスの構造にひどく依存してしまう（スーパークラス依存）

```
class PhysicalAttack
  def single_attack
    10
  end

  def double_attack
    single_attack + single_attack
  end
end

# コンポジション版
class FighterPhysicalAttack
  def initialize
    @physical_attack = PhysicalAttack.new
  end

  def single_attack
    @physical_attack.single_attack + 20
  end

  def double_attack
    @physical_attack.double_attack + 20
  end
end

fighter_physical_attack = FighterPhysicalAttack.new

puts fighter_physical_attack.single_attack  # 30
puts fighter_physical_attack.double_attack  # 40
```


スーパークラス依存による密結合を避けるため、継承より委譲が推奨されます。
委譲とはコンポジション構造にすることです。
委譲（コンポジション構造）にすることで
スーパークラスではなくインスタンス変数として呼び出すことで意図しない結果をうまないようにすることができます。