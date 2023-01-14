
### 対応策
```shell
$ gem install bundler -v 2.1.4
```

### エラー

Gemfile.lockにbundler 2.1.4が指定しているが、そのバージョンのbundlerが見当たらないというエラー。

```shell
Traceback (most recent call last): 2: from 1: from 
#省略 Could not find 'bundler' (2.1.4) required by your /Users/<ユーザー名>/<ルートディレクトリ名>/Gemfile.lock. (Gem::GemNotFoundException) To update to the latest version installed on your system, run `bundle update --bundler`. To install the missing version, run `gem install bundler:2.1.4`
```

Rubyのバージョンの変更をしたときなどに起こる。rbenvではRubyのバージョンごとにgemをインストールする必要があるとのこと。


>参照
>https://qiita.com/Kenchiki/items/a54466d44f4164719b8a