
Configは環境固有の設定を簡単で使いやすい方法で管理できるようになるgemです。

```bash
gem 'config'
```

```bash
$ bundle install
```

```bash
$ rails g config:install
```

```bash
> rails g config:install                                                                                      ?21_password_reset
Running via Spring preloader in process 78325
      create  config/initializers/config.rb
      create  config/settings.yml
      create  config/settings.local.yml
      create  config/settings
      create  config/settings/development.yml
      create  config/settings/production.yml
      create  config/settings/test.yml
      append  .gitignore
```

これにより、カスタマイズ可能な構成ファイル`config/initializers/config.rb`と一連のデフォルト設定ファイルが生成されます。

config/initializers/config.rb → configの設定ファイル

config/settings.yml → すべての環境で利用する定数を定義

config/settings.local.yml → ローカル環境のみで利用する定数を定義

config/settings/development.yml → 開発環境のみで利用する定数を定義

config/settings/production.yml → 本番環境のみで利用する定数を定義

config/settings/test.yml → テスト環境のみで利用する定数を定義