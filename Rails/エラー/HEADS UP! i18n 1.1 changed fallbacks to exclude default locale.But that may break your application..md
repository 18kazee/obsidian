
## bundle install時に注意が出た

### 対応策

```ruby
#/config/environments/production.rb
config.i18n.fallbacks = [I18n.default_locale]
```

### 現象

```shell
$ bundle install
```

```ruby
HEADS UP! i18n 1.1 changed fallbacks to exclude default locale.
But that may break your application.

If you are upgrading your Rails application from an older version of Rails:
Please check your Rails app for 'config.i18n.fallbacks = true'.
If you're using I18n (>= 1.1.0) and Rails (< 5.2.2), this should be

'config.i18n.fallbacks = [I18n.default_locale]'.

If not, fallbacks will be broken in your app by I18n 1.1.x.
If you are starting a NEW Rails application, you can ignore this notice.

For more info see:
https://github.com/svenfuchs/i18n/releases/tag/v1.1.0
```

### Ruby の i18n gemとは

[[i18n gem]]

### config.i18n.fallbacks とは？

[[config.i18n.fallbacks]]

### fallbackとは

[[fallback]]

### 参照
https://qiita.com/homhom_star/items/ac0fd3769f1d7f974fa4
https://mocomo012.hatenablog.com/entry/2019/10/16/194221
