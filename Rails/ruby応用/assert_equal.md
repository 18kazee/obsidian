
**`assert_equal( expected, actual, [msg] )`**
-   `expected == actual`はtrueであると主張する。

つまり第一引数と第二引数が等しいことを期待してテストしている。
また、比較するオブジェクトを文字列として比較します。

rubyのテストでアサーション(assert)は使われる。
他にもいろいろある。


---
以下railsのmini testで使用できるアサーション

アサーションは非常に多くの種類が使えるようになっています。 以下で紹介するのは、[`Minitest`](https://github.com/seattlerb/minitest)で使えるアサーションからの抜粋です。MinitestはRailsにデフォルトで組み込まれているテスティングライブラリです。`[msg]`パラメータは1個のオプション文字列メッセージであり、テストが失敗したときのメッセージをわかりやすくするにはここで指定します（必須ではありません）。

**`assert( test, [msg] )`**
-   `test`はtrueであると主張する。

**`assert_not( test, [msg] )`**
-   `test`はfalseであると主張する。

**`assert_equal( expected, actual, [msg] )`**
-   `expected == actual`はtrueであると主張する。

**`assert_not_equal( expected, actual, [msg] )`**
-   `expected != actual`はtrueであると主張する。

**`assert_same( expected, actual, [msg] )`**
-   `expected.equal?(actual)`はtrueであると主張する。

**`assert_not_same( expected, actual, [msg] )`**
-   `expected.equal?(actual)`はfalseであると主張する。

**`assert_nil( obj, [msg] )`**
-   `obj.nil?`はtrueであると主張する。

**`assert_not_nil( obj, [msg] )`**
-   `obj.nil?`はfalseであると主張する。

**`assert_empty( obj, [msg] )`**
-   `obj`は`empty?`であると主張する。

**`assert_not_empty( obj, [msg] )`**
-   `obj`は`empty?`ではないと主張する。

**`assert_match( regexp, string, [msg] )`**
-   stringは正規表現 (regexp) にマッチすると主張する。

**`assert_no_match( regexp, string, [msg] )`**
-   stringは正規表現 (regexp) にマッチしないと主張する。

**`assert_includes( collection, obj, [msg] )`**
-   `obj`は`collection`に含まれると主張する。

**`assert_not_includes( collection, obj, [msg] )`**
-   `obj`は`collection`に含まれないと主張する。

**`assert_in_delta( expected, actual, [delta], [msg] )`**
-   `expected`と`actual`の個数の差は`delta`以内であると主張する。

**`assert_not_in_delta( expected, actual, [delta], [msg] )`**
-   `expected`と`actual`の個数の差は`delta`以内にはないと主張する。

**`assert_in_epsilon ( expected, actual, [epsilon], [msg] )`**
-   `expected`と`actual`の個数の差が`epsilon`より小さいと主張する。

**`assert_not_in_epsilon ( expected, actual, [epsilon], [msg] )`**
-   `expected`と`actual`の数値には`epsilon`より小さい相対誤差がないと主張する。

**`assert_throws( symbol, [msg] ) { block }`**
-   与えられたブロックはシンボルをスローすると主張する。

**`assert_raises( exception1, exception2, ... ) { block }`**
-   渡されたブロックから、渡された例外のいずれかが発生すると主張する。

**`assert_instance_of( class, obj, [msg] )`**
-   `obj`は`class`のインスタンスであると主張する。

**`assert_not_instance_of( class, obj, [msg] )`**
-   `obj`は`class`のインスタンスではないと主張する。

**`assert_kind_of( class, obj, [msg] )`**
-   `obj`は`class`またはそのサブクラスのインスタンスであると主張する。

**`assert_not_kind_of( class, obj, [msg] )`**
-   `obj`は`class`またはそのサブクラスのインスタンスではないと主張する。

**`assert_respond_to( obj, symbol, [msg] )`**
-   `obj`は`symbol`に応答すると主張する。

**`assert_not_respond_to( obj, symbol, [msg] )`**
-   `obj`は`symbol`に応答しないと主張する。

**`assert_operator( obj1, operator, [obj2], [msg] )`**
-   `obj1.operator(obj2)`はtrueであると主張する。

**`assert_not_operator( obj1, operator, [obj2], [msg] )`**
-   `obj1.operator(obj2)`はfalseであると主張する。

**`assert_predicate ( obj, predicate, [msg] )`**
-   `obj.predicate`はtrueであると主張する (例:`assert_predicate str, :empty?`)。

**`assert_not_predicate ( obj, predicate, [msg] )`**
-   `obj.predicate`はfalseであると主張する (例:`assert_not_predicate str, :empty?`)。

**`flunk( [msg] )`**
-   必ず失敗すると主張する。これはテストが未完成であることを示すのに便利。


https://railsguides.jp/testing.html