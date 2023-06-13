
```
SELECT COUNT(*) FROM Film INNER JOIN Inventory ON film.film_id = inventory.film_id WHERE title = 'Hoge';
```

上記のようなSQL文だと以下がActiveRecord文になる

```
Film.Joins(:inventory).where(title: 'Hoge').count
```

joinsはINNER JOINを表している

https://railsdoc.com/page/joins