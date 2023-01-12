ハッシュ同士を結合することができます。  
例えば、paramsに含まれない値をストロングメソッドに加えたい場合などに、ストロングパラメータの後に記述することができます。

```ruby
params.require(:user).permit(:name,:email).merge(user_id: current_user.id)
```

>参照
>https://qiita.com/ozackiee/items/f100fd51f4839b3fdca8
