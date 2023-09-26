
この結果を見ると、`activation_token`がデータベースに保存されていないことが分かります。これは`generate_activation_token`メソッドが呼び出されたときに`activation_token`に適切な値が代入されたにもかかわらず、データベースのINSERT文で`activation_token`が`nil`として扱われていることを示しています。

この問題の原因は、`attr_accessor :activation_token, :activation_token_expires_at`の行が原因です。`attr_accessor`は、Rubyのメタプログラミング機能を使ってインスタンス変数に対するゲッターとセッターを自動的に生成するためのものです。しかし、これはRailsのモデルには必要ありません。

`attr_accessor`はゲッターとセッターを自動的に定義するため、`activation_token`というインスタンス変数が存在しない場合、代わりに新しいインスタンス変数を作成します。しかし、`User`モデルには既に`activation_token`というカラムがデータベースに存在するため、`attr_accessor`が新しいインスタンス変数を作成してしまい、データベースのカラムとは別物として扱われてしまいます。

このため、`attr_accessor :activation_token`の行を削除することで問題が解決されるはずです。以下のように修正してみてください：

rubyCopy code

`class User < ApplicationRecord   authenticates_with_sorcery!   before_create :generate_activation_token    # 省略... end`

これにより、`activation_token`カラムがデータベースに正しく保存されるようになり、ログインのエラーが解消されるはずです。

`attr_accessor`の使い方は便利な場面もありますが、注意が必要な場面もあります。特にRailsのモデルでは、既存のデータベースカラムと同じ名前のインスタンス変数を作成することで予期せぬバグが発生することがありますので、そのような場合は`attr_accessor`を避ける方が良いでしょう。
