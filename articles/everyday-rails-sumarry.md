---
title: "Everyday Railsまとめ"
emoji: "🧪"
type: "tech"
topics: ["rspec", "test", "ruby", "rails"]
published: true
---

## はじめに

RSpecを体系的に学ぶために[Everyday Rails - RSpecによるRailsテスト入門](https://leanpub.com/everydayrailsrspec-jp)の書籍から業務で使用している項目を抜粋してまとめた記事になります。
私の会社では[GraphQL Ruby](https://graphql-ruby.org/)を採用しておりこの記事では モデルスペック を中心に取り上げ、以下のような理由から コントローラーやAPI、システムスペック についての解説は割愛しています。

- GraphQLでは GraphqlController のみを使用しており、一般的なコントローラーは存在しないため、テスト対象外としています。
- エンドポイントが1つ（/graphql）に集約されるため、REST APIのようなリクエストスペックは必要ありません。

## 対象読者

- これからRSpecを学びたい方
- RSpecは使用したことはあるが、より理解を深めたい方

## モデルスペック

モデルのテストは、アプリケーションのコアとなる部分をテストすることになるため、一番学習しやすいテストになります。

### バリデーションテスト

```
RSpec.describe User, type: :model do
    # 姓、名、メール、パスワードがあれば有効な状態であること
    it "is valid with a first name, last name, email, and password" do
        user = User.new(
        first_name: "Aaron",
        last_name:  "Sumner",
        email:      "tester@example.com",
        password:   "dottle-nouveau-pavilion-tights-furze",
    )
    expect(user).to be_valid
end
```

- このケースでは be_valid という RSpec のマッチャを使って、モデルが有効な状態かどうかを検証しています。
- trueもしくはfalseになることを検証し、今回は戻り値がtrueになることを期待しています。
- Userモデルに定義されているバリデーションが、全て満たされているため、テストはパスします。
  :::details マッチャ
  「期待値と実際の値を比較して、一致した（もしくは一致しなかった）という結果を返すオブジェクト」のこと
  :::

```
# 名がなければ無効な状態であること
it "is invalid without a first name" do
user = User.new(first_name: nil)
user.valid?
expect(user.errors[:first_name]).to include("can't be blank")
end
```

反対に有効でない場合もテストケースとして、作成する必要があります。

新しく作ったユーザーにfirst_nameにnilをセットし、バリデーションのチェックで、エラーメッセージが出力されることを期待します。

:::details include マッチャ
繰り返し可能な値の中に、ある値が存在するか どうかをチェックする
:::

### テストが失敗するケースを確認

また自分が作成したテストケースが失敗することをコードを変えてみて、テストが意図して動いていることを確認する」必要があります。

例えば先ほどのテストケースのexpect`(user.errors[:first_name]).to include("can't be blank")`の箇所を`expect(user.errors[:first_name]).to_not include("can't be blank")`と`to`から`to_not`に変えてみることでテストが意図して失敗するかを検証してみることは非常に重要です。
また、先ほどのテストコードの変更を元に戻し(`to_not` を` to`に戻す)、モデルのバリデーションの`first_name`をコメントアウトして、テストを実行することでもテストの失敗を確認することができる

### buildとcreateの違い

```
RSpec.describe Project, type: :model do
# ユーザー単位では重複したプロジェクト名を許可しないこと
  it "does not allow duplicate project names per user" do
    user = User.create(
      first_name: "Joe",
      last_name:  "Tester",
      email:      "joetester@example.com",
      password:   "dottle-nouveau-pavilion-tights-furze",
    )

    user.projects.create(
      name: "Test Project",
    )

    new_project = user.projects.build(
      name: "Test Project",
    )

    new_project.valid?
    expect(new_project.errors[:name]).to include("has already been taken")
  end
```

ここでテストしたいのは、 一人のユーザーは同じ名前で二つのプロジェクトを作成できないが、ユーザーが異なるとき は同じ名前のプロジェクトを作成できる、という要件です。
ちなみにbuildで作成している時とcreateで作成している箇所がありますが、違いは何になるか、下記に表してみました。

**build**

- **データベースへの保存:**
  - `build` は、オブジェクトをメモリ上に生成しますが、データベースには保存しません。
- **用途:**
  - オブジェクトの属性や関連付けを確認するテストに適しています。
  - データベースへの保存が不要なテスト、例えばモデルのバリデーションのテストなどで使用されます。
  - データベースへの保存を行わない為、テストの速度がcreateに比べて速いです。
- **利点:**
  - テストの高速化。
  - データベースの状態を汚染しない。

**create**

- **データベースへの保存:**
  - `create` は、オブジェクトを生成し、データベースに保存します。
- **用途:**
  - データベースに保存されたオブジェクトの存在や状態を確認するテストに適しています。
  - 統合テストやシステムテストなど、データベースとの連携が必要なテストで使用されます。
- **利点:**
  - データベースの状態を実際に確認できる。
  - 他のオブジェクトとの関連付けを永続化できる。

今回のテストでは「ユーザー単位では重複したプロジェクト名を許可しないこと」をテストするので、まずcreateで`Test Project`を作成してデータベースに保存した上で、buildして`Test Project`を作成して重複しているかのチェックを行うデータの準備をしています。
2つ目に`Test Project`を作成した場合はバリデーションのチェックのためだけに作成できていればいいので、データベースの保存は必要なくbuildの作成の方が適しているという考えになります。

## Factorybot

FactoryBotは、テスト用のダミーデータ（テストデータ）を簡単に作成するためのライブラリです。
RailsアプリのUserなどのモデルを、テストごとにサクッと作れるようになります。

```
FactoryBot.define do
 factory :user do
    first_name { "Aaron" }
    last_name  { "Sumner" }
    email { "tester@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
 end
end
```

このようにUserのFactoryBotを追加することで、テスト内で FactoryBot.create(:user) と書くだけで、簡単に新しいユーザー を作成できるようになります。
作成された際は、そのユーザーの名前は毎回 基本的に Aaron Sumner になります。メールアドレスやパスワードも最初から設定された状 態になります。
もちろん、文字列だけでなく、整数やブーリアン、日付など、属性に渡せるものなら何でも渡すことができます。

`Factorybotを使った場合`

```
it "has a valid factory" do
 expect(FactoryBot.build(:user)).to be_valid
end
```

`Factorybotを使わない場合`

```
it "is valid with a first name, last name, email, and password" do
  user = User.new(
    first_name: "Aaron",
    last_name:  "Sumner",
    email:      "tester@example.com",
    password:   "dottle-nouveau-pavilion-tights-furze",
)
expect(user).to be_valid end
```

Factorybotを使った場合と使わない場合を実際に見比べてみることで、りファクトリ版の方がより簡潔になっていることがわかると思います。

### 属性をオーバーライドする

```
# 名がなければ無効な状態であること
it "is invalid without a first name" do
 user = FactoryBot.build(:user, first_name: nil)
 user.valid?
 expect(user.errors[:first_name]).to include("can't be blank")
end
```

ここではFactorybotで定義してある`first_name: "Aaron"`を`first_name: nil`と指定して、オーバーライドしています。

## スペックをDryに保つ

### let で遅延読み込みする

`spec/models/note_spec.rb`を対象に既存のbeforeブロックの実装からletメソッドに変えた時の挙動について解説します。
こうすることでFactorybotの一部のセットするデータを一部書き換えることもできます。

### before ブロックの場合

```
require 'rails_helper'

RSpec.describe Note, type: :model do
  before do
    @user = User.create(
      first_name: "Joe",
      last_name:  "Tester",
      email:      "joetester@example.com",
      password:   "dottle-nouveau-pavilion-tights-furze",
    )

    @project = @user.projects.create(
      name: "Test Project",
    )
  end

  it "is valid with a user, project, and message" do
    note = Note.new(
      message: "This is a sample note.",
      user: @user,
      project: @project,
    )
    expect(note).to be_valid
  end

  it "is invalid without a message" do
    note = Note.new(message: nil)
    note.valid?
    expect(note.errors[:message]).to include("can't be blank")
  end

  describe "search message for a term" do
    before do
      @note1 = @project.notes.create(
        message: "This is the first note.",
        user: @user,
      )
      @note2 = @project.notes.create(
        message: "This is the second note.",
        user: @user,
      )
      @note3 = @project.notes.create(
        message: "First, preheat the oven.",
        user: @user,
      )
    end

    context "when a match is found" do
      it "returns notes that match the search term" do
        expect(Note.search("first")).to include(@note1, @note3)
      end
    end

    context "when no match is found" do
      it "returns an empty collection" do
        expect(Note.search("message")).to be_empty
      end
    end
  end
end
```

- beforeブロックを使うとdescribeやcontextブロックの内部で、各テストの実行前に共通のインスタンス変数をセットアップできます。
- この方法も悪くはないのですが、まだ解決できていない問題が二つあります。
- 第一に、before の中に書いたコードはdescribeやcontextの内部に書いたテストを実行するたびに毎回実行されます。これはテストに予期しない影響を及ぼす恐れがあります。また、そうした問題が起きない場合でも、使う必要のないデータを作成してテストを遅くする原因になることもあります。
- 第二に、要件が増えるにつれてテストの可読性を悪くします。

### letメソッドを使用した場合

```
require 'rails_helper'

RSpec.describe Note, type: :model do
  let(:user) { FactoryBot.create(:user) }
  let(:project) { FactoryBot.create(:project, owner: user) }

  it "is valid with a user, project, and message" do
    note = Note.new(
      message: "This is a sample note.",
      user: user,
      project: project,
    )
    expect(note).to be_valid
  end

  it "is invalid without a message" do
    note = Note.new(message: nil)
    note.valid?
    expect(note.errors[:message]).to include("can't be blank")
  end

  describe "search message for a term" do
    let!(:note1) {
      FactoryBot.create(:note,
        project: project,
        user: user,
        message: "This is the first note.",
      )
    }

    let!(:note2) {
      FactoryBot.create(:note,
        project: project,
        user: user,
        message: "This is the second note.",
      )
    }

    let!(:note3) {
      FactoryBot.create(:note,
        project: project,
        user: user,
        message: "First, preheat the oven.",
      )
    }

    context "when a match is found" do
      it "returns notes that match the search term" do
        expect(Note.search("first")).to include(note1, note3)
        expect(Note.search("first")).to_not include(note2)
      end
    end

    context "when no match is found" do
      it "returns an empty collection" do
        expect(Note.search("message")).to be_empty
        expect(Note.count).to eq 3
      end
    end
  end
end
```

- letは呼ばれて、初めて使われた時に実行されるメソッドです。
- let を使うことで、before ブロックに処理を詰め込まず、必要なデータを必要なテストの中だけで呼び出せるので、テストの読みやすさ・メンテナンス性が上がります。
- `user: user`で`let(:user)`を呼び出し、`user`の作成を行います。
- ` "is valid with a user, project, and message"`のテストが終わると、let で作成された値は、それぞれの it ブロックごとにスコープがリセットされます。したがって、ある it で作成された user は、他の it には影響を与えません
- つまり、「let は呼ばれたときに初めて評価される（遅延評価される）」ため。let(:user) は user が実際に参照されるまで評価されない。
- 「テスト実行時に必要なときだけ呼び出されることで、無駄なDBアクセスが避けられる」ため、パフォーマンスと可読性が向上するといったメリットがあります。

### let!かbeforeをちらを使うべきか

letは遅延評価を行うが、let!かbeforeは即時評価を行います。
即時評価は、テスト実行前にその場ですぐ実行されるメソッドになります。
私の場合は可読性やデバッグのしやすさから実行順が明確なbeforeを使うようにしています。
以下を例に解説します。

### let!

```
let!(:user) { create(:user) }
let!(:post) { create(:post, user:) }
let!(:comment) { create(:comment, post:) }

```

一見上から順に実行されそうに見えますが、実際の評価タイミングはRSpec内部の定義処理順に依存していて、いつ実行されたかは見た目では判断できない。
つまり、見た目の順序と実行順が一致しない可能性がある。

### before

```
before do
  user = create(:user)
  post = create(:post, user: user)
  comment = create(:comment, post: post)
end
```

1. userを作って
2. userを使ってpostを作って
3. postを使ってcomment を作る

という 処理の流れがそのままコードの見た目通り。
読む人が上から順にこうやって処理されるんだなとすぐ理解できます。
つまり、見た目の順序と実行順が一致する。

## さいごに

私の所属している会社では、基本的なルールとして[RuboCop RSpec ](https://docs.rubocop.org/rubocop-rspec/cops_rspec.html)のデフォルトルールに則るという方針を元にRspecを書いています。
その他にも外部APIの呼び出しをモックする場合は、基本的に[WebMock](https://github.com/bblimke/webmock)を使ったりしています。
