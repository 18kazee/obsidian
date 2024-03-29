
```
@boards = @q.result(distinct: true).includes(:user).order(created_at: :desc).page(params[:page])
```

このメソッドが必要になるのは「関連する子テーブルの情報を条件に絞り込んで、親テーブルの検索結果を表示するとき」のケースです。
例えば、絞り込み要件が「xxxというコメントがついている掲示板を取得する」場合に distinct: true がないと結果がおかしくなります。

掲示板Aに対して「rubyの勉強楽しい」というコメントと「ruby完全に理解した」というコメントがあったとします。

![[Pasted image 20230205220654.png]]

distinct: true がないと 検索窓でruby とコメントを検索した場合に、掲示板Aが2回取得されて検索結果が2件になってしまいます。

![[Pasted image 20230205220639.png]]

これがなぜ起こるのかというと、関連テーブルをjoinすると、一つの掲示板につきコメントの数だけ行ができます。

これらを ruby という文字列で検索すると、どちらの行も該当するので「掲示板A」の情報が2件取得されます。  
この 同じレコードが結果に複数件表示される場合 に、 distinct を使って重複を取り除く必要があります。

上記例の様な distinct: true が必要なときもあれば、必要ない場合もあります。  
ただ、検索時に重複したデータを結果として表示したいケースは稀ですし、
仮に先ほどの具体例の様に検索条件が変更された場合に問題になるかもしれないので、 
distinct: true は癖として毎回つけておくと良いです。