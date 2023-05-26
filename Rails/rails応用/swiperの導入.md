
## swiperとは

jQueryに依存すること無く動作する最もモダンなモバイルタッチスライダーです。モバイルで使用知ることを想定しているため、スマホやタブレットでもスムーズにスライダー機能を実装することができます。

![](https://i.gyazo.com/4620adff5176307b2947f037e305bf85.gif)

出典：https://swiperjs.com/demos#autoplay


## yarnでパッケージを導入

package.jsonを作成するために以下のコマンドを打ちます。（gemで言う所の$ bundle initに似てますね。）もともとファイルがあればやる必要はないです。

```
$ yarn init
```

次に、swiperを導入します。

```
$ yarn add swiper
```

こうすると、package.jsonにswiperの記載がなされるはずです！
インストールします。

```
$ yarn install
```

必要なファイルがnode_modulesディレクトリ配下に作成されます。

## 導入したファイルの読み込み設定

マニフェストファイルに導入したファイルのpathを記載して、読み込みの設定を書きます。


assets/javascript/application.js
```js
//= require swiper/swiper-bundle.min.js
//= require swiper.js
```

ディレクトリのpathはnode部分は省略できるので、swiperから書きます。`//= require swiper.js`は、後ほど、viewと連動させるためのファイルの読み込みを書いています。
scssにもスタイルの読み込みを書きます。

assets/stylesheets/application.scss
```css
@import 'swiper/swiper-bundle.min.js';
```

この後の部分の記載を失念してしまうのが、ハマりポイントで、私もハマりました。
導入したnode以下のファイルを読み込むようにするための設定を書く必要があります

config/initializers/assets.rb
```ruby
Rails.application.config.assets.paths << Rails.root.join('node_modules')
```

