```ruby
# shared/_error_messages

<% if object.errors.any? %>
  <div class="alert alert-danger">
    <ul class="mb-0">
      <% object.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

```ruby
# app/views/boards/_form.html.erb

<%= form_with model: board, local: true do |f| %>
  <%= render 'shared/error_messages', object: f.object %>
  <div class="form-group">
    <%= f.label :title %>
```

エラーメッセージ表示部分をformのテンプレートに記載せず、専用のパーシャルを作成
-   この際、特定のモデルに依存せず汎用的なつくりにする必要がある。格納場所も`board`ではなく`shared`配下にすること。
-   エラーメッセージの表示方法はアプリケーションごとに異なるため、エラーメッセージを直接生成するようなビューヘルパーはRailsに含まれていません。  
-   表示が必要な場合はパーシャル化して、汎用的に使えるようにしておく。
-   パーシャルで書いているので、[[f.object]]を使用する。

>参照
>https://qiita.com/d0ne1s/items/25acf0e70c12042c4c35
>
>https://railsguides.jp/active_record_validations.html#%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%A8%E3%83%A9%E3%83%BC%E3%82%92%E3%83%93%E3%83%A5%E3%83%BC%E3%81%A7%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B
