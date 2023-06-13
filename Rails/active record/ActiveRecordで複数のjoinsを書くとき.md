
基本的にSQLではINNER JOINやLEFT JOIN使用し、後ろに連結して書いていきます。

```sql
SELECT category.category_id, category.name, sum(payment.amount) AS revenue
FROM category
INNER JOIN film_category on category.category_id = film_category.category_id
INNER JOIN film on film_category.film_id = film.film_id
INNER JOIN inventory on film.film_id = inventory.film_id
INNER JOIN rental on rental.inventory_id = inventory.inventory_id
INNER JOIN payment on rental.rental_id = payment.rental_id
GROUP BY category.category_id, category.name
ORDER BY revenue DESC
LIMIT 5;
```

これをActiveRecordで書くと以下のようになります

```ruby
Category.select("category.category_id, category.name, sum(payment.amount) as revenue").joins(films: { inventories: { rentals: :payments} }}).group("category.category_id, category.name").order(revenue: :desc).limit(5)
```

複数JOINするときは`{ }`の中に入れていきます。