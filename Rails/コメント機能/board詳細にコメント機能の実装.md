
- commentモデルの作成
- アソシエーションの設定、バリデーションの設定
- コントローラーの作成
- routesの設定
- viewの作成

## commentモデルの作成

```shell
$ bin/rails g model comment user:references, board:references 
```

### アソシエーションの設定、バリデーションの設定

```ruby
# user.rb
has_many :boards, dependent: :destroy
has_many :comments, dependent: :destroy
```

```ruby
# board.rb
belongs_to :user
has_many :comments dependent: :destroy
```

```ruby
# comment.rb
belongs_to :user
belongs_to :bpard

validates :body, presence: true, length: { maximum: 65_535 }
```

```shell
$ bin/rails db:migrate
```

## コントローラーの作成

```shell
$ bin/rails g controller comments
```

```ruby
class CommentsController < ApplicationController
	def create
		comment = current_user.comments.build(comment_params)
		if comment.save
			redirect_to board_path(comment.board), success: t('defaults.message.created', item: Comment.model_name.human)
		else
			redirect_to board_path(comment.board), danger: t('defaults.message.not_created', item: Comment.model_name.human)
		end
	end

	private

	def comment_params
		params.require(:comment).permit(:board).merge(board_id: params[:board_id])
	end
end
```

### モデル user.rbにコメントで使用するコントローラー記載

```ruby
# user.rb

authenticates_with_sorcery!

has_many :boards, dependent: :destroy
has_many :comments, dependent: :destroy

validates :password, length: { minimum: 3 }, if: -> { new_record? || changes[:crypted_password] }
validates :password, confirmation: true, if: -> { new_record? || changes[:crypted_password] }

validates :email, presence: true
validates :first_name, presence: true, length: { maximum: 255 }
validates :last_name, presence: true, length: { maximum: 255 }

def own(object)
	id == object.user_id
end
```

##### def own
`(self.)id == object.user_id`という意味で、自分自身のidと同じかどうか検証できるアクションを作成している

### boardsコントローラーに追記

```ruby
def show
	@board = Board.find(params[:id])
	@comment = Comment.new
	@comments = @board.comments.includes(:user).order(created_at: :desc)
```

## routesの設定

```ruby
resouces :boards, only %i[index new create show] do
	resouces :comments, only %i[create], shallow: true
end
```

## viewの作成

### formの作成

```ruby
#boards/_form.html.erb
<div class="row mb-3">
	<div class="col-lg-8 offset-lg-2">
		<%= form_with model: comment, url: [board, comment], local: true do |f|
			<%= f.lavel :body %>
			<%= f.text_area :body, class: 'form-control mb-3', id: 'js-new-comment-body', row: 4, placeholder: Comment.human_attribute_name(:body) %>
			<%= f.submit t('defaluts.post'), class: 'btn btn-primary' %>
		<% end %>
	</div>
</div>
```

### boards/_comment.html.erb

```ruby
<tr id="comment-<%= comment.id %>">
	<td style="width 60px">
		<%= image_tag 'sample.jpg', class: 'round-circle', size: '50x50' %>
	</td>
	<td>
		<h3 class="small"><%= comment.user.decorate.full_name %></h3>
		<div id="js-comment-<%= comment.id %>">
			<%= simple_format(comment.body) %>
		</div>
		<div id="js-textarea-comment-box-<%= comment.id %>", style="display: none;" >
			<textarea id="js-textarea-comment-<%= comment.id %>", class="form-control mb-1"><%= comment.body %></textarea>
			<button class="btn btn-light js-button-edit-comment-cancel"  data-comment-id="<%= comment.id %>">キャンセル</button>
			<button class="btn btn-success js-button-comment-update" data-comment-id="<%= comment.id %>">更新</button>
		</div>
	</td>

	<% if current_user.own?(comment) %>
		<td class action>
			<ul class="list-inline justify-content-center" style="float: right;">
				<li class="list-inline-item">
					<a href="#" class="js-edit-comment-button" data-comment-id="<%= comment.id %>">
						<%= icon 'fa', 'pen' %>
					</a>
				</li>
				<li class="list-inline-item">
					<a href="#" class="js-delete-comment-button" data-comment-id="<%= comment.id %>">
						<%= icon 'fas', 'trash' %>
					</a>
				</li>
			</ul>
		</td>
	<% end %>
</tr>
```

 [[simple_format]]

### boards/comments.html.erb

```ruby
<div class="row">
	<div class="col-lg-8 offset-lg-2">
		<table id="js-table-comment" class="table">
			<%= render conments %>
		</table>
	</div>
</div>
```

### boards/show.html.erb

```ruby
<div class="container pt-5">
	<div class="row mb-3">
		<div class="col-lg-8 offset-lg-2">
			<h1><%= t('.title') %></h1>
			
			<!-- 掲示板内容 -->
			<article class="card">
				<div class="card-body">
					<div class="row">
						<div class="col mb-3">
							<%= image_tag @board.board_image.url, class: 'caard-img-top img-fluid', size: '300x200' %>
						</div>
						<div class="col-md-9">
							<h3 class="d-inline"><%= @board.title %></h3>
							<%= render 'clud_menus', board: @board %>
							<ul class="list-inline">
								<li class="list-inline-item">by <%= @board.user.decorate.full_name %></li>
								<li class="list-inline-item"><%= l @board.created_at, format: :long %></li>
						  </ul>
						</div>	
					</div>
					<p><%= simple_format(@board.body) %></p>
				</div>
			</article>
		</div>
	</div>
	
	<!-- コメントフォーム -->
	<%= render 'comments/form', { board: @board, comment: @comment } %>
	<!-- コメントエリア -->
	<%= render 'comments/comments', { comments: @comments } %>
</div>
```

### boards/clud_menu_btn.html.erb

```ruby
<ul class='crud-menu-btn list-inline float-right'>
	<li class="list-inline-item">
		<%= link_to '#', id: "button-edit-#{board.id}" do %>
			<%= icon 'fa', 'pen' %>
		 <% end %>
	</li>
	<li class="list-inline-item">
		<%= link_to '#', id: "button-delete-#{board.id}" do %>
			<%= icon 'fas', 'trash' %>
		<% end %>
	</li>
</ul>
```
