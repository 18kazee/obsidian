
```
wikipedia_client = Wikipedia::Client.new(Wikipedia::Configuration.new(domain: 'ja.wikipedia.org'))
page = wikipedia_client.find('ドラマーの一覧')
array = page.text.split("\n").compact_blank
```

あ行
```
array.slice(3, 164)
```
か行
```
array.slice(168, 114)
```
さ行
```
array.slice(255, 164)
```
た行
```
array.slice(419, 120)
```
ナ行
```
array.slice(540, 46)
```
は行
```
array.slice(587, 119)
```
マ行
```
array.slice(707, 76)
```
や行
```
array.slice(784, 44)
```
ラ行
```
array.slice(829, 31)
```
ワ行
```
array.slice(861, 5)
```