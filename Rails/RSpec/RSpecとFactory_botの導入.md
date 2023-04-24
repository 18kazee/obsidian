
### Gemファイルに記載
**読み込むのは開発環境とテスト環境だけで、本番環境には読み込まない**点に注意

Gemfile
```ruby
group :development, :test do
  gem 'rspec-rails'
  gem "factory_bot_rails"
end
```

```shell
$ bundle install
```

### RSpec設定ファイルのインストール
```shell
$ bin/rails g rspec:install
```
以下が生成される
-   .rspec→RSpec用の設定ファイル
-   spec→specファイル（テストファイル）関連を格納するディレクトリ
-   spec_helper.rb＆rails_helper.rb→RSpecのカスタマイズ用ヘルパーファイル

### .gitignoreにvendor/bandle追加
インストールしたGemが全てGit追跡対象のため解除する

