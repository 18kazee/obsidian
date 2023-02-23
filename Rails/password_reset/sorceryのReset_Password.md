
- sorceryのreset_psswordモジュールのインストール
- メイラーの生成
- コントローラーの作成
- トークンにユニーク制約を付与
- ビューの作成
- letter_opener_webの追加
- configの追加

## sorceryのreset_psswordモジュールのインストール

```shell
$ bin/rails g sorcery:install reset_password --only-submodules
```

生成されたファイル
```bash
> rails g sorcery:install reset_password --only-submodules          
Running via Spring preloader in process 71125
        gsub  config/initializers/sorcery.rb
      insert  app/models/user.rb
      create  db/migrate/20211009040003_sorcery_reset_password.rb
```

記載確認
```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:reset_password]
```

```ruby
# マイグレーションファイル
class SorceryResetPassword < ActiveRecord::Migration[5.2]
  def change
    add_column :users, :reset_password_token, :string, default: nil
    add_column :users, :reset_password_token_expires_at, :datetime, default: nil
    add_column :users, :reset_password_email_sent_at, :datetime, default: nil
    add_column :users, :access_count_to_reset_password_page, :integer, default: 0

    add_index :users, :reset_password_token
  end
end
```

```shell
$ bin/rails db:migrate
```

## メイラーの作成

```shell
$ bin/rails g mailer UserMailer reset_password_email
```

生成されたファイル
```bash
> rails g mailer UserMailer reset_password_email                                                           
Running via Spring preloader in process 71280
      create  app/mailers/user_mailer.rb
      invoke  erb
      create    app/views/user_mailer
      create    app/views/user_mailer/reset_password_email.text.erb
      create    app/views/user_mailer/reset_password_email.html.erb
```

user_mailerの下の二つのビューは実際にユーザーに対して送るメールの中身です。
text形式のメールとhtml形式のメールで両方記載します。

### パスワードリセットに使用するメイラーの記載

`config/initializers/sorcery.rb` にreset_passwordサブモジュールを追加し、
パスワードリセットに使用するActionMailerとしてUserMailerを定義する記述をする

```ruby
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:reset_password, blabla, blablu, ...]

Rails.application.config.sorcery.configure do |config|
  config.user_config do |user|
    user.reset_password_mailer = UserMailer
  end
end
```

必要な記述がコメントアウトされている中で既に揃っているので、コメントアウトを外して使用する。

### メイラービューの作成

```ruby
# app/views/user_mailer/reset_password_email.text.erb
<%= @user.decorate.full_name %>様
===========================================

パスワード再発行のご依頼を受け付けました。
こちらのリンクからパスワードの再発行を行ってください。
<%= @url %>
```

```ruby
# app/views/user_mailer/password_reset_email.html.erb
<%= @user.decorate.full_name %>様
<p>===========================================</p>
<p>パスワード再発行のご依頼を受け付けました。</p>

<p>以下のリンクからパスワードの再発行を行ってください。</p>

<p><a href="<%= @url %>"><%= @url %></a></p>
```

上記の`@url` の中身は、tokenで発行された再登録用のurlが入っている。

### メールを送信するメソッドを記述

```ruby
# app/mailers/user_mailer.erb
class UserMailer < ApplicationMailer
  default from: 'from@example.com'
  def reset_password_email
    @user = User.find(user.id)
    @url = edit_password_reset_url(@user.reset_password_token)
    mail(to: user.email, subject: 'パスワードリセット')
  end
end
```

-   `default from: 'from@example.com'`でメールの送信元のアドレスを指定
-   `mail(to: user.email,subject: 'パスワードリセット')`でメールの宛先、件名を指定

`@user`、`@url` は先ほど作成したメイラービューの中で使われています。

## コントローラーの作成

```shell
$ bin/rails g contoroller PasswordResets new create edit update
```

```bash
> rails g controller PasswordResets new create edit update                                                   
Running via Spring preloader in process 71542
      create  app/controllers/password_resets_controller.rb
      invoke  erb
      create    app/views/password_resets
      create    app/views/password_resets/new.html.erb
      create    app/views/password_resets/create.html.erb
      create    app/views/password_resets/edit.html.erb
      create    app/views/password_resets/update.html.erb
      invoke  decorator
      create    app/decorators/password_reset_decorator.rb
```

### 生成されたPasswordResetsコントローラーを記述

[公式wiki](https://github.com/Sorcery/sorcery/wiki/Reset-password)を参考にし、記述

```ruby
class PasswordResetsController < ApplicationController
	skip_before_action :require_login

	def new; end

	def create
		@user = User.find_by(email: params[:email])
		@user&.deliver_reset_password_instructions!
		resirect_to login_path success: t('.success')
	end

	def edit
		@token = params[:id]
		@user = User.load_from_reset_password_token(params[:id])
		return not_authenticated if @user.blank?
	end

	def update
		@token = params[:id]
		@user = User.load_from_reset_password_token(params[:id])
		return not_authenticated if @user.blank?
		
		@user.password_confirmation = params[:user][:password_confirmation]

		if @user.chenge_password(params[:user][:password])
			redirect_to login_path success: t('.success')
		else
			flash.now[:danger] = t('.fail')
			render 'edit'
		end
	end
end
```
>参照
([[password_resets_controller]])

### password_resetsコントローラーのルーティングの設定

```ruby
resorces :password_resets, only: %i[new create edit update]
```

## トークンにユニーク制約を付与

Userモデルにバリデーションを記載

```ruby
validates :password_reset_token, presence: true, uniqueness: true, allow_nil: true
```

tokenは一意なものでなければいけませんので、`uniqueness: true` を付与。
しかし、パスワードを変更した際、`reset_password_token` はnilになるのでユニーク制約に引っかかってしまいます。nilを許可する必要があるでしょう。
そこで`allow_nil` オプションを使います。`allow_nil: true` を付与することで、対象の値がnilの場合にバリデーションをスキップします。

## ビューファイルの作成

### パスワードリセット申請画面

```ruby
# app/views/password_resets/new.html
<%= content_for(title: t('.title')) %>
<div class="container">
	<div class="row">
		<div class="col-md-10 offset-md-1 col-lg-8 offset-lg-2">
			<h1><%= t('.title') %></h1>
			<%= form_with url: password_resets_path, local:true do |f| %>
				<div class="form-group">
					<%= f.label :email, User.human_attribute_name(:email) %><br />
					<%= f.email_field :email, class: 'form-control' %>
				</div>
				<p class="text-center">
					<%= f.submit t('password_ressets.new.submit'), class:btn btn-primary %>
				</p>
			<% end %>
		</div>
	</div>
</div>
```

こちらの`form_with` はモデルと紐付いていない点に注意。
また、そのことよりラベルが日本語化しませんので、自分で日本語化する記述を追加しましょう。

### パスワードリセット画面

```ruby
# app/views/password_resets/edit.html
<%= content_for(title: t('.title')) %>
<div class="container">
	<div class="row">
		<div class="col-md-10 offset-md-1 col-lg-8 offset-lg-2">
			<h1><%= t('.title') %>
			<%= form_with model: @user, url: password_reset_path(@token), local: true do |f| %>
				<%= render 'shared/error_messages', object: f.object %>

				<div class="form-group">
					<%= f.label :email %>
					<%= @user.email%>
				</div>
				<div class="form-group">
					<%= f.label :password %>
					<%= f.password_field :password, class: 'form-control' %>
				</div>
				<div class="form-group">
					<%= f.label :password_confirmation %>
					<%= f.password_field :password_confirmation, class: 'form-control' %>
				</div>
				<div class="action">
					<p class="text-center">
						<%= f.submit class: btn btn-primary %>
					</p>
				</div>
			<% end %>
		</div>
	</div>
</div>
```

### パスワードリセット申請画面リンクの表示

ログイン画面にパスワードリセット申請画面へのリンクを追加

```ruby
# app/views/user_sessions/new.html.erb
      <% end %>
      <div class='text-center'>
        <%= link_to (t '.to_register_page'), new_user_path %>
        <%= link_to t('.password_forget'), new_password_reset_path %>
      </div>
    </div>
  </div>
```