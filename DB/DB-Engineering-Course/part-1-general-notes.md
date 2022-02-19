# Notes that I took from [Fundamentals of Database Engineering Course](https://www.udemy.com/course/database-engines-crash-course/)

- ### Generel Notes
	- `Select *` => is a bad query
	- `Select where like` is a bad query
	- To seed a database with `pgsql` in postgres with 1 million records at once you could use `generate_series()` function
	- **Bloom Filters** is so good for filters and it mostly used in in-memory databases (especially Cassandra)
		- It works by making a table (hashed table) and checking it before hitting the DB to decrease the times that you hit the DB for querying. 
