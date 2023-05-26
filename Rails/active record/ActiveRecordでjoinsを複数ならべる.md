
前提
Filmテーブルにhogeテーブルとfugaテーブルをjoinしたい


Filmにhogeもfugaもアソシエーションされている場合
```
Film.joins(:hoges, :fugas)
```

Filmにhogeがアソシエーションされているが、fugaはhogeにアソシエーションされている場合
```
Film.joins(hoges: :fugas)
```

https://note.com/sq_engch5/n/n09210c900799
