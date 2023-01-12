### 現在ログイン中かどうか、true or falseで返す。

コントローラ、ビューで使える。
ログインしているかどうかによって、
場合分けをしたいときに使うことが多い。

```ruby
# view.html.erb
<% if logged_in? %>
  <%= link_to 'プロフィール', user_url(current_user) %>
<% else %>
  <%= link_to 'ログイン', login_url %>
<% end %>
```
