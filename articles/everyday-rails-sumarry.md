---
title: "Everyday Railsのエッセンス凝縮！最速で理解するRSpecモデルテスト"
emoji: "🧪"
type: "tech"
topics: ["rspec", "test", "ruby", "rails"]
published: true
---

## はじめに

RSpecを体系的に学ぶために、[Everyday Rails - RSpec による Rails テスト入門](https://leanpub.com/everydayrailsrspec-jp)から、業務で使用している項目を抜粋してまとめた記事です。

私の会社では[GraphQL Ruby](https://graphql-ruby.org/)を採用しているため、この記事ではモデルスペックに焦点を当てるため、以下のような理由からコントローラーやAPI、システムスペックについての解説は割愛しています。

- GraphQLではGraphqlController のみを使用しており、一般的なコントローラーは存在しないため、テスト対象外としています。
- エンドポイントが/graphql の 1 つに集約されるため、REST APIにおけるリクエストスペックも対象外としています。

## 対象読者

- これからRSpecを学びたい方
- RSpecは使用したことはあるが、より理解を深めたい方

## モデルスペック

モデルのテストは、アプリケーションのコアとなる部分をテストすることになるため、一番学習しやすいテストになります。

### バリデーションテスト

```rb:spec/models/user_spec.rb
RSpec.describe User, type: :model do # 姓、名、メール、パスワードがあれば有効な状態であること
it "is valid with a first name, last name, email, and password" do
  user = User.new(
    first_name: "Aaron",
    last_name: "Sumner",
    email: "tester@example.com",
    password: "dottle-nouveau-pavilion-tights-furze",
  )
  expect(user).to be_valid
end
```

- このケースではbe_validというRSpecのマッチャを使って、モデルが有効な状態かどうかを検証しています。
- trueもしくはfalseになることを検証し、今回は戻り値がtrueになることを期待しています。
- Userモデルに定義されているバリデーションが、全て満たされているため、テストはパスします。

:::details マッチャ
「期待値と実際の値を比較して、一致した（もしくは一致しなかった）という結果を返すオブジェクト」のこと
:::

```rb:spec/models/user_spec.rb
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
繰り返し可能な値の中に、ある値が存在するかどうかをチェックする
:::

### テストが失敗するケースを確認

また自分が作成したテストケースが失敗することをコードを変えてみて、テストが意図して動いていることを確認する」必要があります。

例えば先ほどのテストケースのexpect(user.errors[:first_name]).to include("can't be blank")の箇所をexpect(user.errors[:first_name]).to_not include("can't be blank")と`to`から`to_not`に変えてみることでテストが意図して失敗するかを検証してみることは非常に重要です。
また、先ほどのテストコードの変更を元に戻し(`to_not`を`to`に戻す)、Userモデルのfirst_nameのバリデーションをコメントアウトして、テストを実行することでもテストの失敗を確認することができる

### build と create の違い

```rb:spec/models/project_spec.rb
RSpec.describe Project, type: :model do
# ユーザー単位では重複したプロジェクト名を許可しないこと
  it "does not allow duplicate project names per user" do
    user = User.create(
    first_name: "Joe",
    last_name: "Tester",
    email: "joetester@example.com",
    password: "dottle-nouveau-pavilion-tights-furze",
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

ここでテストしたいのは、一人のユーザーは同じ名前で二つのプロジェクトを作成できないが、ユーザーが異なるときは同じ名前のプロジェクトを作成できる、という要件です。
ちなみにbuildで作成している時とcreateで作成している箇所がありますが、違いは何になるか、下記に表してみました。

**build**
buildは、オブジェクトをメモリ上に生成しますが、データベースには保存しません。

- **用途:**
  - オブジェクトの属性や関連付けを確認するテストに適しています。
  - データベースへの保存が不要なテスト、例えばモデルのバリデーションのテストなどで使用されます。
  - データベースへの保存を行わない為、テストの速度が create に比べて速いです。
- **利点:**
  - テストの高速化。
  - データベースの状態を汚染しない。

**create**
create は、オブジェクトを生成し、データベースに保存します。

- **用途:**
  - データベースに保存されたオブジェクトの存在や状態を確認するテストに適しています。
  - 統合テストやシステムテストなど、データベースとの連携が必要なテストで使用されます。
- **利点:**
  - データベースの状態を実際に確認できる。
  - 他のオブジェクトとの関連付けを永続化できる。

今回のテストでは「ユーザー単位では重複したプロジェクト名を許可しないこと」をテストするので、まずcreateでTestProjectを作成してデータベースに保存した上で、buildしてTestProjectを作成して重複しているかのチェックを行うデータの準備をしています。
2つ目にTestProjectを作成した場合はバリデーションのチェックのためだけに作成できていればいいので、データベースの保存は必要なくbuildの作成の方が適しているという考えになります。

## FactoryBot

FactoryBotは、テスト用のダミーデータ（テストデータ）を簡単に作成するためのライブラリです。
RailsアプリのUserなどのモデルを、テストごとにサクッと作れるようになります。

```rb:spec/factories/users.rb
FactoryBot.define do
  factory :user do
    first_name { "Aaron" }
    last_name { "Sumner" }
    email { "tester@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end

```

このようにUserのFactoryBotを追加することで、テスト内でFactoryBot.create(:user)と書くだけで、簡単に新しいユーザーを作成できるようになります。
作成されるユーザーの名前は、毎回基本的にAaronSumnerになります。メールアドレスやパスワードも最初から設定された状態になります。
もちろん、文字列だけでなく、整数やブーリアン、日付など、属性に渡せるものなら何でも渡すことができます。

`FactoryBotを使った場合`

```rb:spec/models/user_spec.rb
it "has a valid factory" do
  expect(FactoryBot.build(:user)).to be_valid
end
```

`FactoryBotを使わない場合`

```
it "is valid with a first name, last name, email, and password" do
  user = User.new(
    first_name: "Aaron",
    last_name: "Sumner",
    email: "tester@example.com",
    password: "dottle-nouveau-pavilion-tights-furze",
  )
  expect(user).to be_valid
end

```

FactoryBotを使った場合と使わない場合を実際に見比べてみることで、ファクトリ版の方がより簡潔になっていることがわかると思います。

### 属性をオーバーライドする

```rb:spec/models/user_spec.rb
# 名がなければ無効な状態であること
it "is invalid without a first name" do
  user = FactoryBot.build(:user, first_name: nil)
  user.valid?
  expect(user.errors[:first_name]).to include("can't be blank")
end

```

ここではFactoryBotで定義してある`first_name: "Aaron"`を`first_name: nil`と指定して、オーバーライドしています。

### シーケンスを使ってユニークなデータを生成する

ファクトリで複数のユーザーをセットアップする必要が出てきた場合は実際のテストコードが走る前に例外が発生します。
たとえば以下のような場合です。

```

# 複数のユーザーで何かする
it "does something with multiple users" do
  user1 = FactoryBot.create(:user)
  user2 = FactoryBot.create(:user)
  expect(true).to be_truthy
end

```

1回目と2回目で同じメールアドレスが使われるため、一意制約に引っかかりバリデーションエラーが発生します。

```
Failures:
1. User does something with multiple users
   Failure/Error: user2 = FactoryBot.create(:user)

   ActiveRecord::RecordInvalid:
    Validation failed: Email has already been taken
```

FactoryBotではシーケンスを使ってこのようなユニークバリデーションを持つフィールドを扱うことができます。
シーケンスはファクトリから新しいオブジェクトを作成するたびに、カウンタの値を1つずつ増やしながら、ユニークにならなければいけない属性に値を設定します。
ファクトリ内にシーケンスを作成して実際に使ってみます。

```rb:spec/factories/users.rb
FactoryBot.define do
  factory :user do
    first_name { "Aaron" }
    last_name { "Sumner" }
    sequence(:email) { |n| "tester#{n}@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

メール文字列にnの値が入ることで重複を防ぐことができます。
こうすれば新しいユーザーを作成するたびに、tester1@example.com、tester2@example.comというように、ユニークで連続したメールアドレスが設定されます。

### ファクトリで関連を扱う

メモとプロジェクトのファクトリを作成し、FactoryBotで他のモデルと関連を持つモデルを扱ってみます。

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
ファクトリに名前を付けている2行目に、以下に示すようなownerという別名(alias)を付けてみます。

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
しかし、テストのたびにいちいち手作業でプロジェクトとユーザーを作りたくありません。
ここで作成したいのは、メモそのものだけです。

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

ここではFactoryBotを1回しか呼んでいないにもかかわらず、テストの実行結果を見ると必要なデータが全部作成されています。

```
Note
This　note's project is #<Project id: 1, name: "Test Project 1",
description: "Sample project for testing purposes", due_on:
"2017-01-17", created_at: "2017-01-10 04:01:24", updated_at:
"2017-01-10 04:01:24", user_id: 1>
This note's user is #<User id: 2, email: "tester2@example.com", created_at: "2017\
-01-10 04:01:24", updated_at: "2017-01-10 04:01:24",
first_name: "Aaron", last_name: "Sumner">
```

ですが、この例はファクトリで関連を扱う際の落とし穴が潜んでいます。
ユーザーのメールアドレスをよく見ると、tester1@example.comではなく、tester2@example.comになっています。
この理由はメモのファクトリが関連するプロジェクトを作成する際に関連するユーザー(プロジェクトに関連するowner)を作成し、それから2番目のユーザー(メモに関連するユーザー)を作成するからです。

:::details どのような順番で作成されているのか？

1. note を作るには project が必要（association :project）
2. project を作るには owner が必要（association :owner）
3. owner は user ファクトリの alias により user として解決される → User① が生成される
4. さらに note 自身が association :user を持っているため、もう1人のユーザー User② が新たに生成される
   :::

この問題を回避するためにメモのファクトリを次のように更新します。
こうするとデフォルトでユーザーが1人しか作成されなくなります。

```rb:spec/factories/notes.rb
FactoryBot.define do
  factory :note do
    message { "My important note." }
    association :project
    user { project.owner }
  end
end
```

スペックの結果を見てもユーザー 1人だけになります。

:::details どのような作成に変わったのか？

1. projectを生成（owner: user）
2. note.userにそのままproject.ownerを割り当てる
   :::

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
Userファクトリに追加したaliasに戻ってみます。
Projectモデルを見ると、Userの関連はownerという名前になっているのがわかると思います。

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

このようにFactoryBotを使う際はユーザーファクトリに対してownerという名前で参照される場合があると伝えなくてはいけません。
そのために使うのがaliasです。

### ファクトリ内の重複をなくす

FactoryBotでは同じ型を作成するファクトリを複数定義することもできます。
たとえば、スケジュールどおりのプロジェクトとスケジュールから遅れているプロジェクトをテストしたいのであれば、別々の名前を付けてプロジェクトファクトリの引数に渡すことができます。
その際はそのファクトリを使って作成するインスタンスのクラス名と、既存のファクトリと異なるインスタンスの属性値(この例でいうとdue_on 属性の値)も指定します。
また、`1.week.from_now`や`1.day.ago`は[ActiveSupport](https://api.rubyonrails.org/classes/ActiveSupport/Duration.html#method-i-ago)によって利用可能になっています。

```rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" }
    due_on { 1.week.from_now }
    association :owner
  end

  # 昨日が締め切りのプロジェクト
  factory :project_due_yesterday, class: Project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" }
    due_on { 1.day.ago }
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
    description { "Sample project for testing purposes" }
    due_on { 1.day.from_now }
    association :owner
  end
end
```

こうすると上で定義した新しいファクトリをProjectモデルのスペックで使うことができます。
`be_late`はRSpecのマッチャではありません。
[プレディケートマッチャ](https://rspec.info/features/3-13/rspec-expectations/built-in-matchers/predicates/)という仕組みを利用し、projectにlateまたはlate?という名前の属性やメソッドが存在し、それが真偽値を返すようになっていれば`be_late`はメソッドや属性の戻り値がtrueになっていることを検証してくれます。
この仕組みのメリットとしては、expect(project).to be\*late という自然言語っぽい書き方ができたり、`be_〇〇`という形式で、メソッド名をテストコードに書かずに表現できることです。

:::details このケースでの解釈のされ方
`expect(project).to be_late`

RSpec が内部的に下記のように解釈する

`expect(project.late?).to be true`
:::

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

ですが、新しく作ったファクトリには大量の重複があります。
新しいファクトリを定義するときは毎回プロジェクトの全属性を再定義しなければいけません。
つまり、Projectモデルの属性を変更したときは毎回複数のファクトリ定義を変更する必要が出てくる、ということを意味しています。
FactoryBotには重複を減らすテクニックが二つあります。
一つ目はファクトリの継承を使ってユニークな属性だけを変えることです。

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

見た目には少しわかりづらいですが、これが継承の使い方になります。
:project_due_yesterdayと:project_due_todayと:project_due_tomorrowの各ファクトリは継承元となる:projectファクトリの内部で入れ子になっています。
構造だけを抜き出すと次のようになります。

```
factory :project
  factory :project_due_yesterday
  factory :project_due_today
  factory :project_due_tomorrow
```

継承を使うとclass:Projectの指定もなくすことができます。
なぜならこの構造からFactoryBotは子ファクトリでProjectクラスを使うことがわかるからです。
この場合、スペック側は何も変更しなくてもそのままでパスします。
重複を減らすための二つ目のテクニックはトレイト(trait)を使ってテストデータを構築することです。
このアプローチでは属性値の集合をファクトリで定義します。
まず、プロジェクトファクトリの中身を更新します。

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

トレイトを使うためにはスペックを変更する必要があります。
利用したいトレイトを使って次のようにファクトリから新しいプロジェクトを作成します。

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

トレイトを使うことの利点は、複数のトレイトを組み合わせて複雑なオブジェクトを構築でき、簡潔に書くことができる点です！
私の会社でもトレイトを採用しています。

### コールバック

コールバックを使うと、ファクトリがオブジェクトをcreateする前、もしくはcreateした後に何かしら追加のアクションを実行できます。
また、createされたときだけでなく、buildされたり、stubされたりしたときも同じように使えます。
適切にコールバックを使えば複雑なテストシナリオも簡単にセットアップできます。
しかし、一方でコールバックは遅いテストや無駄に複雑なテストの原因になることもありますので、注意して使ってください。

ここでは複雑な関連を持つオブジェクトを作成する方法を説明します。
FactoryBotにはこうした処理を簡単に行うためのcreate_listメソッドが用意されています。
コールバックを利用して、新しいオブジェクトが作成されたら自動的に複数のメモを作成する処理を追加してみましょう。
今回は必要なときにだけコールバックを利用するよう、トレイトの中でコールバックを使います。

```rb:spec/factories/projects.rb
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "Sample project for testing purposes" }
    due_on { 1.week.from_now }
    association :owner

    # メモ付きのプロジェクト
    trait :with_notes do
      after(:create) { |project| create_list(:note, 5, project: project) }
    end
  end
end
```

create_listメソッドではモデルを作成するために関連するモデルが必要です。
今回はメモの作成に必要なProjectモデルを使っています。
プロジェクトファクトリに新しく定義したwith_notesトレイトは、新しいプロジェクトを作成した後にメモファクトリを使って5つの新しいメモを追加します。
それではスペック内でこのトレイトを使う方法を見てみましょう。
最初はトレイトなしのファクトリを使ってみます。

```rb:spec/models/project_spec.rb
# たくさんのメモが付いていること
it "can have many notes" do
  project = FactoryBot.create(:project)
  expect(project.notes.length).to eq 5
end
```

このテストは失敗します。
メモの数が5件ではなく0件という結果になります。
理由は、:with_notesトレイトを使っていないため、after(:create)で定義したcreate_list(:note, 5 ,project:project)が呼び出されていないからです。

```
Failures:

1) Project can have many notes
  Failure/Error: expect(project.notes.length).to eq 5

  expected: 5
       got: 0

  (compared using ==)
  # ./spec/models/project_spec.rb:69:in `block (2 levels) in <top
  (required)>'
  # 以下略
```

そこで with_notes トレイトでセットアップした新しいコールバックを使って、このテストをパスさせてみます。

```rb:spec/models/project_spec.rb
# たくさんのメモが付いていること
it "can have many notes" do
  project = FactoryBot.create(:project, :with_notes)
  expect(project.notes.length).to eq 5
end
```

これでテストがパスします。
コールバックによってプロジェクトに関連する5つのメモが作成することができました。
FactoryBotはトレイトを明示的に指定しない限り実行しません。
with_notesはオプションであり、明示的に指定されたときだけafter(:create)が走ります。

## スペックを Dry に保つ

### let で遅延読み込みする

`spec/models/note_spec.rb`を対象に既存のbeforeブロックの実装からletメソッドに変えた時の挙動について解説します。
こうすることでFactoryBotの一部のセットするデータを一部書き換えることもできます。

`before ブロックの場合`

```rb:spec/models/task_spec.rb
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

beforeブロックを使うとdescribeやcontextブロックの内部で、各テストの実行前に共通のインスタンス変数をセットアップできます。
この方法にも利点はありますが、いくつかの課題が残ります
全体のbeforeブロックはRSpec.describeNoteの直下にあるため、このブロックの中身はこのファイル内のすべてのitに対して毎回実行されます。
またdescribeの中のbeforeブロックはその中の context ブロック内の各テスト実行前に、毎回実行されます。

これら before ブロックの中で定義された create や project.notes.create(...) のような処理は、該当する it ブロックが実行されるたびに毎回評価されてデータベースに書き込まれるということです。

そのため下記のような影響が生まれます。

- 不要なテストでも重いデータを毎回用意してしまい無駄になる
- テストが遅くなる
- テスト結果が依存して壊れやすくなる

`letメソッドを使用した場合`

```rb:spec/models/note_spec.rb

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

letは呼ばれて、初めて使われた時に実行されるメソッドです。
letを使うことで、beforeブロックに処理を詰め込まず、必要なデータを必要なテストの中だけで呼び出せるので、テストの読みやすさ・メンテナンス性が上がります。
`user: user`で`let(:user)`を呼び出し、`user`の作成を行います。
` "is valid with a user, project, and message"`のテストが終わると、letで作成された値は、それぞれのitブロックごとにスコープがリセットされます。
したがって、あるitで作成されたuserは、他のitには影響を与えません
つまり、「letは呼ばれたときに初めて評価される（遅延評価される）」ため。let(:user)はuserが実際に参照されるまで評価されない。
「テスト実行時に必要なときだけ呼び出されることで、無駄なDBアクセスが避けられる」ため、パフォーマンスと可読性が向上するといったメリットがあります。

### let!か before をちらを使うべきか

letは遅延評価を行うが、let!かbeforeは即時評価を行います。
即時評価は、テスト実行前にその場ですぐ実行されるメソッドになります。
私の場合は可読性やデバッグのしやすさから実行順が明確なbeforeを使うようにしています。
以下を例に解説します。

`let!の場合`

```
let!(:user) { create(:user) }
let!(:post) { create(:post, user:) }
let!(:comment) { create(:comment, post:) }
```

一見上から順に実行されそうに見えますが、実際の評価タイミングはRSpec内部の定義処理順に依存していて、いつ実行されたかは見た目では判断できない。
つまり、見た目の順序と実行順が一致しない可能性がある。

`beforeの場合`

```
before do
  user = create(:user)
  post = create(:post, user:)
  comment = create(:comment, post:)
end
```

1. userを作って
2. userを使ってpostを作って
3. postを使ってcommentを作る

という 処理の流れがそのままコードの見た目通り。
読者が処理の流れを視覚的に理解しやすくなります。
つまり、見た目の順序と実行順が一致する。

## さいごに

今回は、実務に直結する RSpec のモデルテストについて効率的に学べる方法をご紹介しました。

私自身、テストケースを洗い出すときには[マインドマップから始めるソフトウェアテスト](https://www.amazon.co.jp/%E3%83%9E%E3%82%A4%E3%83%B3%E3%83%89%E3%83%9E%E3%83%83%E3%83%97%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%E3%83%86%E3%82%B9%E3%83%88-%E6%B1%A0%E7%94%B0-%E6%9A%81/dp/4774131318)という書籍の考え方をベースに、マインドマップを使って思考を整理しています。ツールとしては、シンプルで使いやすい[MindMeiste](https://mindmeister.jp/)を愛用しています。

また、私の所属している会社では[RuboCop RSpec ](https://docs.rubocop.org/rubocop-rspec/cops_rspec.html)のデフォルトルールに沿って記述する方針を採用しており、外部 API のモックには[WebMock](https://github.com/bblimke/webmock)を利用しています。

とはいえ、最も重要なのは「とにかく書いてみること」です。繰り返し書いていくことで、だんだんと感覚が身についていきます。
本記事が、その第一歩の手助けになれば幸いです。

最後まで読んでいただき、ありがとうございました！
