
javascriptの記載は、application.jsには記載せず、別の専用ファイルを作成する。
ここでは、application.jsは個別のJavaScriptを読み込む専用のファイルのため、preview.jsファイルを作成し記載する。

```js
//preiew.js
function previewFileWithId(id) {
    const target = this.event.target;
    const file = target.files[0];
    const reader  = new FileReader();
    reader.onloadend = function () {
        preview.src = reader.result;
    }
    if (file) {
        reader.readAsDataURL(file);
    } else {
        preview.src = '';
    }
}
```

### 1

```js
const target = this.event.target;
```

traget・・・操作を差し込む対象のオブジェクト  
event.target・・・最初にイベントが起こった要素です。今回だと、ファイルを選択した時のイベントを示します。

### 2

```js
const file = target.files[0];
```

targetには「files」というプロパティが用意されています。  
管理されているファイルは、「File」というオブジェクトの形をしています。  
このFileオブジェクトには、ファイルに関する各種の情報や、ファイルアクセスのためのメソッドなどがまとめられているのです。
files[0]で一つ目のFileオブジェクトを取り出しています。

### 3

```js
const reader  = new FileReader();
```

FileReaderオブジェクトで、ファイルを取得します。

### 4

```js
reader.onloadend = function () {
        preview.src = reader.result;
    }
```

onloadend、FileReaderのイベントです。データの読み込みが成功か失敗に関わらず終了した時にloadendイベントが発生し、
ここに設定したコールバック関数が呼び出されます。
そして、取得されたファイルの結果を、previewのsrc属性に指定しています。

### 5

```js
if (file) {
        reader.readAsDataURL(file);
    } else {
        preview.src = '';
    }
```

readAsDataURL()は、FileReaderのメソッドです。ファイルを、Data URIとして読み込むメソッドです。
例えば画像ファイルをこのメソッドで読み込んで、読み込んだデータをimg要素のsrc属性に指定すればブラウザに表示できます。
前文でscr属性を指定したので、ここで、画像を表示させます。





