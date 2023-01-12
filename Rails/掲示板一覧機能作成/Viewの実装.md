
### index.html
```ruby
<div class="container pt-3">
   <div class="row">
     <div class="col-lg-10 offset-lg-1">
       <!-- 検索フォーム -->
       <form>
         <div class="input-group mb-3">
           <input class="form-control" placeholder="検索ワード" type="search" />
           <div class="input-group-append">
             <input type="submit" value="検索" class="btn btn-primary" />
           </div>
         </div>
       </form>
     </div>
   </div>

   <!-- 掲示板一覧 -->
   <div class="row">
     <div class="col-12">
       <div class="row">
         <% if @boards.present? %>
           <%= render @boards %> #下記で説明
         <% else %>
           <p><%= t('.no_result') %></p>
         <% end %>
       </div>
     </div>
   </div>
 </div>
 ```
  
  「<%= render partial: 'board', locals: { board: @board }%>」の省略形は
  「<%= render 'board',　{ board: @board } %>」となっている。
  インスタンス変数名（＠board）とファイル名（_board.html.erb）が同じ場合
  「<%= render @boards %>」のように、[[部分テンプレートの省略]]ができる。


### _ board.html
```ruby
<div class="col-sm-12 col-lg-4 mb-3">
  <div id="board-id-<%= board.id %>">
    <div class="card">
      <%= image_tag 'app/assets/images/に配置した写真のデータ名', class: 'card-img-top', size: '300x200' %>
      <div class="card-body">
        <h4 class="card-title">
          <a href="#">
            <%= board.title %>
          </a>
        </h4>
        <div class='mr10 float-right'>
          <a href="#"><%= icon 'fas', 'trash', class: 'pr-1' %></a>
          <a href="#"><%= icon 'fa', 'pen' %></a>
        </div>
        <ul class="list-inline">
          <li class="list-inline-item">
            <%= icon 'far', 'user' %>
            <%= board.user.decorate.full_name %>
          </li>
          <li class="list-inline-item">
            <%= icon 'far', 'calendar' %>
            <%= l board.created_at, format: :long %>
          </li>
        </ul>
        <p class="card-text"><%= board.body %></p>
      </div>
    </div>
  </div>
</div>
```


### ヘッダーに掲示板一覧ページへのリンクを追加
```ruby
<div class="dropdown-menu dropdown-menu-right">
  <%= link_to t('boards.index.title'), boards_path, class: 'dropdown-item' %>
  <%= link_to '掲示板作成', 'new_board_path', class: 'dropdown-item' %>
</div>
```
