# Notes that I took from [Fundamentals of Database Engineering Course](https://www.udemy.com/course/database-engines-crash-course/)

- ### Generel Notes
	- `Select *` => is a bad query
	- `Select where like` is a bad query
	- To seed a database with `pgsql` in postgres with 1 million records at once you could use `generate_series()` function
	- **Bloom Filters** is so good for filters and it mostly used in in-memory databases (especially Cassandra)
		- It works by making a table (hashed table) and checking it before hitting the DB to decrease the times that you hit the DB for querying. 


- ### DB Cursors 
	- Server-side cursors
	- Client-side cursors
	- Server-side vs Client-side


- ### DB Security
	- It’s better to enable SSL with connecting to DB when you work k8s or something like this
	- Try to make a database user for each operation with different permission suitable to each task (read-only users, update users) 
	- Use Connection Pools & separated pools
		- Ex: dbReadPool with only read permissions
		- Ex: dbCreatePool with only create permissions (not update or delete)
		- Ex: dbDeletePool with only delete permissions
		- Same with update 
		- There is no permission for DROP or delete tables
		- With this approach, you will never be have a problem with SQL injection
		- Always limit your queries, don't make unbounded queries (queries without limit or paging ...etc)
		- **Homomorphic Encryption**
			- <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/DBEngineering-image2.png" width="550" height="300">





