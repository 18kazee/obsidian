
[![Image from Gyazo](https://i.gyazo.com/5812e7ab7a6a2997142829a29d99a90d.png)](https://gyazo.com/5812e7ab7a6a2997142829a29d99a90d)

[![Image from Gyazo](https://i.gyazo.com/2396422af4f0ddd6fb61bc7ec012a1b1.png)](https://gyazo.com/2396422af4f0ddd6fb61bc7ec012a1b1)


画像を投稿するページで、画像未選択のままプレビュー画面を開くとエラーになりました。

サーバのエラーログから`_media_image.html.slim`と`app/models/article.rb`と`app/controllers/admin/articles/previews_controller.rb`が怪しい


### if文を使用する
最終的にレンダリングされている`shared/_media_image.html.slim`を見てみると、画像がない場合を想定していないコードでしたので、if文を用いて条件分岐する

```ruby
ruby:
  medium = local_assigns[:medium]

- if medium.image_url                # 追加
  .media-image
    = image_tag medium.image_url(:lg)
```

下記のように記載すると判定結果がelseの場合に空の`media-image`タグが生成されてしまうのでNGです。

```ruby
  .media-image
    - if medium.image_url
      = image_tag medium.image_url(:lg)
```