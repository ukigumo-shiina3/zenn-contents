---
title: "Everyday Railsのエッセンス凝縮！RailsのRSpecモデルテスト入門"
emoji: "🧪"
type: "tech"
topics: ["rspec", "test", "ruby", "rails"]
published: true
---

## はじめに

RSpec を体系的に学ぶために[Everyday Rails - RSpec による Rails テスト入門](https://leanpub.com/everydayrailsrspec-jp)の書籍から業務で使用している項目を抜粋してまとめた記事になります。

私の会社では[GraphQL Ruby](https://graphql-ruby.org/)を採用しているため、この記事ではモデルスペックを中心に取り上げ、以下のような理由からコントローラーや API、システムスペックについての解説は割愛しています。

- GraphQL では GraphqlController のみを使用しており、一般的なコントローラーは存在しないため、テスト対象外としています。
- エンドポイントが 1 つ（/graphql）に集約されるため、REST API のようなリクエストスペックは対象外としています。

## 対象読者

- これから RSpec を学びたい方
- RSpec は使用したことはあるが、より理解を深めたい方

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
- true もしくは false になることを検証し、今回は戻り値が true になることを期待しています。
- User モデルに定義されているバリデーションが、全て満たされているため、テストはパスします。
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

新しく作ったユーザーに first_name に nil をセットし、バリデーションのチェックで、エラーメッセージが出力されることを期待します。

:::details include マッチャ
繰り返し可能な値の中に、ある値が存在するか どうかをチェックする
:::

### テストが失敗するケースを確認

また自分が作成したテストケースが失敗することをコードを変えてみて、テストが意図して動いていることを確認する」必要があります。

例えば先ほどのテストケースの expect`(user.errors[:first_name]).to include("can't be blank")`の箇所を`expect(user.errors[:first_name]).to_not include("can't be blank")`と`to`から`to_not`に変えてみることでテストが意図して失敗するかを検証してみることは非常に重要です。
また、先ほどのテストコードの変更を元に戻し(`to_not` を` to`に戻す)、モデルのバリデーションの`first_name`をコメントアウトして、テストを実行することでもテストの失敗を確認することができる

### build と create の違い

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
ちなみに build で作成している時と create で作成している箇所がありますが、違いは何になるか、下記に表してみました。

**build**

**データベースへの保存:**

- `build` は、オブジェクトをメモリ上に生成しますが、データベースには保存しません。
  **用途:**
  - オブジェクトの属性や関連付けを確認するテストに適しています。
  - データベースへの保存が不要なテスト、例えばモデルのバリデーションのテストなどで使用されます。
  - データベースへの保存を行わない為、テストの速度が create に比べて速いです。
- **利点:**
  - テストの高速化。
  - データベースの状態を汚染しない。

**create**

**データベースへの保存:**

- `create` は、オブジェクトを生成し、データベースに保存します。
- **用途:**
  - データベースに保存されたオブジェクトの存在や状態を確認するテストに適しています。
  - 統合テストやシステムテストなど、データベースとの連携が必要なテストで使用されます。
- **利点:**
  - データベースの状態を実際に確認できる。
  - 他のオブジェクトとの関連付けを永続化できる。

今回のテストでは「ユーザー単位では重複したプロジェクト名を許可しないこと」をテストするので、まず create で`Test Project`を作成してデータベースに保存した上で、build して`Test Project`を作成して重複しているかのチェックを行うデータの準備をしています。
2 つ目に`Test Project`を作成した場合はバリデーションのチェックのためだけに作成できていればいいので、データベースの保存は必要なく build の作成の方が適しているという考えになります。

## Factorybot

FactoryBot は、テスト用のダミーデータ（テストデータ）を簡単に作成するためのライブラリです。
Rails アプリの User などのモデルを、テストごとにサクッと作れるようになります。

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

このように User の FactoryBot を追加することで、テスト内で FactoryBot.create(:user) と書くだけで、簡単に新しいユーザー を作成できるようになります。
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

Factorybot を使った場合と使わない場合を実際に見比べてみることで、りファクトリ版の方がより簡潔になっていることがわかると思います。

### 属性をオーバーライドする

```
# 名がなければ無効な状態であること
it "is invalid without a first name" do
 user = FactoryBot.build(:user, first_name: nil)
 user.valid?
 expect(user.errors[:first_name]).to include("can't be blank")
end
```

ここでは Factorybot で定義してある`first_name: "Aaron"`を`first_name: nil`と指定して、オーバーライドしています。

### シーケンスを使ってユニークなデータを生成する

ファクトリで複数 のユーザーをセットアップする必要が出てきた場合は実際のテストコードが走る前に例外が 発生します。
たとえば以下のような場合です。

```
# 複数のユーザーで何かする
it "does something with multiple users" do
user1 = FactoryBot.create(:user) user2 = FactoryBot.create(:user) expect(true).to be_truthy
end
```

すると次のようなバリデーションエラーが発生します。

```
Failures:
  1) User does something with multiple users
     Failure/Error: user2 = FactoryBot.create(:user)
     ActiveRecord::RecordInvalid:
       Validation failed: Email has already been taken
```

Factory Bot では シーケンス を使ってこのようなユニークバリデーションを持つフィール ドを扱うことができます。シーケンスはファクトリから新しいオブジェクトを作成するたび に、カウンタの値を 1 つずつ増やしながら、ユニークにならなければいけない属性に値を設 定します。ファクトリ内にシーケンスを作成して実際に使ってみます。

```
FactoryBot.define do
  factory :user do
    first_name { "Aaron" }
    last_name { "Sumner" }
    sequence(:email) { |n| "tester#{n}@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

メール文字列に n の値が入ることで重複を防ぐことができます。
こうすれば新しいユー ザーを作成するたびに、tester1@example.com 、tester2@example.com というように、ユニークで 連続したメールアドレスが設定されます。

### ファクトリで関連を扱う

メモとプロジェクトのファクトリを作成し、Factory Bot で他のモデルと関連を持つモデルを扱ってみます。

```rb:spec/factories/notes.rb
FactoryBot.define do
  factory :note do
    message { "My important note." }
    association :project
    association :user
  end
end
```

```rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Project #{n}" }
    description { "A test project." }
    due_on { 1.week.from_now }
    association :owner
  end
end
```

ユーザーファクトリに少し情報を追加してみます。
ファクトリに 名前を付けている 2 行目に、以下に示すような owner という別名(alias)を付けてみます。

```rb:spec/factories/users.rb
FactoryBot.define do
  factory :user, aliases: [:owner] do
    first_name { "Aaron" }
    last_name { "Sumner" }
    sequence(:email) { |n| "tester#{n}@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

メモはプロジェクトとユーザーの両方に属しています。
しかし、テストのたびにいちいち 手作業でプロジェクトとユーザーを作りたくありません。私たちが作りたいのはメモだけになります。

```rb:spec/models/note_spec.rb
require 'rails_helper'
RSpec.describe Note, type: :model do
  # ファクトリで関連するデータを生成する
  it "generates associated data from a factory" do
    note = FactoryBot.create(:note)
    puts "This note's project is #{note.project.inspect}" puts "This note's user is #{note.user.inspect}"
  end
end
```

ここでは Factory Bot を 1 回しか呼んでいないにもかかわらず、テストの実行結果を見ると 必要なデータが全部作成されています。

```
Note
This note's project is #<Project id: 1, name: "Test Project 1",
description: "Sample project for testing purposes", due_on:
"2017-01-17", created_at: "2017-01-10 04:01:24", updated_at:
"2017-01-10 04:01:24", user_id: 1>
This note's user is #<User id: 2, email: "tester2@example.com", created_at: "2017\
-01-10 04:01:24", updated_at: "2017-01-10 04:01:24",
first_name: "Aaron", last_name: "Sumner">
```

ですが、この例はファクトリで関連を扱う際の潜在的な落とし穴も示しています。ユーザーのメールアドレスをよく見てください。なぜ tester1@example.com ではなく、tester2@example.com になっているのでしょうか?この理由はメモのファクトリが 関連するプロジェクトを作成する際に関連するユーザー(プロジェクトに関連する owner) を作成し、それから 2 番目のユーザー(メモに関連するユーザー)を作成するからです。
この問題を回避するためにメモのファクトリを次のように更新します。こうするとデフォ ルトでユーザーが 1 人しか作成されなくなります。

```rb:spec/factories/notes.rb
FactoryBot.define do factory :note do
    message { "My important note." }
    association :project
    user { project.owner }
  end
end
```

スペックの結果を見てもユーザーは 1 人だけになります。

```
Note
This note's project is #<Project id: 1, name: "Test Project 1", description: "Sam\
ple project for testing purposes", due_on: "2017-01-17", created_at: "2017-01-10 \
04:18:03", updated_at: "2017-01-10 04:18:03", user_id: 1>
This note's user is #<User id: 1, email: "tester1@example.com", created_at: "2017\
-01-10 04:18:03", updated_at: "2017-01-10 04:18:03", first_name: "Aaron", last_na\
me: "Sumner">
```

今回のようなファクトリを使って、件数を扱う検証の場合は注意が必要になります。
User ファクトリに追加した alias に戻ってみます。Project モデルを見 ると、User の関連は owner という名前になっているのがわかると思います。

````rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Project #{n}" }
    description { "A test project." }
    due_on { 1.week.from_now }
    association :owner
  end
end

このように Factory Bot を使う際はユーザーファクトリに対して owner という名前で参照さ れる場合があると伝えなくてはいけません。そのために使うのが alias です。

### ファクトリ内の重複をなくす
Factory Bot では同じ型を作成するファクトリを複数定義することもできます。たとえば、 スケジュールどおりのプロジェクトとスケジュールから遅れているプロジェクトをテストし たいのであれば、別々の名前を付けてプロジェクトファクトリの引数に渡すことができます。 その際はそのファクトリを使って作成するインスタンスのクラス名と、既存のファクトリと 異なるインスタンスの属性値(この例でいうと due_on 属性の値)も指定します。

```rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" } description { "Sample project for testing purposes" } due_on { 1.week.from_now }
    association :owner
  end

  # 昨日が締め切りのプロジェクト
  factory :project_due_yesterday, class: Project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" } due_on { 1.day.ago }
    association :owner
  end

  # 今日が締め切りのプロジェクト
  factory :project_due_today, class: Project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" }
    due_on { Date.current.in_time_zone }
    association :owner
  end

  # 明日が締め切りのプロジェクト
  factory :project_due_tomorrow, class: Project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" } due_on { 1.day.from_now }
    association :owner
  end
end
````

こうすると上で定義した新しいファクトリを Project モデルのスペックで使うことができ ます。
be_late は RSpec に定義されている マッチャではありません。ですが RSpec は賢いので、project に late または late? という名前 の属性やメソッドが存在し、それが真偽値を返すようになっていれば be_late はメソッドや 属性の戻り値が true になっていることを検証してくれます。

```rb:spec/models/project_spec.rb
# 遅延ステータス
describe "late status" do
  # 締切日が過ぎていれば遅延していること
  it "is late when the due date is past today" do
    project = FactoryBot.create(:project_due_yesterday)
    expect(project).to be_late
  end

  # 締切日が今日ならスケジュールどおりであること
  it "is on time when the due date is today" do
    project = FactoryBot.create(:project_due_today)
    expect(project).to_not be_late
  end

  # 締切日が未来ならスケジュールどおりであるこ
  it "is on time when the due date is in the future" do
    project = FactoryBot.create(:project_due_tomorrow)
    expect(project).to_not be_late
  end
end
```

ですが、新しく作ったファクトリには大量の重複があります。新しいファクトリを定義す るときは毎回プロジェクトの全属性を再定義しなければいけません。これはつまり、Project モデルの属性を変更したときは毎回複数のファクトリ定義を変更する必要が出てくる、とい うことを意味しています。
Factory Bot には重複を減らすテクニックが二つあります。一つ目は ファクトリの継承 を使ってユニークな属性だけを変えることです。

```rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" }
    due_on { 1.week.from_now }
    association :owner

    # 昨日が締め切りのプロジェクト
    factory :project_due_yesterday do
      due_on { 1.day.ago }
    end

    # 今日が締め切りのプロジェクト
    factory :project_due_today do
      due_on { Date.current.in_time_zone }
    end

    # 明日が締め切りのプロジェクト
    factory :project_due_tomorrow do
      due_on { 1.day.from_now }
    end
  end
end
```

見た目には少しわかりづらいかもしれませんが、これが継承の使い方です。:project*due*-yesterday と :project_due_today と :project_due_tomorrow の各ファクトリは継承元となる :project ファクトリの内部で入れ子になっています。構造だけを抜き出すと次のようになり ます。

```
factory :project
factory :project_due_yesterday
factory :project_due_today
factory :project_due_tomorrow
```

継承を使うと class: Project の指定もなくすことができます。なぜならこの構造から Factory Bot は子ファクトリで Project クラスを使うことがわかるからです。この場合、スペッ ク側は何も変更しなくてもそのままでパスします。
重複を減らすための二つ目のテクニックは トレイト(trait) を使ってテストデータを構築 することです。このアプローチでは属性値の 集合 をファクトリで定義します。まず、プロジ ェクトファクトリの中身を更新します。

```rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Project #{n}" }
    description { "A test project." }
    due_on { 1.week.from_now }
    association :owner

    trait :with_notes do
      after(:create) { |project| create_list(:note, 5, project: project) }
    end

    trait :due_yesterday do
      due_on { 1.day.ago }
    end

    trait :due_today do
      due_on { Date.current.in_time_zone }
    end

    trait :due_tomorrow do
      due_on { 1.day.from_now }
    end

    trait :invalid do
      name { nil }
    end
  end
end
```

トレイトを使うためにはスペックを変更する必要があります。利用したいトレイトを使っ
て次のようにファクトリから新しいプロジェクトを作成します。

```rb:spec/factories/projects.rb
describe "late status" do
  it "is late when the due date is past today" do
    project = FactoryBot.create(:project, :due_yesterday)
    expect(project).to be_late
  end

  it "is on time when the due date is today" do
    project = FactoryBot.create(:project, :due_today)
    expect(project).to_not be_late
  end

  it "is on time when the due date is in the future" do
    project = FactoryBot.create(:project, :due_tomorrow)
    expect(project).to_not be_late
  end
end

it "can have many notes" do
  project = FactoryBot.create(:project, :with_notes)
  expect(project.notes.length).to eq 5
end
```

トレイトを使うことの利点は、複数のトレイトを組み合わせて複雑なオブジェクト を構築できる点です。
私の会社でもトレイトを採用しています。

## スペックを Dry に保つ

### let で遅延読み込みする

`spec/models/note_spec.rb`を対象に既存の before ブロックの実装から let メソッドに変えた時の挙動について解説します。
こうすることで Factorybot の一部のセットするデータを一部書き換えることもできます。

`before ブロックの場合`

```

require 'rails_helper'

RSpec.describe Note, type: :model do
before do
@user = User.create(
first_name: "Joe",
last_name: "Tester",
email: "joetester@example.com",
password: "dottle-nouveau-pavilion-tights-furze",
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

before ブロックを使うと describe や context ブロックの内部で、各テストの実行前に共通のインスタンス変数をセットアップできます。

この方法も悪くはないのですが、まだ解決できていない問題が二つあります。

第一に、before の中に書いたコードは describe や context の内部に書いたテストを実行するたびに毎回実行されます。これはテストに予期しない影響を及ぼす恐れがあります。また、そうした問題が起きない場合でも、使う必要のないデータを作成してテストを遅くする原因になることもあります。

第二に、要件が増えるにつれてテストの可読性を悪くします。

`letメソッドを使用した場合`

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

let は呼ばれて、初めて使われた時に実行されるメソッドです。

let を使うことで、before ブロックに処理を詰め込まず、必要なデータを必要なテストの中だけで呼び出せるので、テストの読みやすさ・メンテナンス性が上がります。

`user: user`で`let(:user)`を呼び出し、`user`の作成を行います。

` "is valid with a user, project, and message"`のテストが終わると、let で作成された値は、それぞれの it ブロックごとにスコープがリセットされます。したがって、ある it で作成された user は、他の it には影響を与えません

つまり、「let は呼ばれたときに初めて評価される（遅延評価される）」ため。let(:user) は user が実際に参照されるまで評価されない。

「テスト実行時に必要なときだけ呼び出されることで、無駄な DB アクセスが避けられる」ため、パフォーマンスと可読性が向上するといったメリットがあります。

### let!か before をちらを使うべきか

let は遅延評価を行うが、let!か before は即時評価を行います。
即時評価は、テスト実行前にその場ですぐ実行されるメソッドになります。
私の場合は可読性やデバッグのしやすさから実行順が明確な before を使うようにしています。
以下を例に解説します。

`let!の場合`

```

let!(:user) { create(:user) }
let!(:post) { create(:post, user:) }
let!(:comment) { create(:comment, post:) }

```

一見上から順に実行されそうに見えますが、実際の評価タイミングは RSpec 内部の定義処理順に依存していて、いつ実行されたかは見た目では判断できない。
つまり、見た目の順序と実行順が一致しない可能性がある。

`beforeの場合`

```

before do
user = create(:user)
post = create(:post, user: user)
comment = create(:comment, post: post)
end

```

1. user を作って
2. user を使って post を作って
3. post を使って comment を作る

という 処理の流れがそのままコードの見た目通り。
読む人が上から順にこうやって処理されるんだなとすぐ理解できます。
つまり、見た目の順序と実行順が一致する。

## さいごに

私の所属している会社では、基本的なルールとして[RuboCop RSpec ](https://docs.rubocop.org/rubocop-rspec/cops_rspec.html)のデフォルトルールに則るという方針を元に Rspec を書いています。
その他にも外部 API の呼び出しをモックする場合は、基本的に[WebMock](https://github.com/bblimke/webmock)を使ったりしています。

```

```
