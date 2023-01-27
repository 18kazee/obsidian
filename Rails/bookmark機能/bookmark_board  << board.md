
掲示板の情報のレコードが引数boardに格納されbookmarks_boardsに`<<演算子`で追加されている。

`<<`は指定されたオブジェクトの末尾に破壊的に追加できるメソッド。  
強制的に追加されて保存もされているので`saveメソッドなどは必要ない。`

** `bookmarks_boards << board`は`bookmarks.create!(board_id: board.id)`と同様の処理がされている。**