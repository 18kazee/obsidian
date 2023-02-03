
- コントローラーの修正
- ルーティングの設定
- コメント一覧のview、ボタン修正
- comments/_formの修正
- jsファイルの作成

## コントローラーの修正

```ruby
#commentsコントローラー
def create
	@comment = current_user.comments.build(params_comment)
	@comment.save
end

def destroy
	@comment = current_user.comments.find(params[:id])
	@comment.destroy!
end

private

def comment_params
	params.require(:comment).permit(:body).merge(board_id: params[:board_id])
end

```

## ルーティングの設定

```ruby
resources :boards , shallow: true do
  resources :comments, only: %i[create destroy]
  collection do
    get 'bookmarks'
  end
end
resources :bookmarks, only: %i[create destroy]
```

comments destroyのルーティングを追加

## コメント一覧のView、ボタン修正

```ruby
<li class="list-inline-item">
	<%= link_to comment_path(comment),
							method: :delete,
							class: 'js-delete-button',
							data: { confirm: t('defaluts.message.delete_confirm') }
							remote: true do %>
		<%= icon 'fas', 'trash' %>
	<% end %>
</li>
```


## comments/_formの修正

```ruby
<div class="row mb-3">
	<div class="col-lg-8 offset-lg-2">
		<%= form_with model: comment, url: [board, comment], id: 'new_comment' do |f| %>
			<%= render 'shared/error_messages', object: f.object %>
			<%= f.label :body %>
			<%= f.text_area :body, class: 'form-control mb-3', id: 'js-new-comment-body', row: 4, placeholder: Comment.human_attribute_name(:body) %>
			<%= f.submit t('defaults.post'), class: 'btn btn-primary' %>
		<% end %>
	</div>
</div>
```

`id: 'new_comment'`の追加
`id: 'js-new-comment-body'`の追加
`local: true`の削除

## jsファイルの作成

```ruby
# create.js.erb
$("#error_messages").remove()
<% if @comment.errors.present? %>
	$("#new_comment").prepend("<%= j(render('shared/error_messages', object: @comment)) %>")
<% else %>
	$("#js-table-comment").prepend("<%= j(render('comments/comment', comment: @comment) %>)")
	$("#js-new-comment-body").val('')
<% end %>
```

```ruby
# destroy.js.erb
$("tr#comment-<%= @comment.id %>").remove()
```

---
>参照
[[remove]]
[[prepend]]
[[Val()]]
[[コメント機能のajax　サイト]]