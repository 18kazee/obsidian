
redirect_backを使うと、`直前のページにリダイレクト`をしてくれる。

```
redirect_back fallback_location: 直前のページに戻れなかった際のパス

#使用例
redirect_back fallback_location: root_path
```

redirect_backを使う際は、直前のページに戻れなかった際のパスの記述が必要。