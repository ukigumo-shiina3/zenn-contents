---
title: "『読む人の脳に優しいコード』とは何か？リーダブルコード要点まとめ"
emoji: "📘"
type: "tech"
topics:
  ["frontend", "技術書", "リーダブルコード", "コードレビュー", "コーディング"]
published: false
---

## はじめに

コードレビューを受ける中で、コード品質に関する指摘が多いことに気づきました。
名著である[リーダブルコード](https:#www.amazon.co.jp/%E3%83%AA%E3%83%BC%E3%83%80%E3%83%96%E3%83%AB%E3%82%B3%E3%83%BC%E3%83%89-%E2%80%95%E3%82%88%E3%82%8A%E8%89%AF%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E6%9B%B8%E3%81%8F%E3%81%9F%E3%82%81%E3%81%AE%E3%82%B7%E3%83%B3%E3%83%97%E3%83%AB%E3%81%A7%E5%AE%9F%E8%B7%B5%E7%9A%84%E3%81%AA%E3%83%86%E3%82%AF%E3%83%8B%E3%83%83%E3%82%AF-Theory-practice-Boswell/dp/4873115655)を読み、要点を整理してコード品質の向上に繋げたいと思っています。

この記事を読むことで、リーダブルコードの要点の把握と実際の活用例を学ぶことができます。

## 対象読者

- コード品質を向上させたい方
- リーダブルコードを読んでない方、または読み直したい方

## 名前に情報を加える

わかりやすい命名とは、「名前を見ただけで意味が伝わる」ことである。

**[名前を見ただけでは意味が伝わらない場合]**

```
a = Firebase::User.find(id)
foo = CSV.read(csv_path, headers: true)

user = Firebase::User.find(id)
data = CSV.read(csv_path, headers: true)
```

- `a`, `foo`
  → 情報がゼロ
- `user`, `data`
  → 抽象的すぎて文脈に依存する

**[何が問題か]**

- 何のユーザーか分からない
- 何の結果なのか分からない
- 読み手が毎回、前後の文脈を頭の中で補完する必要がある

**[名前を見ただけで意味が伝わる場合]**

```
firebase_user = Firebase::User.find(id)
csv_data = CSV.read(csv_path, headers: true)
```

名前に情報を含めることで、「何を表す変数なのか」を考えるコストが消える。

**[ポイント]**

- 短さより情報量
- 抽象名（user, data）は基本避ける
- 取得元・役割・状態を名前に含める

## 誤解されない名前

### 限界値の名前

**[限界値が曖昧な命名]**
リトライする回数が10回目だとする。

```
RETRY_COUNT_LIMIT = 10
```

limitは限界という意味だが、10を含めるのか含めないのかが明確にわからない。

**[限界値が明確な命名]**

```
MAX_RETRY_COUNT = 10
```

**[ポイント]**

- 限界値を明確にするには、**max**と**min**をつける

### 範囲指定の名前

```
array = %w[a b c d e]

start = 0
stop = 4

# 範囲内の値を返す処理
array[start...stop]
```

startは0だと分かるが、stopは[0,3]なのか[0,4]なのかが明確にわからない。

```
array = %w[a b c d e]

first = 0
last = 4

# 範囲内の値を返す処理
array[first...last]
```

4まで含めているという包含的な意味で使うのであればstopよりlastの方が適切である。

**[ポイント]**

- 範囲を指定するときは、**first**と**last**をつける

### ブール値の名前

```
is_published = true
is_visible = false
```

true / false が入ることが名前だけで分かる。
相反する状態（公開 / 非公開、表示 / 非表示）が想像しやすい。

```
has_permission = true
has_password = false
```

所有・付与・存在を表すときに使う。

**[ポイント]**

- true / false を取る変数は、状態が分かる名前にする。

## コメントすべきことを知る

### コメントすべきではないこと

【コードからすぐに推測できること】

```
# Companyクラスを定義
class Company

# 会社詳細を更新するメソッドを定義
def update_company_detail

# adminのIDが存在していたらnilを返却
 return nil if admin_id.present?
```

新しい情報を提供するわけでもなく、読み手がコードを理解しやすくなるためのコメントでもない。
コードから容易に推測できる場合はコメントは不要である。

【コードに改善が必要な時はコメントをつける】

```
# TODO: 毎回DBアクセスしてるので変えたい
```

よく使うアノテーションコメント記法は下記のテーブルにまとめています。

:::details アノテーションコメントとは

- コメントにメタデータを付加するもの
- コードの欠陥がわかりやすくなる
  :::

| 記法   | 典型的な意味合い         |
| ------ | ------------------------ |
| TODO:  | あとで手をつける         |
| FIXME: | 既知の不具合があるコード |
| HACK:  | あまり綺麗じゃない解決策 |
| XXX:   | 危険!大きな問題がある    |

### 読み手の立場になって考える

【要約コメントを残す】
コード量が多い場合は、処理の最初に要約コメントを残すことで、読み手に優しいコードにする。

```
# 有効な注文データを抽出し、請求処理用のハッシュに変換する
orders.each do |order|
  if order.paid?
    # 支払い済み注文の請求情報を生成
    invoice_items << build_paid_invoice(order)
  else
    # 未払い注文の督促用データを生成
    reminder_items << build_reminder(order)
  end
end
```

【ポイント】

- コードから容易に推測できる場合はコメントは不要である
- コードに改善が必要な場合は、アノテーションコメントを使用する
- 処理の最初に要約コメントを残すことで、読み手に優しいコードにする

## コメントは正確で簡潔に

### 曖昧な代名詞を避ける

【代名詞が入っている場合】

```
# ユーザーをDBに保存する。そのステータスを確認する
user.save!
update_user_status(user)
```

この場合、「そのステータス」がユーザーのステータスなのか保存結果のステータスなのか一瞬考える必要があります。

【代名詞を入れない場合】

```
# ユーザーをDBに保存する。ユーザーのステータスを確認する
user.save!
update_user_status(user)
```

「その」を「ユーザー」に変えることで、「その」が明確になり、何を指しているかが即座に分かります。

### コードの意図を書く

【コードの処理をただコメントした場合】

```
# ordersをcreated_atで並び替えて先頭を取得する
latest_order = orders.sort_by(&:created_at).last
```

【コードの意図を書く】

```
# 最新の注文を取得して管理画面に表示する
latest_order = orders.sort_by(&:created_at).last
```

【ポイント】

- 不要な代名詞は使わずに、対象を明確な単語に置き換える
- コードの意図を読み手にわかりやすく書く

## 制御フローを読みやすくする

### 条件式の引数の並び順は左側に「調査対象」で右側に「比較対象」

[調査対象が左側]

```
if user.age < 20
  restrict_access!
end
```

- 「user.age を調べている」ことが即座に分かる
- 読み手は左から自然に理解できる

```
if 20 > user.age
  restrict_access!
end
```

- 条件自体は同じだが「何を比較しているか」を一拍考える必要がある

| 左側                       | 右側                               |
| -------------------------- | ---------------------------------- |
| 「調査対象」の式。変化する | 「比較対象」の式。あまり変化しない |

### if/elseブロックの並び順は否定形よりも肯定形を使う

[否定形を使った場合]

```
if !user.active?
  # 第2のケース
else
  # 第1のケース
end
```

- 「アクティブでない場合」と頭の中で反転が必要
- 条件の意図が一瞬遅れる

[肯定形を使った場合]

```
if user.active?
  # 第1のケース
else
  # 第2のケース
end
```

- 条件の意図がわかりやすい

### ネストを浅くする

[ネストが深い場合]

```
if user_result == "SUCCESS"
  if permission_result != "SUCCESS"
    reply.write_errors("reading permissions")
    reply.done
    return false
  end

  reply.write_errors("")
else
  reply.write_errors("user_result")
end

reply.done
```

[ポイント]

- user_result と permission_result を頭に保持し続ける必要がある
- 正常系がネストの奥に埋もれ、処理の流れを追いづらくなる

[早めに返してネストを削除する]

```
if user_result != "SUCCESS"
  reply.write_errors("user_result")
  reply.done
  return false
end

if permission_result != "SUCCESS"
  reply.write_errors("reading permissions")
  reply.done
  return false
end

reply.write_errors("")
reply.done
```

[ポイント]

- 失敗条件を先に捨てる
- 正常系が下に一直線で読める

## 巨大な式を分割する

### 説明変数を利用する

【説明変数を利用しない場合】

```
if order.status == "paid" && order.total_price >= 10_000 && !order.canceled?
  process_shipment(order)
end
```

[ポイント]

- 条件を頭の中で分解しないと意味が取れない
- 「何を判定したいのか」が即座に伝わらない

【説明変数を利用する場合】

```
paid_order        = order.status == "paid"
expensive_order   = order.total_price >= 10_000
active_order      = !order.canceled?

if paid_order && expensive_order && active_order
  process_shipment(order)
end
```

[ポイント]

- 条件式や計算の「意図」が名前で即座に分かり、可読性が上がる
- 複雑な式を一度評価して名前に置くことで、読み手は「値」ではなく「意味」で考えられるため、認知負荷が下がる

### 要約変数を利用する

【要約変数を使わない場合】

```
if current_user.role == "admin" || current_user.id == post.author_id
  # 投稿を削除できる
end

if current_user.role != "admin" && current_user.id != post.author_id
  # 投稿は削除できない
end
```

[ポイント]

- 条件を毎回解釈する必要がある
- 何を判定したいのかが即座に見えない

【要約変数を使う場合】

```
can_delete_post =
  current_user.role == "admin" ||
  current_user.id == post.author_id

if can_delete_post
  # 投稿を削除できる
else
  # 投稿は削除できない
end
```

[ポイント]

- 思考の対象を「条件」から「意味」に変えられる
- 投稿を削除できるのかを考えるだけでよくなる

### 巨大な文を分割する

【コードの文を分割していない場合】

```
if user.profile.role == "admin"
  user.settings.permission_level = 10
  user.settings.max_projects = 100
elsif user.profile.role == "editor"
  user.settings.permission_level = 5
  user.settings.max_projects = 20
elsif user.profile.role == "viewer"
  user.settings.permission_level = 1
  user.settings.max_projects = 5
end
```

[ポイント]

- user.profile.roleとuser.settings.xxx を何度も読む
- 横に長く、目が滑る
- タイプミス耐性が低い

【コードの文を分割した場合】

```
role = user.profile.role
permission_level = user.settings.permission_level
max_projects = user.settings.max_projects

case role
when "admin"
  permission_level = 10
  max_projects = 100
when "editor"
  permission_level = 5
  max_projects = 20
when "viewer"
  permission_level = 1
  max_projects = 5
end

user.settings.permission_level = permission_level
user.settings.max_projects = max_projects
```

[ポイント]

- user.settings.permission_level を1回しか書かないため、タイプミスが局所化になる
- case による分岐が明確になる

## 変数と読みやすさ

### 不要な変数を削除する

【削除すべき変数】

```
def update_last_login(user)
  current_time = Time.current
  user.last_login_at = current_time
  user.save!
end
```

この場合current_timeは削除すべき変数になります。

[ポイント]

- Time.currentは十分に意味が明確
- 複雑な式ではない
- 1回しか使われていない
- 重複排除にもなっていない

【不要変数を削除】

```
def update_last_login(user)
  user.last_login_at = Time.current
  user.save!
end
```

[ポイント]

- 読み手がこの変数を解釈する必要がなくなり、認知負荷が下がる
- 1行で意図が完結し、処理の流れを追いやすい

### 変数のスコープを縮める

【グローバル変数を利用した場合】

```
# ファイル全体で共有されている状態
status = nil

def step1
  # statusを変更
  status = "initialized"
end

def step2
  # statusは使わない
end

def step3
  # statusは使わない
end

```

[ポイント]

- status が どこで読まれているか常に警戒する必要がある
- 実際は step1 でしか使われていないのに追跡コストが発生

【グローバル変数をローカル変数に置き換え】

```
def step1
  status = "initialized"
  status
end

def step2
  # statusは存在しない
end

def step3
  # statusは存在しない
end

```

[ポイント]

- status の影響範囲が step1の中だけ になる
- 他メソッドを読むときに余計な前提を持たなくてよい

## さいごに

この記事は、本書での特に重要なポイントを中心にまとめました。
今回紹介したポイントを意識すれば、より読みやすいコードが書けるようになります。
読みやすいコードは、自分だけでなく、チーム全体の生産性を確実に高めてくれます。
この記事が、日々書くコードを少しだけ脳に優しくするきっかけになれば幸いです。
最後まで読んでいただき、ありがとうございました。
