
   メソッドの１つで、[[バリデーション]]を設定する時に使用します。
 
```ruby
validates :カラム名, オプション引数
```

### 例

```ruby
validates :name, presence: true, length: {maximum: 255}
```

`presence: true`
中身が空でないこと

`length: {maximum: 10}`
長さを[[バリデーション]]、10文字まで


>参照
>https://railsdoc.com/page/validates