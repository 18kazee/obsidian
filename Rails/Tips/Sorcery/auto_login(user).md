その名の通りオートログイン。
メールアドレスやパスワードを使わずuserとしてログインする。

### ユーザー登録後の自動ログイン

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    if @user.save
      auto_login(@user)
      redirect_to root_url, notice: 'ユーザーを登録しました'
    else
      flash.now[:alert] = 'ユーザーが登録できませんでした'
      render :new
    end
  end
end
```
### ゲストユーザーログイン

```ruby
# app/controllers/user_sessions_controller.rb
class UserSessionsController < ApplicationController
  def guest_login
    guest_user = User.find_by!(role: 'guest')
    auto_login(guest_user)
    redirect_to root_path, notice: 'ログインしました'
  end
end
```
