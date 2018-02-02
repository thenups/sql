### Question 1

__1a.__ Display the first and last names of all actors from the table actor.
```sql
SELECT first_name,last_name
FROM actor;
```

__1b.__ Display the first and last name of each actor in a single column in upper case letters. Name the column Actor Name.
```sql
SELECT CONCAT(first_name,' ',last_name)
AS `Actor Name`
FROM actor;
```

### Question 2

__2a.__ You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?
```sql
SELECT actor_id, first_name, last_name
FROM actor
WHERE first_name LIKE 'Joe';
```

__2b.__ Find all actors whose last name contain the letters GEN:
```sql
SELECT *
FROM actor
WHERE last_name LIKE '%GEN%';
```

__2c.__ Find all actors whose last names contain the letters LI. This time, order the rows by last name and first name, in that order:
```sql
SELECT *
FROM actor
WHERE last_name LIKE '%LI%'
ORDER BY last_name, first_name;
```

__2d.__ Using IN, display the country_id and country columns of the following countries: Afghanistan, Bangladesh, and China:
```sql
SELECT country_id, country
FROM country
WHERE country IN ('Afghanistan', 'Bangladesh', 'China');
```

### Question 3

__3a.__ Add a middle_name column to the table actor. Position it between first_name and last_name. Hint: you will need to specify the data type.
```sql
ALTER TABLE actor
ADD middle_name VARCHAR(50) AFTER first_name;
```

__3b.__ You realize that some of these actors have tremendously long last names. Change the data type of the middle_name column to blobs.
```sql
ALTER TABLE actor
MODIFY COLUMN middle_name BLOB;
```

__3c.__ Now delete the middle_name column.
```sql
ALTER TABLE actor
DROP COLUMN middle_name;
```

### Question 4

__4a.__ List the last names of actors, as well as how many actors have that last name.
```sql
SELECT last_name, COUNT(last_name) AS amount
FROM actor
GROUP BY last_name;
```

__4b.__ List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors
```sql
SELECT last_name, COUNT(last_name) AS amount
FROM actor
GROUP BY last_name
HAVING COUNT(last_name) > 1;
```

__4c.__ Oh, no! The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.
```sql
UPDATE actor
SET first_name='HARPO'
WHERE first_name='GROUCHO' AND last_name = 'WILLIAMS';
```

__4d.__ Perhaps we were too hasty in changing GROUCHO to HARPO. It turns out that GROUCHO was the correct name after all! In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO. Otherwise, change the first name to MUCHO GROUCHO, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO MUCHO GROUCHO, HOWEVER! (Hint: update the record using a unique identifier.)
```sql
UPDATE actor
SET first_name = CASE
	WHEN first_name = 'HARPO' THEN 'GROUCHO'
    ELSE 'MUCHO GROUCHO'
    END
WHERE actor_id =
	(SELECT actor_id
     WHERE (first_name = 'HARPO' OR first_name = 'GROUCHO') AND last_name = 'WILLIAMS'
	)
;

-- Check work from above
SELECT *
FROM actor
WHERE actor_id =
	(SELECT actor_id
     WHERE (first_name = 'HARPO' OR first_name = 'GROUCHO' OR first_name = 'MUCHO GROUCHO') AND last_name = 'WILLIAMS'
	);
```

### Question 5

__5a.__ You cannot locate the schema of the address table. Which query would you use to re-create it?
```sql
DESCRIBE address;
-- OR
SHOW CREATE TABLE address;
```

### Question 6

__6a.__ Use JOIN to display the first and last names, as well as the address, of each staff member. Use the tables staff and address:
```sql
SELECT staff.staff_id, staff.first_name, staff.last_name, address.address
FROM staff
INNER JOIN address ON staff.address_id=address.address_id;
```

__6b.__ Use JOIN to display the total amount rung up by each staff member in August of 2005. Use tables staff and payment.
```sql
SELECT staff.staff_id, staff.first_name, staff.last_name, SUM(payment.amount) AS amount
FROM payment
INNER JOIN staff ON staff.staff_id = payment.staff_id
GROUP BY staff_id;
```

__6c.__ List each film and the number of actors who are listed for that film. Use tables film_actor and film. Use inner join.
```sql
SELECT film.film_id, film.title, COUNT(film_actor.actor_id) AS actor_count
FROM film
INNER JOIN film_actor ON film.film_id = film_actor.film_id
GROUP BY film_id;
```

__6d.__ How many copies of the film Hunchback Impossible exist in the inventory system?
```sql
SELECT film.film_id, film.title, COUNT(film.film_id) AS film_count
FROM film
INNER JOIN inventory ON film.film_id = inventory.film_id
GROUP BY film_id
HAVING title = 'Hunchback Impossible';
```

__6e.__ Using the tables payment and customer and the JOIN command, list the total paid by each customer. List the customers alphabetically by last name:
```sql
SELECT customer.customer_id, customer.first_name, customer.last_name, SUM(payment.amount) AS total_amount
FROM customer
INNER JOIN payment ON customer.customer_id = payment.customer_id
GROUP BY customer_id
ORDER BY last_name, first_name;
```

### Question 7

__7a.__ The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters K and Q have also soared in popularity. Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.
```sql
SELECT film.film_id, film.title
FROM film
WHERE
	(title LIKE 'K%' OR  title LIKE 'Q%')
  AND
	language_id =
  	(
  		SELECT language_id
  		FROM language
  		WHERE name LIKE 'English'
  	);
```

__7b.__ Use subqueries to display all actors who appear in the film Alone Trip.
```sql
SELECT film_actor.actor_id, actor.first_name, actor.last_name, film.title
FROM film_actor, actor, film
WHERE
		film_actor.actor_id = actor.actor_id
    AND
		film_actor.film_id = film.film_id
	AND
    actor.actor_id IN
		(
			SELECT actor_id
			FROM actor
			WHERE title = 'Alone Trip'
		);
```

__7c.__ You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.
```sql
SELECT country.country, customer.first_name, customer.last_name, customer.email
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
WHERE country.country = 'Canada';
```

__7d.__ Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as famiy films.
```sql
SELECT film.title, category.name
FROM film, film_category, category
WHERE
		film.film_id = film_category.film_id
	AND
		film_category.category_id = category.category_id
	AND
		category.name = 'family';
```

__7e.__ Display the most frequently rented movies in descending order.
```sql
SELECT film.film_id, film.title, COUNT(rental.rental_id) AS rentals
FROM film
INNER JOIN inventory ON inventory.film_id = film.film_id
INNER JOIN rental ON rental.inventory_id = inventory.inventory_id
GROUP BY film.film_id
ORDER BY rentals DESC;
```

__7f.__ Write a query to display how much business, in dollars, each store brought in.
```sql
SELECT customer.store_id, SUM(payment.amount) AS total_amount
FROM payment
INNER JOIN customer ON payment.customer_id = customer.customer_id
GROUP BY store_id;
```

__7g.__ Write a query to display for each store its store ID, city, and country.
```sql
SELECT store.store_id, city.city, country.country
FROM store, address, city, country
WHERE
		store.address_id = address.address_id
	AND
		address.city_id = city.city_id
	AND
		city.country_id = country.country_id
;
```

__7h.__ List the top five genres in gross revenue in descending order. (Hint: you may need to use the following tables: category, film_category, inventory, payment, and rental.)
```sql
SELECT category.name, SUM(payment.amount) AS total_amount
FROM category, film_category, inventory, rental, payment
WHERE
		category.category_id = film_category.category_id
	AND
		film_category.film_id = inventory.film_id
	AND
		inventory.inventory_id = rental.inventory_id
	AND
		rental.rental_id = payment.rental_id
GROUP BY name
ORDER BY total_amount DESC
LIMIT 5;
```

### Question 8

__8a.__ In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.
```sql
CREATE VIEW `top_five_genres_by_gross_rev` AS
	SELECT category.name, SUM(payment.amount) AS total_amount
	FROM category, film_category, inventory, rental, payment
	WHERE
			category.category_id = film_category.category_id
		AND
			film_category.film_id = inventory.film_id
		AND
			inventory.inventory_id = rental.inventory_id
		AND
			rental.rental_id = payment.rental_id
	GROUP BY name
	ORDER BY total_amount DESC
	LIMIT 5;
```

__8b.__ How would you display the view that you created in 8a?
```sql
SELECT *
FROM top_five_genres_by_gross_rev;
```

__8c.__ You find that you no longer need the view top_five_genres. Write a query to delete it.
```sql
DROP VIEW IF EXISTS top_five_genres_by_gross_rev;
```
