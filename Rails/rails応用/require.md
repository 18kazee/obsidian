
```ruby
require File.expand_path(File.dirname(__FILE__) + '/environment')
```


コードを紐解いていく

### File.expand_path

リファレンスマニュアルでは以下のように記載がある
```ruby
File.expand_path(path, default_dir = '.') -> String
```
これはpath名を絶対パスに展開した文字列を返すメソッドです。
今回は第二引数を指定しない場合で、カレントディレクトリを相対パスの基準にして、絶対パスに変換します。
例で、

```ruby
curent_path = '~/environment/workdir'
path名 = 'ruby'

File.expand_path(ruby)　-> "~/environment/workdir/ruby" 
```

###  File.dirname
これはfile名がもつディレクトリを文字列で返すメソッドです。
ディレクトリを持たない場合はカレントディレクトリを返します。

```ruby
File.dirname("home/dir/file.ext") # => "home/dir" 
File.dirname("file.ext") # => "."
```


```ruby
(__FILE__)
```

上記は現在の実行中のソースファイル名を返す変数です。コマンドを実行したディレクトリからの相対パスでファイル名を返します。

```ruby
#environment/workdir/hbkk_ruby.rb 
puts __FILE__ 

#~/environmentで 
$ ruby workdir/hbkk.rb 
#=> workdir/hbkk.rb 

#~/environment/workdirで 
$ ruby hbkk.rb 
#=> hbkk.rb
```

### まとめ

schedule.rbのディレクトリである「configまたはカレントパス(.)のどちらか」プラス「"/environment"」をrequireしていることになります。

```ruby
require "./environment" or "config/environment"
```


>参考
https://note.com/hbkk/n/n6115dfd3b6a4
https://stray-light.info/wp/post-597/#toc2
