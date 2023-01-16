
## formをパーシャルで作成

```ruby
#_form.html.erb

<%= form_with model: board, local: true, class: "form-group" do |f| %>
  <%= render "shared/error_messages", object: f.object %>
  <div class="form-group">
    <%= f.label :title %>
    <%= f.text_field :title, class: "form-control", id: "board_title" %>
  </div>  
  <div class="form-group">
    <%= f.label :body %>
    <%= f.text_area :body, class: "form-control", id: "board_body", rows: "10" %>
  </div>
  <div class="form-group">
    <%= f.label :board_image %>
    <%= f.file_field :board_image, class: "form-control", accept: 'image/*', onchange: 'previewFileWithId(preview)' %>
    <%= f.hidden_field :board_image_cache %>
    <div class='mt-3 mb-3'>
    <%= image_tag board.board_image.url, id: 'preview', size: '300x200' %>
    </div>
  </div>
  <%= f.submit (t 'defaults.create'), class: "btn btn-primary"%>
<% end %>

```


### 1

```ruby
<%= f.file_field :board_image, class: "form-control", accept: 'image/*', onchange: 'previewFileWithId(preview)' %>
```

`file_field`
画像用formはfile_fieldを使用。

 `accept: 'image/*'`
画像ファイル全般を指定しています。

`onchange`
[[onchange]]


### 2

```ruby 
<%= image_tag board.board_image.url, id: 'preview', size: '300x200' %>
```

プレビューを表示しています。boardモデルのboard_imageのurlを呼び出し、表示しています。

### 3

```ruby
<%= f.hidden_field :board_image_cache %>
```

hidden属性でimage_cacheは、画像を指定したけれども、バリデーションエラーなどにより保存が失敗した場合の画面再表示時などに、
画像情報をキャッシュしておくための領域。

