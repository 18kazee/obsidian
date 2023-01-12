
### コントローラーの作成

```ruby
# ターミナル
$ bin/rails generate controller boards
```

コントローラ、view、decoratorが作成される。


### 掲示板の一覧をDBから取得する処理を追加。

```ruby
#app/controllers/boards_controller

class BoardController < ApplicationController 
	def index 
		@boards = Board.all.includes(:user).order(created_at: :desc)	
	end
end
```

```ruby
# index.html.erb
<%= render @boards %> 
```

>参照
> [[includes]]メソッド
> [[部分テンプレートの省略]]


### 掲示板作成のrouteを追加

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root 'static_pages#top'

  get 'login', to: 'user_sessions#new'
  post 'login', to: 'user_sessions#create'
  delete 'logout', to: 'user_sessions#destroy'

  resources :users, only: %i[new create]
  resources :boards, only: %i[index new create] #この部分を追加
end
```
