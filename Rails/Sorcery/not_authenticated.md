`require_login`内で実行される。  
デフォルトでは`redirect_to root_path`（自動的にルートに飛ばされる）と定義されているが、カスタマイズしたい場合は`application_controller`で上書きをする。

```ruby
# application_controller.rb
class ApplicationController < ActionController::Base 
  protected 

  def not_authenticated
    redirect_to login_url, alert: 'ログインしてください' 
  end 
end
```
