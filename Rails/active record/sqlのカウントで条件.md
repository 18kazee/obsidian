
HAVINGを使用する。
```sql
SELECT category.category_id, category.name, COUNT(film.film_id) AS number_of_film FROM category LEFT JOIN film_category ON category.category_id = film_category.category_id LEFT JOIN film ON film_category.film_id = film.film_id GROUP BY category.category_id, category.name HAVING(number_of_film >= 60) ORDER BY number_of_film desc;
```

HAVINGはGROUP BYでまとめた後に条件を抽出したい場合に使用する
WHEREは逆でGROUP BYを使用する前に条件を抽出する。

https://dev.classmethod.jp/articles/difference-where-and-having/
https://sql-oracle.com/sqlserver/?p=424