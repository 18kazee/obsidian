
- モデルの作成
- コントローラーの作成
- ルーティンの設定
- Viewの設定

## モデルの作成

```shell
$ bin/rails g model like user:references post:references
```

### アソシエーション設定とバリデーション

```ruby
# post.rb
belongs :user
has_many :likes, dependent: :destroy
```

```ruby
#user.rb
has_many :post, dependent: :destroy
has_many :likes, dependent: :destroy
has_many :like_posts, through: :likes, source: :post
```

```ruby
#like.rb
belongs_to :post
belongs_to :user

validates :user_id, uniqueness: { scope: :post_id }
```

### マイグレーションファイル

```ruby
# マイグレーションファイル 
class CreateLikess < ActiveRecord::Migration[5.2] 
	def change
		create_table :Likes do |t| 
			t.references :user, foreign_key: true 
			t.references :post, foreign_key: true 
			t.timestamps 
			t.index [:user_id, :post_id], unique: true　#追記する
		end 
	end
end
```

```shell
$ bin/rails db:migrate
```

## コントローラーの作成

```shell
$ bin/rails g controller likes
```

### userモデルにlikesコントローラーの内容を記載

```ruby
#user.rb
has_many :post, dependent: :destroy
has_many :likes, dependent: :destroy
has_many :liked_posts, through: :likes, source: :post

def mine?(object)
	# 呼び出し元のオブジェクトのIDを示す self.id を省略した記法。
	# @user.mine?(object)のように利用すると、object.user_id と @user.id を比較する。
	object.user_id == id
end

def like(post)
	liked_posts << post
end

def unlike(post)
	liked_posts.destroy(post)
end

def like?(post)
	liked_posts.include?(post)
end
```

### likesコントローラー

```ruby
# likes_controller
def create
	post = Post.find(params[:post_id])
	current_user.like(post)
	redirect_back fall_location: posts_path, success: 'liked post.'
end

def destroy
	post = current_user.liked_posts.find(params[:post_id])
	current_user.unlikes(post)
	redirect_back fall_lacation: posts_path, success: 'unliked post.'
end
```

### postsコントローラー

```ruby
def likes
	@liked_posts = current_user.liked_posts
end
```
likeした掲示板一覧を表示するのに必要

## ルーティングの設定

```ruby
Rails.application.routes.draw do
	get 'login', to: 'user_sessions#new'
	post 'login', to: 'user_sessions#create'
	delete 'logout', to: 'user_sessions#destroy'
	
	resources :posts do
		resources :likes, only: %i[create destroy]
		collection do
			get :likes
		end
	end
	resources :users, only: %i[new create]
end
```

## Viewの設定

### posts/index.html.erb

```ruby
<h1>Posts</h1>
<table>
	<thead>
		<tr>
			<th>Title</th>
			<th>Content</th>
			<th colspan="3"></th>
		</tr>
	</thead>
	<tbody>
		<% @posts.each do |post| %>
			<tr>
				<td><%= post.title %></td>
				<td><%= post.content %></td>
				<td><%= link_to 'Show', post %></td>
				<% if current_user.mine?(post) %>
					<td><%= link_to 'Edit', edit_post_path(post) %></td>
					<td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
				<% else %>
					<td><%= render 'posts/like_button', post: post %>
				<% end %>
			</tr>
		<% end %>
	</tbody>
</table>

<br>

<%= link_to 'New Post', new_post_path %>
```

### posts/_like_button.html.erb

```ruby
<% if current_user.like?(post) %>
	<%= link_to 'UnLike', post_like_path(post_id: post.id, id: current_user.likes.find_by(params[:id])), method: :delete %>
<% else %>
	<%= link_to 'Like', post_likes_path(post_id: post.id), method: :post %>
<% end %>
```

### post/likes.html.erb

```ruby
<h1>Liked Posts</h1>
<table>
	<thead>
		<tr>
			<th>Title</th>
			<th>Content</th>
			<th colspan="3"></th>
		</tr>
	</thead>
	<tbody>
		<% @liked_posts.each do |liked_post| %>
			<tr>
				<td><%= post.title %></td>
				<td><%= post.content %></td>
				<td><%= link_to 'Show', post %></td>
			</tr>
		<% end %>
	</tbody>
</table>

<br>

<%= link_to 'New Post', new_post_path %>
```

### shared/_header.html.erb

```ruby
<header>
	<nav class="navbar navbar-expand-lg navigation navbar-light bg-light">
		<button class="navbar-toggler" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
			<span class="navbar-toggler-icon"></span>
		</button>
		<div class="collapse navbar-collapse" id="navbarSupportedContent">
			<ul class="navbar-nav ml-auto main-nav align-items-center">
				<li class="nav-item dropdown dropdown-slide">
					<a href="#" class="nav-link dropdown-toggle" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
						Post
					</a>
					<div class="dropdown-menu dropdown-menu-right">
						<%= link_to 'List of Posts', posts_path, class: 'dropdown-item' %>
<%= link_to 'New Post', new_post_path, class: 'dropdown-item' %>
					</div>
				</li>
				<li class="nav-item">
					<%= link_to 'List of Likes', likes_posts_path, class: 'nav-link' %>
				</li>
				<li class="nav-item dropdown dropdown-slide">
					<a href="#" class="nav-link" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" id="header-profile">
						<%= image_tag 'sample', size: '40x40', class: 'rounded-circle mr15'%>
					</a>
					<div class="dropdown-menu dropdown-menu-right">
						<div class="dropdown-item"><%= "#{current_user.last_name} #{current_user.first_name}" %></div>
						<div class="dropdown-divider"></div>
						<%= link_to 'Profile', '#', class: 'dropdown-item' %>
						<%= link_to 'Logout', logout_path, method: :delete, class: 'dropdown-item' %>
					</div>
				</li>
			</ul>
		</div>
	</nav>
</header>
```




