
`Pundit`とは｢認可｣の仕組みを提供するもの。  
ユーザーによってページの表示の許可・拒否をしたり､表示情報の範囲を変えられるgem。

分かりやすいイメージで言うと

・未ログインの場合に、ログインが必要なページをリクエストすると、ログインページへ遷移  
・ログイン済みの場合に、許可されていないページをリクエストすると、ルートURLへ遷移、などが挙げられる。

`Pundit`は`current_user`(ログイン中のユーザー)メソッドを扱うので､`sorcery gem`などが｢認証｣の仕組みありきで、認可は認証に依存している。

## Punditの仕組み

`Pundit`はコントローラの各アクションで`authorize`リソースオブジェクトを呼ぶと､**対象のリソースに対して権限があるかどうかを確認してくれる｡**
**その設定を`app/policies`にあるポリシーファイルで細かく定義できる｡

## Punditの使用法

Gemfileに`gem "pundit"`と記入し、`bundle install`を実施。
`Pundit`を適用したいコントローラの継承元で`include`を記入。

app/controllers/application_controller.rb
```ruby
class ApplicationController < ActionController::Base
  include Pundit
end
```

認可のルールを記述するファイルを作成するために、下記コマンドを入力。
すると`app/policies/application_policy.rb`というファイルが生成される。

```
$ rails g pudit:install
```


app/policies/application_policy.rb
```ruby
class ApplicationPolicy
  attr_reader :user, :record #読み取りの属性を定義している

  def initialize(user, record)
    @user = user
    @record = record
  end

  def index?
    false
  end
end
```

このファイルで定義している`ApplicationPolicyクラス`を継承して､他のコントローラごとの認可ルールを記述していく｡
`initialize`で定義される`user`は、デフォルトで`current_user`が引数に割り当てられるようになっている。
第二引数の`record`は認可をチェックしたいモデルオブジェクト。対応するモデルのインスタンスを手動で割り当て｡
これを利用することで､アクセスしているユーザーオブジェクトと､対象のリソースオブジェクトを知ることができる｡

## policyファイルの設定

今回権限を調整したい`Tag`, `Author`, `Category`モデルはSTI(単一テーブル継承)なので、継承元の`Taxonomy`モデルに対してpolicyを作成しましょう。

```ruby
class TaxonomyPolicy < ApplicationPolicy
  def index?
    user.admin? || user.editor?
  end

  def create?
    user.admin? || user.editor?
  end

  def update?
    user.admin? || user.editor?
  end

  def destroy?
    user.admin? || user.editor?
  end
end
```

policyファイルを適用するには、コントローラ側からPunditを呼び出しましょう。

```ruby
class Admin::TagsController < ApplicationController

  def index
    @tags = Tag.all.order(:slug)
    authorize(Tag)               # インスタンスモデルを利用しない(policy側ではuserの情報のみ扱う)場合に限り、authorize(Tag) という記述が可能。
    
    @tag = Tag.new
  end
  
  -----------一部省略------------------
 
end
```

https://osamudaira.com/378/
https://qiita.com/mmaumtjgj/items/c7fc40619a15cce5ccfc