
ページング機能を追加するGem。Kaminari 1.2.1で確認。

以下のgemを追加する。

```
gem 'kaminari'
```

`bin/rails g kaminari:config`で設定ファイルを生成する。

## モデルメソッド

```
# ページを指定
User.page(1)
# ページと1ページに何件表示するか指定
User.page(1).per(10)
# 1ページに何件表示するか
User.page(1).limit_value
# ページ数
User.page(1).total_pages
# 現在のページ数
User.page(1).current_page
# 次のページ数。現在のページが最終ページならnilが返る
User.page(1).next_page
# 前のページ数。現在のページが最初のページならnilが返る
User.page(1).prev_page
# 最初のページならtrue
User.page(1).first_page?
# 最後のページならtrue。最終ページが10ページ目なら11ページ目以降はfalseになる
User.page(10).last_page?
# 最終ページが10ページ目なら11ページ目以降はtrueになる
User.page(11).out_of_range?
# 指定数分表示をずらす。
# 以下の例なら10件ずつ表示し2ページ目なので10-20件目の表示だが
# padding(5)があるので15-25件目が表示される
User.page(2).per(10).padding(5)
# 全件数。countだとページ内の件数になる
User.page(1).total_count

# 全件数を取得するクエリの発行を抑制する
# <Prev Next> のようなページングで全件数の取得が不要な場合に指定する
User.page(1).without_count

# 配列のページング
Kaminari.paginate_array([*1..10]).page(1).per(5)
```

## ヘルパーメソッド

```
@users = User.page(1)
# config/initializers/kaminari_config.rbの設定内容で表示
paginate(@users)
# windowを指定。kaminari_config.rbのwindowを参照
paginate(@users, window: 2)
# outer_windowを指定。kaminari_config.rbのouter_windowを参照
paginate(@users, outer_window: 2)
# leftを指定。kaminari_config.rbのleftを参照
paginate(@users, left: 2)
# rightを指定。kaminari_config.rbのrightを参照
paginate(@users, right: 2)
# param_nameを指定。kaminari_config.rbのparam_nameを参照
paginate(@users, param_name: :no)
# コントローラーやアクション、クエリパラメータの変更や追加
paginate(@users, params: { controller: :foos, action: :foo_action, xx: 1 })
# Ajax化
paginate(@users, remote: true)
# ページングビューのprefixを指定
paginate(@users, views_prefix: 'directory')

# 次のページのリンク
link_to_next_page(@users, '次のページ')
# 前のページのリンク
link_to_prev_page(@users, '前のページ')
# 何件中 何件目から何件目を表示　というような感じの情報を出力
page_entries_info(@users, entry_name: 'ユーザー')
```

## 設定

`config/initializers/kaminari_config.rb`を書き換える。

```
Kaminari.configure do |config|
  # デフォルトの1ページの表示件数
  config.default_per_page = 25

  # perメソッドで指定できる1ページの表示件数の上限
  config.max_per_page = nil

  # 現在8ページだとして、左右に表示するページリンクの個数を指定する
  # 4だと以下のような感じ
  # <Prev …4 5 6 7 8 9 10 11 12 …Next>
  config.window = 4

  # 最初と最後のページリンクをいくつ表示するか
  # 2だと以下のような感じ
  # <Prev 1 2 …（略） …30 31 Next>
  config.outer_window = 0

  # 最初のページリンクをいくつ表示するか
  # 2だと以下のような感じ
  # <Prev 1 2 …（略） …Next>
  # outer_windowとleftで大きい方が有効になる
  config.left = 0

  # 最後のページリンクをいくつ表示するか
  # 2だと以下のような感じ
  # <Prev …（略） …30 31 Next>
  # outer_windowとrightで大きい方が有効になる
  config.right = 0

  # pageメソッドの名前を変える
  config.page_method_name = :page

  # クエリパラメータのパラメータ名を変える（params[:page]）
  config.param_name = :page

  # ページ数の上限。あくまでViewに表示するページ数なので、
  # 現実的にページ数を制限したいならControllerで制限をかけなければならない
  config.max_pages = nil

  # ransack_memoryというgemを使っているときに挙動がおかしい場合にtrueにする
  # もともとは1ページ目のリンクのパラメータ消失を回避するためのオプションらしいのだが、
  # falseでも特にパラメータは消えないのでよくわからない
  config.params_on_first_page = false
end
```

### モデル個別での指定

```
class User < ApplicationRecord
  # 1ページに表示する件数。default_per_pageを上書きする
  paginates_per 10
  # perメソッドで指定できる1ページの表示件数の上限。max_per_pageを上書きする
  max_paginates_per 50
end
```

### 多言語化

[I18n and Labels : GitHub - kaminari/kaminari](https://github.com/kaminari/kaminari#i18n-and-labels)を参照。

[Kaminari公式の多言語化ファイル](https://github.com/kaminari/kaminari/wiki/Internationalization-and-Locales)、多言語化Gemの[kaminari-i18n](https://github.com/tigrish/kaminari-i18n)もある。

## Viewのカスタマイズ

`bin/rails g kaminari:views default`で`app/views/kaminari/*.html.erb`ファイルが作られる。`bin/rails g kaminari:views`を実行すると`default`の部分に指定できる他のテーマが一覧表示される。

`bin/rails g kaminari:views default --views-prefix admin`とすると`app/views/admin/kaminari/*.html.erb`ファイルが作られる。このViewを使うには`paginate(@users, views_prefix: 'admin')`としなければならない。

`app/views/kaminari/custom_theme`ディレクトリを作り、`app/views/admin/kaminari/*.html.erb`ファイルのコピーを格納すると、`paginate(@users, theme: 'custom_theme')`という方法でViewを切り替えることができる。


https://linyclar.github.io/rails_memos/libraries/kaminari/