
RAILSGUIDS / Railアプリを設定する/3 Railsコンポーネントを構成する/[3.5 i18nを設定する](https://railsguides.jp/configuring.html#i18n%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)  によると、

> config.i18n.fallbacksは訳文がない場合のfallback動作を設定します。

として、具体的な３つの設定パターンが紹介されています。

## 1.　defaultのlocaleをfallback先として使う場合、trueを設定。

/config/environments/production.rb
```
config.i18n.fallbacks = true
```

## 2.　localeの配列をfallback先に使う場合。

/config/environments/production.rb
```
config.i18n.fallbacks = [:tr, :en]
```

## 3.　localeごとに個別のfallbackを設定する場合。 例:azと:deに:trを、:daに:enをそれぞれfallback先として指定する場合は、次のようにします。

/config/environments/production.rb
```
config.i18n.fallbacks = { az: :tr, da: [:de, :en] }
#or
config.i18n.fallbacks.map = { az: :tr, da: [:de, :en] }
```
