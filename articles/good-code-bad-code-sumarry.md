---
title: "良いコード／悪いコードを「実務で使える形」に要約してみた"
emoji: "📘"
type: "tech"
topics: ["設計", "技術書", "リファクタリング", "レビュー", "コーディング"]
published: true
---

## はじめに
コード設計を正しくすることで、可読性の向上、潜在的なバグの解消を行い、コード品質向上を目的として、本記事を執筆しました。
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
    # 不正な値をチェック
    raise ArgumentError, "金額には0以上を指定してください。" if amount < 0
    raise ArgumentError, "通貨単位を指定してください。" if currency.nil?

    @amount = amount
    @currency = currency
  end
end
```

コンストラクタとは、インスタンス生成時に自動的に実行される初期化メソッドです。
Rubyでは `initialize` がコンストラクタにあたります。

コンストラクタ内で値を検証することで、不正な状態のオブジェクト生成を防げます。

もし不正な値を持ったままプログラムが動作すると、後続の処理でバグの原因になります。
そのため、不正値の混入を防ぐためにコンストラクタ内でバリデーションを行い、不正な値の場合は例外を発生させます。

今回は次のルールを正常値とします。

- 金額 `amount` : 0以上の整数  
- 通貨 `currency` : nil以外


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
インスタンス変数に不正値を直接代入できなくなるので、不正に強い構造になります。

### 変更したい場合は新しいインスタンスを作成する

```
class Money
  # 省略
  def add(other)
    added = @amount + other.amount
    Money.new(added, @currency)
  end
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
そのため再代入は避けましょう。

### 変数を分ける
```
def damage(member, enemy)
  basic_attack_power = member.power + member.weapon_attack
  final_attack_power = basic_attack_power * (member.speed / 100.0)
  reduction = enemy.defence / 2
  damage = [0, final_attack_power - reduction].max

  damage
end
```

変数を分けて、再代入を避けることで以下のメリットを受けることができます。

- 処理の意味が理解しやすくなる
- 変数の役割が明確になる
- バグが減る

## 低凝集

```
class GiftPoint
  MIN_POINT = 0

  attr_reader :value

  def initialize(point)
    if point < MIN_POINT
      raise ArgumentError, "ポイントが0以上ではありません。"
    end

    @value = point
  end

  # ポイントを加算する
  # @param other [GiftPoint]
  # @return [GiftPoint]
  def add(other)
    GiftPoint.new(@value + other.value)
  end

  # 残余ポイントが消費ポイント以上であればtrue
  # @param point [ConsumptionPoint]
  # @return [Boolean]
  def enough?(point)
    point.value <= @value
  end

  # ポイントを消費する
  # @param point [ConsumptionPoint]
  # @return [GiftPoint]
  def consume(point)
    unless enough?(point)
      raise ArgumentError, "ポイントが不足しています。"
    end

    GiftPoint.new(@value - point.value)
  end
end
```

一見このGiftPointクラスにはポイントの加算メソッドや消費メソッドが定義されており、ギフトポイントに関するロジックが十分に凝集されているように見えます。

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
つまり、GiftPointは入会ギフトポイントを表すというドメインルールが保証されます。
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
どこからどこまでがif文の処理ブロックなのか分かりづらくなります。

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
仕様変更のたびに、膨大なcase文から仕様変更に関する箇所を探し出せねばならず、可読性が低下します。

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
この原則は端的に言うと、同じ条件式を複数箇所に書かず、一箇所にまとめようとする原則です。
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

コレクションとは、複数のデータ（要素）をまとめて扱うためのデータ構造の総称です。
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
スーパークラスではなくインスタンス変数として呼び出すことで意図しない結果を生まないようにすることができます。

## 設計の健全性
### マジックナンバー

```
class ComicManager
  attr_accessor :value

  def initialize(value)
    @value = value
  end

  def is_ok?
    60 <= @value
  end

  def try_consume
    tmp = @value - 60
    raise RuntimeError if tmp < 0
    @value = tmp
  end
end
```

60という数字が何を表しているのかわからない。
実はこの60という数字は、コミックをお試しで購読するときに消費するポイントを表しています。
is_okメソッドは、お試し購読可能かどうかを返すメソッドで、
try_consumeメソッドは、お試し購読により購読消費するメソッドです。
ここまで、説明しないと、60の意図が見えてきません。
このようにロジック内に直接書き込まれている意図不明な値を**マジックナンバー**と呼びます。

```
class ComicManager
  TRIAL_READING_POINT = 60

  attr_accessor :value

  def initialize(value)
    @value = value
  end

  def ok?
    TRIAL_READING_POINT <= @value
  end

  def try_consume
    raise RuntimeError if @value < TRIAL_READING_POINT
    @value -= TRIAL_READING_POINT
  end
end
```

マジックナンバーを書かないようにするには、定数として定義する。
お試し購読の消費ポイントを**TRIAL_READING_POINT**として定義します。
定数として定義することで、お試し購読の消費ポイントに仕様が変更されても、**TRIAL_READING_POINT**のみ値を変更すれば良くなりました。

### null問題

```
class Member
  attr_accessor :head, :body, :arm
  attr_reader :defence

  def initialize(defence, head: nil, body: nil, arm: nil)
    @defence = defence
    @head = head
    @body = body
    @arm = arm
  end

  # 防具の防御力を加味した総合防御力を返す
  def total_defence
    total = @defence

    total += @head.defence if @head
    total += @body.defence if @body
    total += @arm.defence if @arm

    total
  end

  # 全ての防具を外す
  def take_off_all_equipments
    @head = nil
    @body = nil
    @arm = nil
  end
end
```

nullが入り込む前提でロジックを組むと、いたるところでnullチェックをしなければなりません。
nullチェックが漏れるとバグに繋がります。

```
class Equipment
  attr_reader :name, :price, :defence, :magic_defence

  EMPTY = new("装備なし", 0, 0, 0)

  def initialize(name, price, defence, magic_defence)
    raise ArgumentError, "無効な名前" if name.nil? || name.empty?

    @name = name
    @price = price
    @defence = defence
    @magic_defence = magic_defence
  end
end
```

nullチェックを避けるために、そもそもnullを取り扱わない設計にすることが大事です。
「装備がない状態」をnullではなくオブジェクトで表現しています。
EMPTYを使えば、Equipment::EMPTYとして、「装備がない状態」ということがコード上から読み取れるようになります。

### 例外の握りつぶし
```
def load_equipment
  begin
    read_file
  rescue StandardError
    # 何の処理もしていない
  end
end

def read_file
  raise "読み込みエラー"
end
```

**rescue StandardError**で例外をキャッチしていますが、何の処理もしていません。
そのため、これは**例外の握りつぶし**と呼ばれます

```
def load_equipment
  begin
    read_file
  rescue StandardError => e
    puts "装備読み込み失敗"
    raise e
  end
end

def read_file
  raise "読み込みエラー"
end
```

今回の例では例外をキャッチした場合は、確実にエラー通知を行うようにします。

## 名前設計
### 名前を設計する

名前はコードの可読性を高める以上の効果があります。
下記が重要なポイント。

- 可能な限り具体的で、意味範囲が狭い、特化した名前を選ぶ
  - 名前に関係のないロジックを排除できたり、仕様変更時の影響範囲が小さくすむ。
- 存在ベースではなく、目的ベースで名前を考える
  - 「存在ベース」のように単純に存在を示すだけの名前は、意味が多重になりがちで、目的不明オブジェクトになります。ロジックレベルで混乱をきたします。
  - 「目的ベース」のように具体的な目的がわかるようにする。
  - 下記が「存在ベース」と「目的ベース」の例になります。

    | 存在ベース | 目的ベース |
    ---|---|
    | 住所 | 配送元、配送先、勤務先、本籍地 |
    | 金額 | 請求金額、消費税額、延滞保証料、キャンペーン割引料金 |
    | ユーザー | アカウント、個人プロフィール、職務経歴 |
    | ユーザー名 | アカウント名、表示名、本名、法人名 |
    | 商品 | 入庫品、予約品、注文品、配送品 |
- どんな関心事があるか分析する
  - 登場人物や事柄を列挙したり、関係性を整理、分析する。
- 声に出して話してみる
  - ラバーダッキングのように会話することで整理されたり、無駄に気づくことができる
    <details>
    <summary>ラバーダッキングとは</summary>

    ラバーダッキングとは、問題解決のために  
    自分の考えを誰かに説明することで整理する手法です。

    </details>
- 利用規約を読んでみる
  - サービスの取扱やルールが厳密に書かれているので、特化した名前の設計に役立つ
    - 「購入者」、「出品者」、「売買契約」などの厳密な名前が例として挙げられます。
- 違う名前に置き換えられないか検討する
  - 意味範囲が適切か、複数の意味を持っていないか検討する。
  - また、違う名前を探すには類語辞典などを使うと役に立つ。
- 疎結合高凝集になっているか点検する
  - 目的以外のロジックが混入していないかチェックする。
  - ほかのクラスとどれくらい関連つけられているかも気をつける。
  - 何個も関連つけられている場合は密結合の危険性があるのでもっと狭い意味での名前にして分割するなど検討する

### 名前の省略
```
tr_fee = br_fee + LRF * dod
```

省略した名前だと意図がわからなくなります。
実際はレンタル料金総額の計算式で「基本料金 + 延滞料 + 延滞日数」ですが、コードからは読み取れないです。

```
total_rental_fee = basic_rental_fee + LATE_RENTAL_FEE_PER_DAY * days_overdue
```

全ての命名に言えることですが、メソッド名やクラス名、パッケージ名なども省略せずに書きましょう。
そうすることで可読性が上がり、他のメンバーや将来の自分を助けることになります。
ただしSNSやVIPといった、慣習的に省略形が使われており、意味が通じるものは問題ありません。

## コメント
### 意図や仕様変更時の注意点を読み手に伝える

```
class Member
  def initialize(states)
    @states = states
  end

  # 苦しい状態の場合はtrueを返す
  def painful?
    # 今後の仕様変更で状態異常による表情変化が追加される場合、
    # 本メソッドへロジックを追加すること
    if @states.include?(StateType::POISON) ||
       @states.include?(StateType::PARALYZED) ||
       @states.include?(StateType::FEAR)
      return true
    end

    false
  end
end
```

コード保守の際、読み手が気にするのは「このロジックはどういう意図で動いているのか」です。
仕様変更の際、読み手が気にするのは「何に注意すれば安全に変更できるか」です。
これらの課題を解決できるよう、意図や仕様変更時の注意点をコメントします。

## メソッド(関数)
### 必ず自身のインスタンス変数を使う
- メソッドは、必ず自身のインスタンス変数を使う。
  - 他のクラスのインスタンス変数を変更するメソッド構造にした場合は、低凝集に陥る

### 引数
- 引数は不変にする
  - 引数を変更すると値の意味が変わり、どんな意味なのか推測が困難になります。
  - また、どこで変更されたのか分かりにくくなります。
- フラグ引数は使わない
  - 何が起こるか読み手に想像させにくい。
  - ストラテジーパターンを使うなど別の機能に切り替える設計をしてみる.
    <details>
    <summary>ストラテジーパターンとは</summary>

      アルゴリズムを共通のインターフェースを持つ独立したクラスとして定義し、実行時に交換可能にすることで、処理内容の変更を利用側のコードから独立させるデザインパターンのこと。
    </details>
- nullを渡さない
  - nullを前提としたロジックは、nullチェックによる複雑化など様々な弊害が起きます。
  - nullを渡さないようにするには、nullに意味を持たせないようにします。
- 出力引数は使わない
  - 出力引数を使うと低凝集に陥る。
  - 引数は入力値として用いる。
    - 可読性の低下につながるので出力引数は使わない。
- 引数は可能な限り少なく
  - 多くの引数を使いたいメソッドでは、引数を使う分だけ処理内容が増加する。
  - 処理内容が増えるとロジックが複雑化する。

## モデリング
### モデリングとは
システム構造を説明するために、単純な箱で図式化したものをモデルと言います。
モデルは、特定の目的達成のために最低限考慮が必要な要素を備えたものです。
モデルの意図を定義し、構成を設計することをモデリングという。

### モデルの見直し方
クラスの構造に問題がある場合、モデルに問題があります。
モデルに歪さ、不自然さがあり、一貫性がない時には以下を検討してみる。

- そのモデルが達成しようとしている目的をすべて洗い出す
- 目的それぞれ特化したモデリングをし直す
- 目的駆動名前設計にもとづき、モデルに命名する
- モデルに目的外の要素が入り込んでいる場合、さらに見直す

## 設計の意義と設計への向き合い方
### 設計しないと開発生産性が低下する
変更容易性の設計をしないと、開発生産性が低下します。
低下要因は大きく分けて2つあります。
  <details>
  <summary>変更容易性とは</summary>
    仕様変更や機能追加が発生したときに、コードを簡単に修正できる性質のこと
  </details>

**要因1:バグを埋め込みやすくなる**
コード変更時にバグを埋め込みやすくなります。
バグがないよう、正確に変更できるまでに時間がかかってしまいます。
- 低凝集な構造によって仕様変更時に修正漏れが起きやすくなり、バグになる。
- コードの理解が難しいために、実装ミスが起こりやすく、バグになる。
- 不正値が容易に混入する構造になりがちで、バグが起きやすくなる。

**要因2:可読性の低下**
可読性が低下し、意図を正確に理解するまでに時間がかかってしまいます。
- ロジックの見通しが悪く、読み解くのに時間がかかる。
- 関係し合うロジックがあちこちに散在しているために、仕様変更に関連するロジックを全て探し回る手間が増える。
- 不正値の混入でバグが発生した場合に、どこから不正値が混入したのか追跡が困難になる。

## コミュニケーション

### 心理的安全性
心理的安全性とは、「自分が発言することを恥じたり、拒絶されるなど、不利益を被ることがないことをチームで共有されている心理状態」や「安心して自由に発言したり、行動できる状態」という定義がされています。
意見や提案をする上で、冷笑されたり、煙たがられたり、聞く耳を持たれない状態では、情報共有がうまくいきません。ましてチーム単位での設計品質向上はとても難しくなるでしょう。
コミュニケーションに課題がある時、まず心理的安全性の向上に努めましょう。

## 設計

### 「早く終わらせたい」心理が品質低下の罠
品質の低いシステムを作るチームには、共通した特徴があります。それは、クラス設計などの設計工程が軽視されていることです。忙しさや厳しい納期に追われる中で、「とにかく早く動くものを作る」ことが優先され、クラス図を描くといった基本的な設計すら省略されがちになります。

このような環境では、実装スピードの速いプログラマーが評価されやすくなります。実際、開発の初期段階では、設計を省いてコードを書いた方が早く成果が出るため、一見すると効率的に見えます。しかしその裏では、設計が不十分なまま機能追加や修正が繰り返され、コードは徐々に複雑化していきます。

やがて、粗悪なコードが蓄積されることで問題が顕在化します。コードは読みづらくなり、少しの修正でも思わぬ箇所に影響が及び、バグが頻発するようになります。その結果、修正や調査に時間がかかり、開発全体の生産性は大きく低下していきます。最初は速く実装できていたはずの開発者も、後になるほど手戻りやバグ対応に追われ、結果的にスピードを失っていきます。

「動くコード」を優先すると、短期的には速く見えますが、長期的にはコストが増大します。バグが多く、修正に多大な時間を要する状態では、「完成した」とは言い難いでしょう。真に価値のあるシステムとは、単に動くだけでなく、変更しやすく、安定して動き続けるものです。そのためには、初期段階から設計品質を意識することが不可欠です。

## レビュー

### 敬意と礼儀
コードレビューでは、技術的に正しい指摘であっても、攻撃的な言い方は許されません。強い言葉や否定的な表現は、相手の人格を傷つけるだけでなく、チーム全体の生産性を下げ、コードを良くするという本来の目的を妨げてしまいます。

そのため、コードレビューで最も重要なのは「敬意と礼儀」です。まず意識すべきは、技術的な正しさよりも、一緒に働く仲間を尊重する姿勢です。相手への敬意を持って伝えることが、結果的にコード品質を高める近道になります。

また、レビューを行う際には、相手が最善を尽くしていることを前提にし、感情的な否定や決めつけではなく、理由を添えて建設的に伝えることが大切です。人ではなくコードに対して意見を述べるという意識も重要です。

こうした考え方は、レビュー時の具体的な振る舞いとして次のように整理できます。

| いけないこと | 解説 |
| --- | --- |
| 人を辱めない | 相手は最善を尽くしていることを前提にします。「なぜ気づかなかった？」といった無意味なコメントをしないことです。 |
| 極端な言葉やネガティブな表現を使わない | 「まともな人ならこうはしない」といったネガティブな表現をレビュー時に使ってはいけません。人を侮辱して、思い通りに動かそうなどと考えてはいけません。人ではなく、コードについて議論します。 |
| ツールの使用を思いとどまらせない | コードフォーマッタなど自動化ツールを導入してくれたら、まずそのことに感謝します。ツールの利用の是非や好みを押し付けてはいけません。 |
| 自転車置き場の議論をしない | どちらでもいいようなことについて、レビュー上で決着を付けようとしないでください。レビューの目的は「勝ち負け」ではありません。 |

例えば、「この実装は良くない」と一方的に否定するのではなく、「現状でも問題なく動いているが、パフォーマンスの観点でより良い方法がある」といったように、相手を尊重しつつ改善案を提示する形が望ましいです。

無意識のうちに強い言い方になってしまうこともあるため、「正しければ何を言ってもいい」という考えは避け、常に相手への配慮を忘れないことが重要です。敬意と礼儀を持ったコミュニケーションこそが、健全なレビュー文化と高品質なコードを支えます。

## まとめ

本記事では、「良いコード／悪いコード」で紹介されている設計の考え方をもとに、可読性・保守性・変更容易性を高めるためのポイントを整理しました。

良いコードを書くためには、不正な値を防ぐ設計や不変性の活用、責務の分離といった基本を押さえることが重要です。また、ネストの深い条件分岐やマジックナンバー、密結合といった問題を避けることで、コードの見通しが良くなり、バグの発生を抑えることができます。

さらに、名前設計やコメント、レビュー時のコミュニケーションといった要素も、コード品質に大きく影響します。単に「動くコード」を書くのではなく、「変更しやすく、意図が伝わるコード」を意識することが、長期的な開発生産性の向上につながります。

日々の実装やレビューの中でこれらの観点を少しずつ取り入れていくことが、良いコードへの第一歩です。

最後まで読んでいただき、ありがとうございました！