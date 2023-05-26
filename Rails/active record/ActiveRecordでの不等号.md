
### ◯◯より小さい (less than <)

### Rails(ActiveRecord)
```ruby
User.where(age: ...30)
```

```sql
-- SQL
SELECT "users".* FROM "users" WHERE "users"."age" < 30"
```

### ◯◯以下 (less than or equal <=)

### Rails(ActiveRecord)
```ruby
User.where(age: ..30)
```

```sql
-- SQL
SELECT "users".* FROM "users" WHERE "users"."age" <= 30"
```

### ◯◯以上 (greater than or equal >=)

### Rails(ActiveRecord)
```ruby
User.where(age: 11..)
```

```sql
-- SQL
SELECT "users".* FROM "users" WHERE "users"."age" >= 11"
```

### ◯◯よりも大きい (greater than >)

範囲オブジェクトを使った書き方では、今のところ以下の[SQL](http://d.hatena.ne.jp/keyword/SQL)クエリは発行できません。
```ruby
SELECT "users".* FROM "users" WHERE "users"."age" > 10"
```

なので、範囲オブジェクトを使う場合は◯◯以上 (greater than or equal >=)という[SQL](http://d.hatena.ne.jp/keyword/SQL)クエリに置き換える必要があります。ですが、あまり困ることもなさそうです。

### 範囲抽出(Beetween)

今まで通り

### Rails(ActiveRecord)
```ruby
User.where(age: 10..30)
```

```sql
-- SQL
SELECT "users".* FROM "users" WHERE "users"."age" BETWEEN 10 AND 30
```
と書くことができます。

### 範囲抽出 (◯以上□未満)

### Rails(ActiveRecord)
```ruby
User.where(age: 10...30)
```

```sql
-- SQL
SELECT "users".* FROM "users" WHERE "users"."age" >= 10 AND "users"."age" < 30
```

https://simple-minds-think-alike.moritamorie.com/entry/active-record-where-with-range