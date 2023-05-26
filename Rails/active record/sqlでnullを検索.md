
WHERE文でNULLが入ったデータを検索したい場合はis NULLと入れて検索する

```
SELECT * FROM hoge WHERE return_date is NULL;
```

---
以下のように =NULLとしてもエラーとなるので注意
```
SELECT * FROM hoge WHERE return_date = NULL;
```

https://www.shift-the-oracle.com/sql/is-null.html