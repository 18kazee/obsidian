
## enum用意

Embedモデルのembed_typeカラムにyoutubeとtwitterというenum を設定します。

```ruby
# models/embed.rb

class Embed < ApplicationRecord
  has_one :article_block, as: :blockable, dependent: :destroy
  has_one :article, through: :article_block

  enum embed_type: { youtube: 0, twitter: 1 }

  validates :identifier, length: { maximum: 200 }
end
```

## 記事ブロック見出しアイコンをTwitterに対応させる

![](https://i.gyazo.com/60fb0c386264e3251409f53e79663194.png)
Twitterのとき

![](https://i.gyazo.com/9ae9a5f39aeabd846d72227303e115d8.png)
YouTubeのとき

`embed`が`youtube`なのか、`twitter`なのか条件分岐させます。 下記で使われているメソッドは`embed_type`としてenumを設定しているので使えています。

```ruby
# decorators/article_block_decorator.rb

module ArticleBlockDecorator
  def box_header_icon
    if sentence?
      '<i class="fa fa-edit"></i>'.html_safe
    elsif medium?
      '<i class="fa fa-image"></i>'.html_safe
    elsif embed?
      blockable.youtube? ? '<i class="fa fa-youtube-play"></i>'.html_safe : '<i class="fa fa-twitter"></i>'.html_safe      # 三項演算子を使ってスッキリ書く
    end
  end
end
```

## 記事ブロック挿入画面にTwitterアイコンを追加

![](https://i.gyazo.com/dafa64f08f766118512bec874387633f.jpg)

記事ブロック挿入画面のアイコンを、YouTubeとTwitterが横並びになるようにしましょう。

#### d-inline-flex
```ruby
#views/admin/article_blocks/_insert_block.html.slim

.d-inline-flex
  i.fa.fa-youtube-play
  i.fa.fa-twitter
'
| 埋め込み
```

bootstrapで用意されているインラインのFlexコンテナーです。 こちらを使うことで、横並びを実現しています。 ブラウザでは下記のようにレンダリングされています。

```ruby
<div class="d-inline-flex">
  <i class="fa fa-youtube-play"></i>
  <i class="fa fa-twitter"></i>
</div>
```

## 入力フォーム

simple_formの中で、`Youtube`か`twitter`かを選択し、`identifier`を入力できるようなフォームにします。

```ruby
# views/admin/articles_blocks/edit_embed.html.slim

.box-body
    = f.input :embed_type, collect: Embed.embed_types_i18n.invert, include_blank: false
    = f.input :identifier
```

## youtubeのID抽出メソッド作成

以下のように、`/` を区切り文字としてyoutubeのURLを後ろの`id`のみを抽出する`split`メソッドを`Embed`モデル定義しましょう。

```ruby
# models/embed.rb

def split_id_from_youtube_url
  identifier.split('/').last if youtube?
end
```

`embed`に対して、作成したメソッドを実行して下11桁の文字列だけ取り出して表示させましょう。


```ruby
# app/shared/_embed_youtube.html.slim

.embed-youtube
  = content_tag 'iframe', nil, width: width, height: height, src: "https://www.youtube.com/embed/#{embed.split_id_from_youtube_url}", \
    frameborder: 0, gesture: 'media', allow: 'encrypted-media', allowfullscreen: true
```

## Twitter埋め込み表示

▷[タイムラインを埋め込む方法](https://help.twitter.com/ja/using-twitter/embed-twitter-feed)  
貼り付け用に変換してくれる[公式ツール](https://publish.twitter.com/#)があるので、ここにツイートを貼り付けて埋め込み用HTMLを生成する。

以下実際にHTMLをコピーしていた内容。

```html
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Twitter上にいる全猫たちと飼い主さんたち、今日はたくさんの癒しをありがとうございました！<a href="https://twitter.com/hashtag/%E7%8C%AB%E3%81%AE%E6%97%A5?src=hash&amp;ref_src=twsrc%5Etfw">#猫の日</a><br><br>写真は <a href="https://twitter.com/hashtag/Twitter%E7%8C%AB%E3%81%AE%E6%97%A5%E3%82%A2%E3%83%B3%E3%83%90%E3%82%B5%E3%83%80%E3%83%BC?src=hash&amp;ref_src=twsrc%5Etfw">#Twitter猫の日アンバサダー</a> <a href="https://twitter.com/slurmQuEBJUJK3X?ref_src=twsrc%5Etfw">@slurmQuEBJUJK3X</a> さんの猫ちゃんです✨ <a href="https://t.co/QVa21bdv3E">pic.twitter.com/QVa21bdv3E</a></p>&mdash; Twitter Japan (@TwitterJP) <a href="https://twitter.com/TwitterJP/status/1496116675858804736?ref_src=twsrc%5Etfw">February 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
```

これもYoutube同様無駄な部分を省く。

```html
<blockquote class="twitter-tweet"> <a href="https://twitter.com/TwitterJP/status/1496116675858804736?ref_src=twsrc%5Etfw">February 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
```

省いたものを、`view`に反映させればOK。


```ruby
# _embed_twitter.html.slim
.embed-twitter
  blockquote.twitter-tweet
    a href="#{embed.identifier}"
  script async="" charset="utf-8" src="https://platform.twitter.com/widgets.js"
```

`#{embed.identifier}`で任意のツイートのURLを埋め込むことができる。

https://qiita.com/mmaumtjgj/items/de76cfbc14f8690eae63