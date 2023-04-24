
### JSとRubyの根本的な違い

JS・・・ブラウザで動く(Node.jsなど、サーバで動くものもあります)。  
Ruby・・・サーバサイドで動く。

よって、当然ですがJSに関するエラーログなどはサーバ側で動いているわけではないため、　rails s　したコンソールには出ません。

### JSとRubyの役割

JS・・・DOM(後述)をいじって画面を装飾する(ブラウザで動作します)。  
Ruby・・・ブラウザなどクライアントからのリクエストを元に、クライアントが求めている情報を提供する(サーバで動作します)。

### DOMとは

Document Object Modelの略です。

端的にいうと**JSからHTMLの要素にアクセスするための仕組み**です。  
DOMはこんなツリー構造になってます。

![[pic_htmltree.gif]]

----
>参考
[JavaScript HTML DOM](https://www.w3schools.com/js/js_htmldom.asp)
----

DOMという仕組みのおかげで、JSからHTML内の要素にアクセスできます。  
具体的には「見出しのテキストを変える」や「リンクのテキストを変える」などができます。  
DOMを操作して画面をちょこちょこ変えるのが(レガシー)JSの役割です。


### DOMを操作するには　id　や　class　が必要

例えば、こんなHTMLがあったとします。

```html
<button>削除ボタン</button>
<button>更新ボタン</button>
```

人間からすると「削除ボタン」やら「更新ボタン」と書いてあるので簡単に識別できるのですが、コンピュータは表示名の違う2つのボタンとしか識別できません。
そこで登場するのが **id** や **class** です。

```html
<button id="button-delete">削除ボタン</button>
<button id="button-update">更新ボタン</button>
```

このようにHTMLの要素に何かしらの識別子をつけてあげることで、コンピュータからも簡単に識別できるようになります。

### イベントドリブンとは

コンソールにJSを打ち込めばDOMの取得や操作ができることが確認できますが、実際は何かのイベントをフックにDOMの操作を行うことが普通でしょう。  
それを「イベントドリブン」といったりします。

端的にいうと**何かをしたら何かが起きる**という処理のことです。イベント駆動ともいいます。

例

-   あるボタンをクリックしたらポップアップを出す
-   マウスポインターがある文字の上に乗ったらその文字を赤くする
-   あるテキストエリアの内容が変更されたらそれらの文字をプレビュー表示する

ブラウザはユーザーが操作するものなので「何かをしたら何かが起こる」というイベントドリブン的な考え方が非常にマッチしてますね。

### イベントをセットしてみる

``` js
document.addEventListener('DOMContentLoaded', () => {
    console.log("DOMContentLoaded")

    // 削除ボタンのDOMを取得
    const buttonDelete = document.getElementById("button-delete")
    // 更新ボタンのDOMを取得
    const buttonUpdate = document.getElementById("button-update")
    // 削除ボタンにクリックイベントを仕掛ける
    buttonDelete.addEventListener('click', () => {
      // クリックされた時に動く処理
      alert("削除！")
    })
    // 更新ボタンにクリックイベントを仕掛ける
    buttonUpdate.addEventListener('click', () => {
      // クリックされた時に動く処理
      alert("更新！")
    })
});
```

DOMに対して **addEventListener** を設定することでイベントを仕掛けることができます。  
第一引数にはイベントの種類（クリック時なのかホバー時なのかetc）を指定します。

ちなみに一番外側の **document.addEventListener('DOMContentLoaded', () => {})** も一種のイベント設定処理です。  
「ブラウザがDOMの解析をし終わったら」のようなイメージのイベントです。  
この記述がないとDOMの解析が終わる前に、削除ボタンのDOMを取得をしようとしたりしてうまく動かないことがあります。

### JSができることは

-   イベントをセットする
-   DOMを操作する

極論この二つを知っておけば大体のことができます。


##### アラートボタンを押すとテキストボックスの内容がポップアップする
```html
<textarea id="textarea"></textarea>
<button id="button">アラート</button>
```

```js
const buttonAlert = document.getElementById("button")
const textarea = document.getElementById("textarea")
buttonAlert.addEventListener("click",() => {
  alert(textarea.value)
})
```

##### 数値を足したり引いたりする

```html
<label for="amount">数値</label>
<input type="number" name="amount" id="js-amount">
<button id="js-add-button">増やす</button>
<button id="js-minus-button">減らす</button>
<p>結果</p>
<p id="js-result">0</p>
```


```ruby
document.addEventListener('DOMContentLoaded', () => {
  console.log("DOMContentLoaded")
  
  const jsAmount = document.getElementById("js-amount")
  const jsAddButton = document.getElementById("js-add-button")
  const jsMinusButton = document.getElementById("js-minus-button")
  const jsResult = document.getElementById("js-result")
  var currentValue = 0
  
  jsAddButton.addEventListener('click', () => {
    currentValue = currentValue + parseInt(jsAmount.value)
    jsResult.innerText = currentValue
  })
  
  jsMinusButton.addEventListener('click', () => {
    currentValue = currentValue - parseInt(jsAmount.value)
    jsResult.innerText = currentValue
  })
})
```


