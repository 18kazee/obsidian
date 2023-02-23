
```ruby
class PasswordResetsController < ApplicationController
  # Rails 5以降では、次の場合にエラーが発生します。
  # before_action :require_login
  # ApplicationControllerで宣言されていません。
  skip_before_action :require_login
  
  # パスワードリセット申請画面へレンダリングするアクション
  def new; end
    
  # パスワードのリセットを要求するアクション。
  # ユーザーがパスワードのリセットフォームにメールアドレスを入力して送信すると、このアクションが実行される。
  def create 
    @user = User.find_by(email: params[:email])
        
    # この行は、パスワード（ランダムトークンを含むURL）をリセットする方法を説明した電子メールをユーザーに送信します
    @user&.deliver_reset_password_instructions!
    # 上記は@user.deliver_reset_password_instructions! if @user と同じ
        
    # 電子メールが見つかったかどうかに関係なく、ユーザーの指示が送信されたことをユーザーに伝えます。
    # これは、システムに存在する電子メールに関する情報を攻撃者に漏らさないためです。
    redirect_to login_path, success: t('.success')
  end
    
  # パスワードのリセットフォーム画面へ遷移するアクション
  def edit
    @token = params[:id]
    @user = User.load_from_reset_password_token(params[:id])
　　return not_authenticated if @user.blank?
  end
      
  # ユーザーがパスワードのリセットフォームを送信したときに発生
  def update
    @token = params[:id]
    @user = User.load_from_reset_password_token(params[:id])
    return not_authenticated if @user.blank?

    # 次の行は、パスワード確認の検証を機能させます
    @user.password_confirmation = params[:user][:password_confirmation]
    # 次の行は一時トークンをクリアし、パスワードを更新します
    if @user.change_password(params[:user][:password])
      redirect_to login_path, success: t('.success')
    else
      flash.now[:danger] = t('.fail')
      render :edit
    end
  end
end
```


## deliver_reset_password_instructions!メソッド

有効期限付きのリセットコード（トークン）を生成し、ユーザーに電子メールを送信します。

```ruby
# Generates a reset code with expiration and sends an email to the user.
def deliver_reset_password_instructions!
  mail = false
  config = sorcery_config
  # hammering protection
  return false if config.reset_password_time_between_emails.present? && send(config.reset_password_email_sent_at_attribute_name) && send(config.reset_password_email_sent_at_attribute_name) > config.reset_password_time_between_emails.seconds.ago.utc

  self.class.sorcery_adapter.transaction do
    generate_reset_password_token!
    mail = send_reset_password_email! unless config.reset_password_mailer_disabled
  end
  mail
end
```

>参照
>https://github.com/Sorcery/sorcery/blob/6fdc703416b3ff8d05708b05d5a8228ab39032a5/lib/sorcery/model/submodules/reset_password.rb#L98

`@user&.deliver_reset_password_instructions!` ですが、`&.`はメソッドの実行対象のオブジェクトが`nil`(ユーザーがいなかったとき)だった場合を考慮した記法です。

`@user = User.find_by(email: params[:email])`でユーザーが見つからなかったときは`@user`には`nil`が入ります。
その`nil`に対して`deliver_reset_password_instructions!`メソッドを実行すると、`UndefinedMethod`の例外が発生します。

そこで`&.`を使用すれば、もし`nil`であっても`nil`を返すだけになり、例外は発生しません。

## load_from_reset_password_tokenメソッド

トークンでユーザーを検索し、有効期限もチェックします。 トークンが見つかり、有効な場合、ユーザーを返します。

```ruby
# Find user by token, also checks for expiration.
# Returns the user if token found and is valid.
def load_from_reset_password_token(token, &block)
  load_from_token(
    token,
    @sorcery_config.reset_password_token_attribute_name,
    @sorcery_config.reset_password_token_expires_at_attribute_name,
    &block
  )
end
```

>参照
>https://github.com/Sorcery/sorcery/blob/6fdc703416b3ff8d05708b05d5a8228ab39032a5/lib/sorcery/model/submodules/reset_password.rb#L62

`User.load_from_reset_password_token(params[:id])`にて、有効期限切れなどでユーザーが取得できなかったとき、not_authenticatedとなります。

## change_passwordメソッド

トークンをクリアし、ユーザーの新しいパスワードを更新しようとします。

```ruby
# Clears token and tries to update the new password for the user.
def change_password(new_password, raise_on_failure: false)
  clear_reset_password_token
  send(:"#{sorcery_config.password_attribute_name}=", new_password)
  sorcery_adapter.save raise_on_failure: raise_on_failure
end
```

>参照
>https://github.com/Sorcery/sorcery/blob/6fdc703416b3ff8d05708b05d5a8228ab39032a5/lib/sorcery/model/submodules/reset_password.rb#L126

