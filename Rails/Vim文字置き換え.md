
visualモードで置き換えたい場所を選択後、stackをoverflowに変更したい場合は以下
```
:'<,'>s/stack/overflow/g
```

`'<,'>`はvisualを意味していて、`'<`は最初の行、`'>`は最後の行を意味します。
つまり選択範囲の最初の行から、最後の行までという意味です。

https://stackoverflow.com/questions/7759455/in-vim-visual-mode-why-does-the-command-start-with