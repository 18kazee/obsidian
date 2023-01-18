
不必要なURLを短くできるRailsのルーティングのオプション。
ネストしたresourcesで使用する。

```ruby
resources boards, only %i[index new create] do
	resources comments, only %i[create], shallow: true
end
```

`shallow`オプションを指定すると、
index,new,createなどのidを持たないアクション（コレクション）の場合は、idが省略されます。
反対に、show,edit,update,destroyのようなidが必要なアクション（メンバー）の場合は、
ネストに含まれないようになります。

### ネストしたURLの場合のform_with

コメント作成のときのform_with
modelに`comment`を指定することと、URLに`board`と`comment`の両方使うため、引数を2つ指定するようにします。

```ruby
# 省略形
<%= form_with model: comment, url:[board, comment], local: true do |f| %>  
```

```ruby
<%= form_with model: comment, url: board_comment_path(board, comment), local: true do |f| %>
```

>参照
>https://study-output.hatenadiary.com/entry/2021/04/30/203733
>