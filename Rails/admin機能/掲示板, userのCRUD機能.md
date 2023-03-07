
- コントローラー作成
- ルーティングの作成
- enum_helpの導入
- Viewの作成
- 検索機能の作成
- 掲示板のメニューをアクティブにする

## コントローラーの作成

```shell
$ bin/rails g controller admin::users
```

```shell
$ bin/rails g controller admin::boards
```

コントローラーは普通の[CRUD](http://d.hatena.ne.jp/keyword/CRUD)機能ですので、一般での時と同じように作成します。

### コントローラーの記述

admin/users_controller
```ruby
class Adminn::UsersController < Admin::BaseController
	before_action :set_user, only: %i[show edit update destroy]

	def index
		@q = User.ransack(params[:q])
		@users = @q.result(distinct: true).order(created_at: :desc).page(params[:page])
	end

	def show; end

	def edit; end

	def update
		if @user.update(user_params)
			redirect_to admin_user_path(@user), success: 'ユーザーを更新しました'
		else
			flash.now[:danger] = '更新できませんでした'
			render :edit
		end
	end

	def destroy
		@user.destroy!
		redirect_to admin_users_path, success: 'ユーザーを削除しました'
	end

	private

	def set_user
		@user = User.find(params[:id])
	end

	def user_params
		params.require(:user).permit(:email, :first_name, :last_name, :avatar, :avatar_cache)
	end
end
```

admin/boards_controller
```ruby
class Admin::BoardsController < Admin::BaseController
	before_action :set_board, only: %i[show edit update destroy]

	def index
		@q = Board.ransack(params[:q])
		@boards = @q.result(distinct: true).includes(:user).order(created_at: :desc).page(params[:page])
	end

	def show; end

	def edit; end

	def update
		if @board.update
			redirect_to admin_board_path(@board), sucess: '掲示板を更新しました'
		else
			flash.now[:danger] = '更新できませんでした'
		end
	end

	def destroy
		@board.destroy!
		redirect_to admin_boards_path, success: '掲示板を削除しました'
	end

	private

	def set_board
		@board = Board.find(params[id])
	end

	def board_params
		params.require(:board).permit(:title, :body, :board_image, :board_image_cache)
	end
end
```

## ルーティングの作成

config/routes.rb
```ruby
name_space :admin do
	root 'dashboards#index', as: :root
	get 'login', to: 'user_sessions#new'
	post 'login', to: 'user_sessions#create'
	delete 'logout', to: 'user_sessions#destroy'
	
	resources :users, only: %i[index show edit update destroy] 追加
	resources :boards, only: %i[index show edit update destroy] 追加
end
```

## Viewの作成

### 検索機能の作成(管理者検索)
admin/users/index.html.erb (検索部分のみ）
```ruby
<%# 検索機能 %>
<%= search_form_for @search, url: admin_users_path do |f| %>
	<div class="row">
		<div class="form-inline align-items-center mx-auto">
			<div class="col-auto">
				<%= f.search_field :first_name_or_last_name_cont, class: 'form-control',　placeholder: '検索ワード' %>
			</div>
			<div class="col-auto">
				<%= f.select :role_eq, User.roles_i18n.invert.map　
						{ |key, value| [key, User.roles[value]]}, 
						{ include_blank: '指定なし' }, { class: 'form-control' } %>
			</div>
			<div class="col-auto">
				<%= f.submit value: '検索', class: 'btn btn-primary' %>
			</div>
		</div>
	</div>
<% end %>
```

[[adminをセレクトボックスで検索]]

### 検索機能の作成(日付検索)
admin/boards/index.html.erb(検索機能のみ)
```ruby
<%# 検索機能 %>
<%= search_form_for @search, url: admin_boards_path do |f| %>
	<div class="row">
		<div class="form-inline align-items-center mx-auto">
			<div class="col-auto">
				<%= f.search_field :title_or_body_cont, class: 'form-control', placeholder: '検索ワード' %>
			</div>
			<div class="col-auto">
				<%= f.date_field :created_at_gteq, class: 'form-control' %>
				<span>〜</span>
				<%= f.date_field :created_at_lteq_end_of_day, class: 'form-control' %>
			</div>
			<div class="col-auto">
				<%= f.submit value: '検索', class: 'btn btn-primary' %>
			</div>
		</div>
	</div>
<% end %>
```

config/initializers/ransack.rb
```ruby
	Ransack.configure do |config|
		config.add_predicate 'lteq_end_of_day',
		arel_predicate: 'lteq', # カスタマイズしたいpredicateを指定
		formatter: proc { |v| v.end_of_day } # どのようにフォーマットするか指定
	end
```

[[日付による検索機能]]

## 掲示板のメニューをアクティブにする

app/helpers/application_helper.rb
```ruby
def active?(controller_name)
	return 'active' if controller_name == params[:controller]
end
```

admin/shared/_sidebar.html.erb
```ruby
<%= link_to admin_boards_path, class: "nav-link #{ active?("admin/boards") }" do %>
```

```ruby
<%= link_to admin_users_path, class: "nav_link #{ active?("admin/users") }" de %>
```

[[admin機能のサイドバーをアクティブに]]

