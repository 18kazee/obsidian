

```ruby
arg = ARGV[0].to_i

1.upto(arg) do |num|
	if num % 15 == 0
		p 'らんてくん'
	elsif num % 5 == 0
		p 'くん'
	elsif num % 3 == 0
		p 'らんて'
	else
		p num
end
```

- argにrubyのコマンドライン引数で入力された文字をto_iでintegeerに変換して代入
- [[1.uptoメソッド]]を使用して`arg`の数字分ループさせます。ループさせる際`num`に代入します
- もし`num`が15で割り切れる時は`らんてくん`と出力
- もし`num`が5で割り切れる時は`くん`と出力
- もし`num`が3で割り切れる時は`らんて`と出力
- それ以外の時は`num`の値をそのまま出力
