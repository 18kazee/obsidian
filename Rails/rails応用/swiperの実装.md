
[[swiperの導入]]　の続き

## JSファイルの作成

CDNから持ってくるJSファイル、  もしくはYarnでインストールしてnode_modules配下に置かれるJSファイルとは別に、  
新しくJSファイルを自分で作成する必要があります。

ここで作成するJSファイルが、先ほど設定した`swiper/js/swiper.js`を参照し、  
スライダー機能を各クラスに適用させるような仕組みとなっています。

app/assets/javascripts/swiper.js
```js
document.addEventListener('DOMContentLoaded', function() {

	const Swiper = new Swiper('.swiper', {
	  // Optional parameters
	  direction: 'vertical',
	  loop: true,
	
	  // If we need pagination
	  pagination: {
	    el: '.swiper-pagination',
	  },
	
	  // Navigation arrows
	  navigation: {
	    nextEl: '.swiper-button-next',
	    prevEl: '.swiper-button-prev',
	  },
	
	  // And if we need scrollbar
	  scrollbar: {
	    el: '.swiper-scrollbar',
	  },
	}),
});
```

公式のデモサイトを参照し、該当のスライダー機能においてどのようなソースコードが  
使用されているか確認してみるとよいでしょう。

## 4. CSSの追加設定

こちらも公式に書いてある内容をそのまま書いているだけなのですが、  
適宜Swiperのサイズなどを設定してください。

> [Swiper公式サイトで紹介されている実装方法（Getting Started With Swiper）](https://swiperjs.com/get-started/)


application.css.scss
```css
.swiper {
    width: 100%;
    height: 400px;
    object-fit: cover;
}
```

## headerにswiperの追加

```slim
//views/layouts/_header.html.slim
header 
  .swiper
    .swiper-wrapper
      - if current_site.main_images.present?
        -current_site.main_images.each do |main_image|
          = image_tag main_image, class: 'swiper-slide'
      - else
        = image_tag '/images/cover.jpg', class: 'swiper-slide'
    .swiper-button-next    # ナビゲーションボタン(→)
    .swiper-button-prev    # ナビゲーションボタン(←)
    .swiper-pagination     # ページネーション

  .container.blog-title
    h1 = link_to current_site.name, root_path
    p.lead = current_site.subtitle
```


----

## 重要

### JSが発火するタイミング

以下公式の書き方では動きません。
```js
const Swiper = new Swiper('.swiper', {
	// Optional parameters
	direction: 'vertical',
	loop: true,

	// If we need pagination
	pagination: {
		el: '.swiper-pagination',
	},

	// Navigation arrows
	navigation: {
		nextEl: '.swiper-button-next',
		prevEl: '.swiper-button-prev',
	},

	// And if we need scrollbar
	scrollbar: {
		el: '.swiper-scrollbar',
	},
}),
```

JSはheadタグ内で呼び出されるよう配置しているためです。
htmlのheadタグ内呼び出されると、bodyタグ内にあるswiperクラスなどのDOM構造を検知していないため動きません。

```html
<html>
	<head> 
		<!-- ここでSwiperのJSを読み込むとする -->
	  <!-- ここでJSが発火すると、<body>タグ内にあるswiper-containerクラスなどのDOM構造をまだ検知していない --> 
	  <!-- SwiperのJSの適用先が分からないので、Swiperによるスライド機能は当然実装されない --> 
	  <!-- ここにSwiperのJSのファイルを置く場合、発火のタイミングを指定してあげる必要がある --> 
	  <!-- jQueryを導入していると $(function() {...}) を活用して、簡単に発火のタイミングを指定してあげることができる -->
	  <!-- jQueryを導入していなくても、JSのコードを書けば発火のタイミングを指定できる --> 
	</head> 
	<body>
		<div class="swiper-container"> 
			<!-- Swiperの構造を示すdivタグが延々と続く -->
		</div> 
		<!-- ここで発火させれば、既にswiper-containerクラスなどのDOM構造を検知した後なので、上手く作動する -->
		<script src="../package/swiper-bundle.js"></script>
		<script>
			var swiper = new Swiper('.swiper-container', { 
				pagination: {
					el: '.swiper-pagination',
			   〜 省略 〜 
				}
			});
		</script>
	</body> 
</html>
```

DOMがロードされてから読み込むようにしたいので、
前にコードが必要となる
https://qiita.com/bakatono_super/items/fcbc828b21599568a597
上記参照

JQueryを使用してもいいが、JQueryに依存しないのがswiperの利点なので、
JSのコードを追加する。

```js
document.addEventListener('DOMContentLoaded', function() {
	内容
});
```

https://www.edureka.co/community/65872/what-is-the-document-ready-equivalent-without-jquery
上記参照


----
https://swiperjs.com/get-started
https://qiita.com/miketa_webprgr/items/0a3845aeb5da2ed75f82
https://osamudaira.com/411/
https://web-parts-box.com/parts/swiper-02/