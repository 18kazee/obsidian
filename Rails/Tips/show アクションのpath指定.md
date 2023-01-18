
```ruby
<%= link_to board.title, board_path(board.id ) %>
```

board.showに遷移させるためには、idを指定していないとどこに移動するか決まらないため、
`board.id`のように引数で指定すること。
