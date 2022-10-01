# Notes that I took from [Fundamentals of Database Engineering Course](https://www.udemy.com/course/database-engines-crash-course/)

- ### Notes about Indexing in DB
	- An index is a data structure that you build and assign on top of an existing table that basically looks through your table and then tries to analyze it and summarizes it so that it can create kind of shortcuts.
	- In the databases, the index is in another place (different data structure) than the table itself.
	- `EXPLAIN ANALYZE` query in RDBMS is so useful for analyzing the performance of your query.
	- we cannot apply an index of expression, ex => `like` keyword
	- Big sized indexes slow your write queries, It will not improve the performance.
	- Having an index doesn't mean that your DB will always use it
	- When creating an index you can add with it another column of data this is so powerful
	- [difference between key column and non key one](https://stackoverflow.com/questions/4302165/difference-between-key-column-and-non-key-one)
	- [Bitmap Indexing in DBMS](https://www.geeksforgeeks.org/bitmap-indexing-in-dbms/)
		- Building a bitmap is taking time, so you can escape building it by limiting your query by `limit` keyword 
	- [PostgreSQL indexing: index scan vs. bitmap scan vs. sequential scan (basics)](https://www.cybertec-postgresql.com/en/postgresql-indexing-index-scan-vs-bitmap-scan-vs-sequential-scan-basics/)
		- **`Index only scan` is the best scan you can get** This happens when you search using the index and get the data that is already in the index itself, like making index in “name” and searching on “name”
		- remember you can add an index with other columns by non-key columns 
		- But pay attention to your index size, because the more index size you have is the more your query going to be bad
	- ## Composite index
		- A composite index is an index on multiple columns. (MySQL allows you to create a composite index that consists of up to 16 columns).
		- also known as a multiple-column index.
		- The query optimizer uses the composite indexes for queries that test all columns in the index, or queries that test the first columns, the first two columns, and so on.
		- If you specify the columns in the right order in the index definition, a single composite index can speed up these kinds of queries on the same table.
		- You can't write in the database while creating the index, `pgsql` solves this by creating index concurrently




