
アクションケーブルで削除ボタンの表示を、current_userで判定する時

viewでid指定とcurrent_userのuser-idを入れ込む
```ruby
<div id="current-user-info" data-user-id="<%= current_user.id %>"></div>
```


jsで受け取る
```js
const currentUserId = document.getElementById('current-user-info').getAttribute('data-user-id');
```

jsで判定
```js
${
    data.comment.user_id == currentUserId
    ? `
        <a class="delete-comment-button" 
                  data-turbo-method="delete" 
                  data-turbo-confirm="本当に削除しますか？" 
                  data-comment-id="${data.comment.id}" 
                  href="/posts/${data.comment.post_id}/comments/${data.comment.id}">
                  <i class="bi bi-trash"></i>
        </a>`
    : ''
}
```

以上

以下全文

```ruby
class CommentChannel < ApplicationCable::Channel
  def subscribed
    stream_from "comment_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
end
```



app/javascript/channels/comment_channel.js
```js
import consumer from "./consumer"

consumer.subscriptions.create("CommentChannel", {
  connected() {
    // Called when the subscription is ready for use on the server
  },

  disconnected() {
    // Called when the subscription has been terminated by the server
  },

  received(data) {
    const currentUserId = document.getElementById('current-user-info').getAttribute('data-user-id');
    console.log(currentUserId);
    if (data.action === 'delete') {
      const deletedCommentContainer = document.querySelector(`#comment_${data.comment_id}`);
      if (deletedCommentContainer) {
      deletedCommentContainer.remove();
      }
    } else {
      const html = `
        <div class="card" id="comment_${data.comment.id}">
          <div class="card-body">
            <div class="comment-user">
              <div class="comment-user-name">${data.user.name}</div>
              <div class="comment-user-body">${data.comment.body}</div>
              <div class="comment-user-time">${data.comment.created_at}</div>
              ${
                data.comment.user_id == currentUserId
                ? `
              <a class="delete-comment-button" 
                        data-turbo-method="delete" 
                        data-turbo-confirm="本当に削除しますか？" 
                        data-comment-id="${data.comment.id}" 
                        href="/posts/${data.comment.post_id}/comments/${data.comment.id}">
                        <i class="bi bi-trash"></i>
              </a>`
                : ''
              }
            </div>
          </div>
        </div>
      `;
      const comments = document.getElementById('comments');
      const newComment = document.getElementById('comment_body');
      comments.insertAdjacentHTML('afterbegin', html);
      newComment.value='';
    }
  },
});
```

app/views/post/show.html/erb
```html
<div class="show-comment">
  <% if logged_in? %>
    <%= bootstrap_form_with model: [@post, @comment],local: true do |f| %>
      <div class="card">
        <div class="card-body"> 
          <%= f.text_field :body, class: 'form-control' %>
          <%= f.submit 'コメントする', class: "btn btn-primary btn-sm" %>
        </div>
      </div>
    <% end %>
  <% end %>

  <div class="comments">コメント一覧</div>
    <div id="comments">
      <% @comments.each do |comment| %>
        <div class="card" id="comment_<%= comment.id %>">
          <div class="card-body">
            <div class="comment-user">
              <div class="comment-user-name"><%= comment.user.name %></div>
              <div class="comment-user-body"><%= comment.body %></div>
              <div class="comment-user-time"><%= comment.created_at.strftime("%Y/%m/%d %H:%M") %></div>
              <% if current_user %>
              <div id="current-user-info" data-user-id="<%= current_user.id %>"></div>
              <% end %>
              <% if comment.user == current_user %>
              <%= link_to post_comment_path(@post, comment), class: 'delete-comment-button', data: {turbo_method: :delete, turbo_confirm: '本当に削除しますか？', comment_id: comment.id} do %>
                <i class="bi bi-trash"></i>
              <% end %>
              <% end %>
            </div>
          </div>
        </div>
      <% end %>
    </div>
  </div>
</div>
```