
## enum_helpを使用したプルダウンのセレクトボックスの実装

```ruby
#権限選択画面のコード
<%= f.select :role_eq, User.roles_i18n.invert.map{|key, value| [key, User.roles[value]]}, { include_blank: t(‘defaults.unspecified’) }, { class: ‘form-control mr-1’ } %>
```

`User.roles_i18n.invert.map{|key, value| [key,　User.roles[value]]}`の部分が非常にややこしいので順に読み解いていく。

まずは[enum](http://d.hatena.ne.jp/keyword/enum)_helpを使って日本語化する前のコード

```ruby
<%= f.select :role_eq, User.roles.values %>
```

```
#rails console内
User.roles
=> {"general"=>0, "admin"=>1}

User.roles.values
=> [0, 1]
```

`User.roles.values`により、roleカラムの値を配列として取得できる。 これを、[enum](http://d.hatena.ne.jp/keyword/enum)_helpを使用し[i18n](http://d.hatena.ne.jp/keyword/i18n)で日本語化する。

```ruby
#rails.console内
User.roles_i18n
=> {"general"=>"一般", "admin"=>"管理者"}

User.roles_i18n.values
=> ["一般", "管理者"]
```

```ruby
# admin/users/index.html.erb内
<%= search_form_for(@q, url: admin_users_path, method: :get) do |f| %>
  <div class="input-group mb-3">
      <%= f.label :role_eq, '権限' %>
      <%= f.select :role_eq, User.roles_i18n.values, include_blank: '指定なし' %>
  </div>
<% end %>
```

しかし、このコードだと発行される[SQL](http://d.hatena.ne.jp/keyword/SQL)が常に`”role” = 0`となってしまい正常に動作しない。

```shell
Processing by Admin::UsersController#index as HTML
  Parameters: {"utf8"=>"✓", "q"=>{"first_name_or_last_name_cont"=>"","role_eq"=>"管理者"}, "commit"=>"検索"}
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 8], ["LIMIT", 1]]
  ↳ vendor/bundle/ruby/2.6.0/gems/activerecord-5.2.3/lib/active_record/log_subscriber.rb:98
  Rendering admin/users/index.html.erb within layouts/admin_application
  User Load (0.3ms)  SELECT DISTINCT "users".* FROM "users" WHERE "users"."role" = 0 ORDER BY "users"."created_at" DESC
  ↳ app/views/admin/users/index.html.erb:23
```

どうやらこれは、ransackがそもそも[enum](http://d.hatena.ne.jp/keyword/enum)に対応していない事が原因で、status_idが常に0になってしまうというバグらしい。  

> 参考記事  
[RansackはRailsのenumに対応していないっぽい | 地方でリモートワーク](https://www.tom08.net/entry/2016/12/05/121746)  


そこで、invertメソッドとmapメソッドを組み合わせてstatus_idを正しく取得できるようにする。

Invertメソッドとmapメソッドの挙動について
```ruby
#rails.console内
User.roles_i18n
=> {"general"=>"一般", "admin"=>"管理者"}

User.roles_i18n.invert
=> {"一般"=>"general", "管理者"=>"admin"}

User.roles_i18n.invert.map{|key,value| [key, User.roles[value]]}
=> [["一般", 0], ["管理者", 1]]
```

３つ目の挙動が少し難しいかもしれないが、これは、 “一般” “管理者”というキーが key に、”general” “admin”という値が [value](http://d.hatena.ne.jp/keyword/value)にそれぞれ代入され、mapメソッドによってそれぞれに対して処理が行われている。  
  
>参考記事  
[ハッシュのキー、値の配列を取得する (keys, values) | まくまくRubyノート](https://maku77.github.io/ruby/hash/keys-values.html)  
  
```ruby
#rails.console内
User.roles["general"]
=> 0

User.roles["admin"]
=> 1
```

この結果、一般タブが選択されると`value=“0”`が、管理者タブが選択されると`value="1"`が選択されるようになり、正常に動作するようになった。

```ruby
#発行されているhtml
<select name="q[role_eq]" id="q_role_eq"><option value="">指定なし</option>
<option value="0">一般</option>
<option value="1">管理者</option></select>
```
