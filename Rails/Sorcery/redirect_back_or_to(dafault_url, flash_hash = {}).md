### require_loginのときにセッションに格納されたURLまたはデフォルトのURLにリダイレクトする。

ログインのアクションで使う。
flash_hashはデフォルトでsuccess, dangerにも対応しています。

```ruby
# user_sessions_controller.rb
class UserSessionsController < ApplicationController
  def create
    @user = login(params[:email], params[:password])
    if @user
      redirect_back_or_to root_url, success: 'ログインしました'
    else
      flash.now[:danger] = 'ログインに失敗しました'
      render :new
    end
  end
end
```