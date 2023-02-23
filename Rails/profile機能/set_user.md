
プロフィール編集対象のユーザー情報（@user）をcurrent_userのインスタンスではなく、DBから取得したオブジェクトを利用していること

app/controllers/profiles_controller.rb

```
# BAD
def set_user
  @user = current_user
end

# GOOD
def set_user
  @user = User.find(current_user.id)
end
```

BAD例で実装すると、プロフィール名の変更に失敗した際に、画面上でプロフィールとして表示される名前が変わってしまいます。  
これはプロフィール編集の失敗時にcurrent_userがupdateの影響を受けてしまうためです。

一度@userに格納しているのに、どうしてcurrent_userの中身が変わってしまうのかと疑問に思う方もいるかもしれません。それはrubyでの変数の受け渡しが、参照渡しであることが原因です。  
[参照渡しとは](https://magazine.rubyist.net/articles/0032/0032-CallByValueAndCallByReference.html)
