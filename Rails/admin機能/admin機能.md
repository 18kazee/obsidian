-   AdminLTEバージョン3をyarnでインストール
-   管理者画面用のマニフェストファイルを作成
-   マニフェストファイルの読み込み設定
-   usersテーブルに管理者権限を判定する為のroleカラムを追加
-   管理者用のコントローラを作成(`admin/base_controller`)
-   管理画面用のトップページに遷移するコントローラーを作成(`admin/dashboards_controller.rb`)
-   管理者ログイン機能を担うコントローラを作成(`admin/user_sessions_controller`)
-   ルーティングの設定
-   ビューファイルの作成
-   管理画面用のページタイトルの設定
-   管理者ログイン機能を作成
-   ローケルファイル追記

## AdminLTEバージョン3をyarnでインストール

[こちら](https://adminlte.io/docs/3.0/index.html)を参考

```zsh
$ yarn add admin-lte@^3.1
```

`node_modules/admin-lte` ディレクトリ内に、多数テンプレートファイルが用意されている。
その中の`starter.html` を使用。　　

## 管理者画面用のマニフェストファイルを作成

今まで一般ユーザのマニフェストファイルは`app/assets/stylesheets/application.scss`と`app/assets/javascripts/application.js` を使用。
管理者画面は一般ユーザの画面と構成や見た目が大きく異なる事から、別々に管理するのが一般的。
[[マニフェストファイル]]

### admin.jsの設定

```js
//= require jquery3
//= require jquery_ujs

//= require admin-lte/plugins/bootstrap/js/bootstrap.bundle.min       # 拡張子は省略可能
//= require admin-lte/dist/js/adminlte.min
```
-   `//= require rails_ujs` だと現在はエラ－が起こるようで、`//= require jquery_ujs` を記述する必要がある。
参考：[](https://qiita.com/Statham/items/372234e23749ff1f6bf8)[https://qiita.com/Statham/items/372234e23749ff1f6bf8](https://qiita.com/Statham/items/372234e23749ff1f6bf8)

-   `//= require admin-lte/plugins/jquery/jquery.min` は必要ない
こちらはjQuery本体を読み込む記述なので、Gemfileに記述してインストールしているgemの`jquery-rails`と重複しているので、不要です。
`//= require jquery3`は、`jquery-rails` を利用するための記述です。

-   「.min」はminifyを表す
`.min` はパソコンにとって可読性が良いコードの形式に変換したものです。minifyすることで読み込み速度が向上します。

### admin.scssの設定

```ruby
@import 'admin-lte/plugins/fontawesome-free/css/all.min.css';
@import 'admin-lte/dist/css/adminlte.min.css';
```

### application.jsの設定を変更

```js
//= require jquery3
//= require popper
//= require bootstrap-sprockets

//= require rails-ujs
//= require activestorage
```

不要なファイルを読み込むのを避けるために、`//= require_tree .` は使わず、必要なファイルを適切な順番で個別に記述する。

今回、`app/assets/javascripts/application.js`と同じ階層に、`app/assets/javascripts/admin.js`を作っているため、`app/assets/javascripts/application.js`で`//= require_tree .`を記述すると、`app/assets/javascripts/application.js`で`app/assets/javascripts/admin.js`を読み込んでしまう。

## マニフェストファイルの読み込み設定

config/initializers/assets.rb
```ruby
# Be sure to restart your server when you modify this file.

# Version of your assets, change this if you want to expire all your assets.
Rails.application.config.assets.version = '1.0'

# Add additional assets to the asset load path.
# Rails.application.config.assets.paths << Emoji.images_path
# Add Yarn node_modules folder to the asset load path.
Rails.application.config.assets.paths << Rails.root.join('node_modules')

# Precompile additional assets.
# application.js, application.css, and all non-JS/CSS in the app/assets
# folder are already added.
Rails.application.config.assets.precompile += %w( admin.js admin.css )
```

プリコンパイルの設定をします。application以外のマニフェストファイルを個別に読み込む場合は、プリコンパイルの設定をしないと、そのファイルは対象外とされてしまうためエラーが起きてしまう。

`Rails.application.config.assets.precompile += %w( admin.js admin.css )` のコメントアウトを外してプリコンパイルの設定をします。

## usersテーブルに管理者権限を判定する為のroleカラムを追加

```zsh
$ rails g migration add_role_to_users
```

マイグレーションファイル
```ruby
class AddRoleToUsers < ActiveRecord::Migration[5.2]
  def change
    add_column :users, :role, :integer, default: 0, null: false
  end
end
```

一般ユーザと管理者を区別し、整数の`integer`型で`role` というカラムを追加しています。
一般ユーザが`0`、管理者は`1`とします。デフォルトは一般ユーザである`0`とします。

```ruby
bin/rails db:migrate
```

### enumを設定

user.rb
```ruby
class User < ApplicationRecord
	enum role: { general: 0, admin: 1 }
end
```

enumとは、モデルの数値カラムに対して文字列による名前を付けれるようになる仕組み。
今回は一般ユーザを`general`、管理者を`admin`として定義。

## 管理者用のコントローラを作成

```bash
$ rails g controller Admin::Base
```

管理者用のコントローラを作成するにあたって、「管理系」のカテゴリとして区分したいため、`Admin::BaseController` という名前を付けることにします。
この名前は、`Admin`というモジュールの名前空間の中に`BaseController`というクラスを定義するという意味。

## 管理画面用のトップページに遷移するコントローラーを作成

```bash
$ rails g controller Admin::Dashboards index
```

## 管理者のログイン機能を担うコントローラを作成

```bash
$ rails g controller Admin::User_sessions new
```

## ルーティングの設定

```ruby
namespace :admin do
  root to: 'dashboards#index'
  get 'login', to: 'user_sessions#new'
  post 'login', to: 'user_sessions#create'
  delete 'logout', to: 'user_sessions#destroy'
end
```

Adminというモジュールの名前空間を使用したコントローラを作成したので、それに対応するようにルーティングにも名前空間を設定

## ビューファイルの設定

管理者用画面は一般用の画面とは見た目も異なるので、専用のテンプレートファイルを用意

### 管理者用画面のレイアウトファイルの作成
`starter.html`から流用してレイアウトファイルを作成
ヘッダー、メニュー、フッター部分は別の部分テンプレートとして切り分けて`views/admin/shared` に配置

views/admin/layouts/application.html.erb
```ruby
<!DOCTYPE html>

<html>
  <head>
    <meta charset="utf-8">
    <meta lang='ja'>
    <meta name="robots" content="noindex, nofollow">　                # SEOに関する記述
    <title><%= page_title(yield(:title), admin: true) %></title>      # 後半で使う設定
    <%= csrf_meta_tags %>　　# クロスサイトフォージェリ対策
    <%= stylesheet_link_tag 'admin', media: 'all' %>   # ここでadmin.scssを読み込む
    <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Source+Sans+Pro:300,400,400i,700&display=fallback">
  </head>

  <body class="hold-transition sidebar-mini">
  <div class="wrapper">

    <%= render 'admin/shared/header' %>   # headerをレンダーする
    <%= render 'admin/shared/sidebar' %>  # sidebarをレンダーする
 
    <!-- Content Wrapper. Contains page content -->
    <div class="content-wrapper">
     <%= render 'shared/flash_message' %>  # フラッシュメッセージを呼び込む
     <%= yield %>　　　　　　　　　　　　　　# bodyを読み込む
    </div>
    <!-- /.content-wrapper -->

    <%= render 'admin/shared/footer' %>   # footerをレンダーする

  </div>
  <%= javascript_include_tag 'admin' %>   # ここでadmin.jsを読み込む
  </body>
</html>
```

### ヘッダー、メニュー、フッターは部分テンプレートとしてadmin/shared 配下に作成

ヘッダー
```ruby
<!-- Navbar -->
  <nav class="main-header navbar navbar-expand navbar-white navbar-light">
    <!-- Left navbar links -->
    <ul class="navbar-nav">
      <li class="nav-item">
        <a class="nav-link" data-widget="pushmenu" href="#"><i class="fas fa-bars"></i></a>
      </li>
    </ul>

    <!-- Right navbar links -->
    <ul class="navbar-nav ml-auto">
      <!-- Navbar Search -->
      <li class="nav-item">
        <%= link_to t('defaults.logout'), admin_logout_path, method: :delete, class: 'nav-link' %>
      </li>
    </ul>
  </nav>
<!-- /.navbar -->
```

サイドバー

```ruby
<!-- Main Sidebar Container -->
<aside class="main-sidebar sidebar-dark-primary elevation-4">
  <!-- Brand Logo -->
  <a href="index3.html" class="brand-link">
    <%= image_tag 'AdminLTELogo.png', class: 'brand-image img-circle elevation-3' %>
    <span class="brand-text font-weight-light">AdminLTE 3</span>
  </a>

  <!-- Sidebar -->
  <div class="sidebar">
    <!-- Sidebar user panel (optional) -->
    <div class="user-panel mt-3 pb-3 mb-3 d-flex">
      <div class="image">
        <%= image_tag current_user.avatar_url, class: 'img-circle elevation-2' %>
      </div>
      <div class="info">
        <a href="#" class="d-block"><%= current_user.decorate.full_name %></a>
      </div>
    </div>

    <!-- Sidebar Menu -->
    <nav class="mt-2">
      <ul class="nav nav-pills nav-sidebar flex-column" data-widget="treeview" role="menu" data-accordion="false">
        <li class="nav-item">
          <%= link_to '#', class: "nav-link" do %>
            <i class="nav-icon far fa-file"></i>
            <p>
              掲示板一覧
            </p>
          <% end %>

        <li class="nav-item">
          <%= link_to '#', class: "nav-link" do %>
            <i class="nav-icon far fa-user"></i>
            <p>
              ユーザ一一覧
            </p>
          <% end %>
        </li>
      </ul>
    </nav>
    <!-- /.sidebar-menu -->
  </div>
  <!-- /.sidebar -->
</aside>
```

フッター
```ruby
<!-- Main Footer -->
<footer class="main-footer">
  <strong>Copyright © 2019 RUNTEQ.</strong>
  All rights reserved. 
</footer>
```

### 管理者画面用トップページのビューファイル

```ruby
<% content_for(:title, t('.title')) %>
<div class="content-wrapper">
  <div class="row">
    <p>ダッシュボードです</p>
  </div>
</div>
```

### Admin::Baseコントローラ にlayout宣言を追記する

admin/base_controller
```ruby
class Admin::BaseController < ApplicationController
  layout 'admin/layouts/application'    # layout宣言
end
```

通常、何も指定が無ければコントローラは`app/views/layouts/application.html.erb`をレイアウトとして探索する。
管理者機能の根幹クラスである`Admin::BaseController` で個別の管理者用画面のレイアウトファイルを呼び込むようにlayout宣言する。
https://railsguides.jp/layouts_and_rendering.html#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E7%94%A8%E3%81%AE%E3%83%AC%E3%82%A4%E3%82%A2%E3%82%A6%E3%83%88%E3%82%92%E6%8C%87%E5%AE%9A%E3%81%99%E3%82%8B

### Admin::Dashboardsコントローラ を修正

```ruby
class Admin::DashboardsController < Admin::BaseControlller
  def index; end
end
```
管理者画面トップページへ遷移させるためのコントローラは、管理者機能の根幹となる`Admin::BaseControlller` を継承する設計へと変更

## 管理画面用のページタイトルの設定

```ruby
module ApplicationHelper
  def page_title(page_title = '', admin = false)
    base_title = if admin
                 'SAMPLE BOARD APP(管理画面)'
                 else
                 'SAMPLE BOARD APP'
                 end

    page_title.empty? ? base_title : page_title + ' | ' + base_title
  end
end
```

「ダッシュボード | 固定タイトル(管理画面)」のように、語尾に`(管理画面)`の文字が表示されるような機能を追加
管理者か否かの判定ロジックを組み、
デフォルトを `admin = false` とすることによって、既存の一般ファイルに影響が出ないようにしている。

### 管理者用レイアウトファイルでタイトル設定

今一度、管理者用のレイアウトファイル(`views/admin/layouts/application.html.erb`)を見る。

```ruby
<title><%= page_title(yield(:title), admin: true) %></title>
```
管理者用レイアウトファイルで`admin: true` を設定しておくことで、すべての管理者画面は`admin: true`が付与される仕組みとなっている

### 各管理者用ビューファイル

```ruby
<% content_for(:title, t('.title')) %>     # titleはローケルファイルから読み込み
```

## 管理者ログイン機能を作成

-   ログイン画面で読み込むレイアウトファイルは`admin_login.html.erb`とし、`views/layouts`配下に作成
-   管理者ログインページから一般ユーザーでログインした場合は`root_path`にリダイレクトされるよう設定
-   管理者がログイン後は管理画面のトップページにリダイレクトされるように設定
-   管理者がログインに失敗したら、管理者ログインページへリダイレクトされる
-   ログアウトしたらログインページへ

### Admin::Baseコントローラの設定

```ruby
class Admin::BaseController < ApplicationController
  bofore_action :check_admin
  layout 'admin/layouts/application'


  private



  def not_authenticated
    redirect_to admin_login_path, warning: t('defaults.message.require_login')
  end

  def check_admin
    redirect_to root_path, warning: t('defaults.message.not_authorized') unless current_user.admin?
  end
end
```

今回、このコントローラ内の処理ではログインしていないユーザを管理者用のログインページへリダイレクトさせたいので、`not_authenticated`メソッドの中身を`redirect_to_ admin_login_path`に書き換えています。
`check_admin` メソッドについてですが、`unless current_user.admin?` の部分で現在ログインしているユーザが管理者かどうかを判定しています。
`admin?` メソッドはというと、enumを設定した際に`role`カラムの`1`を`admin`としたので使えるようになっているメソッドです。
`before_action :check_admin` とすることで、各アクションより先に実行されます。

### Admin::UserSessionsコントローラの設定

```ruby
class Admin::UserSessionsController < Admin::BaseController     # #BaseControllerを継承する
  skip_before_action :require_login, only: %i[new create]
  skip_before_action :check_admin, only: %i[new create]
  layout 'admin/layouts/admin_login'        # ログインページ用のレイアウトを用意するので宣言

  def new; end

  def create
    @user = login(params[:email], params[:password]) # Sorceryメソッド　　emailとpasswordでログイン認証する
　  if @user
      redirect_to admin_root_path  , success: 'ログインしました'
    else
      flash.now[:danger] = 'ログインに失敗しました'
      render :new
    end
  end

  def destroy
    logout                                      # ログアウトするためのSorceryメソッド
    redirect_to admin_login_path, success: 'ログアウトしました'
  end
end
```

`Admin::Base` コントローラでは`admin/layouts/application`をレイアウトとして使用するようlayout宣言していました。
`Admin::UserSessions`コントローラも`Admin::Base` コントローラを継承してますので、そのままでは`admin/layouts/application`をレイアウトとして使用するでしょう。
しかしログイン画面は、またもや異なる見た目にするので、別のログインページ用のレイアウトファイルを用意してlayout宣言しています。
また、ログインしていないユーザや、管理者権限のないユーザでも、ログイン画面が表示されるように、`skip_before_action`でフィルタをスキップさせてあげましょう。
ログインした後は、`admin_root_path` へリダイレクトされ、`admin/dashboards#index` アクションが呼び出されます。
このとき、管理者ではない一般ユーザでログインした場合は、`before_action :check_admin`フィルタによって、一般ユーザ用のトップページへリダイレクトされます。

### 管理者ログイン画面のビューファイルを作成

`views/layouts/` 配下に`admin_login.html.erb`という名前で作成

```ruby
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title><%= page_title(yield(:title), admin: true) %></title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag 'admin', media: 'all' %>        # admin.scssを読み込む
  </head>
  <body class="hold-transition login-page">
    <div>
      <%= render 'shared/flash_message' %>
      <%= yield %>
    </div>
  </body>
</html>
```

views/admin/user_sessions/new.html.erb
```ruby
<% content_for(:title, t('.title')) %>
<div class="login-box">
  <div class="login-logo">
    <h1><%= t('.title') %></h1>
  </div>
  <!-- /.login-logo -->
  <div class="card">
    <div class="card-body login-card-body">

      <%= form_with url: admin_login_path, locale: true do |f| %>
        <%= f.label :email, User.human_attribute_name(:email) %>
        <div class="input-group mb-3">
          <%= f.email_field :email, class: 'form-control', placeholder: 'Email'%>
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-envelope"></span>
            </div>
          </div>
        </div>

        <%= f.label :password, User.human_attribute_name(:password) %>
        <div class="input-group mb-3">
          <%= f.password_field :password, class: 'form-control', placeholder: :password %>
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-lock"></span>
            </div>
          </div>
        </div>

        <div class="row">
          <div class="col-12">
            <%= f.submit t('defaults.login'), class: 'btn btn-block btn-primary' %>
          </div>
        </div>
      <% end %>
    </div>
  </div>
</div>
```

### ロケールファイル追記

```ruby
admin:
  user_sessions:
    new:
      title: 'ログイン'
    create:
      success: 'ログインしました'
　    fall: 'ログインに失敗しました'
    destroy:
      success:  'ログアウトしました'
  dashboards:
    index:
      title: 'ダッシュボード'
```

----

>参照
>https://osamudaira.com/257/
>https://qiita.com/mmaumtjgj/items/9ee5da80202c3bc3be75
>https://study-diary.hatenadiary.jp/entry/2020/09/02/170755