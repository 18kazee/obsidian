
![[Pasted image 20230428011606.png]]

直訳すると「nilからStringへの暗黙の変換がない」という意味のエラーです。

```ruby
result << if article_block.sentence?
                  sentence = article_block.blockable
                  sentence.body
```

`sentence.body`の中身が`nil`である事からエラーになっていることが推測されます。

## nilガードを使用

今回は`nil`を防ぐ為のテクニックとして、`nil`ガードを使いたいと思います。

```ruby
result << if article_block.sentence?
                  sentence = article_block.blockable
                  sentence.body ||= ''
```

```plain
sentence.body ||= ''
```

上記コードはつまり、`sentence.body`があれば`sentence.body` を返し、無ければ`''`を代入する。といった意味です。

nilガードは、変数にnilが入っているかもしれない状況で、nilの代わりに何らかのデフォルト値を入れておきたいという場面で便利なメソッドです。

これで`sentence.body`の中身が`nil`である場合も、安心して中身が必ず入っている前提のコードを書くことができます。

----

>参考
>https://osamudaira.com/345/
