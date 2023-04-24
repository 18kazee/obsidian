
![[スクリーンショット 2023-04-24 11.33.16.png]]![[スクリーンショット 2023-04-24 11.37.16.png]]


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