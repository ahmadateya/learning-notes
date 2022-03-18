# Notes that I took from [Fundamentals of Database Engineering Course](https://www.udemy.com/course/database-engines-crash-course/)

**Partitioning** is the database process where very large tables are divided into multiple smaller parts. By splitting a large table into smaller, individual tables, queries that access only a fraction of the data can run faster because there is less data to scan. The main of goal of partitioning is to aid in maintenance of large tables and to reduce the overall response time to read and load data for particular SQL operations.
- Horizontal partitioning vs vertical partitioning
	- **Horizontal partitioning** involves putting different rows into different tables.
	- **Vertical partitioning** involves creating tables with fewer columns and using additional tables to store the remaining columns. usually used with BLOBs or the huge size columns.
	- Partitioning types
		- <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/DBEngineering-image1.jpg" width="550" height="300">
	- Sharding vs Partitioning
		- sharding implies the data is spread across multiple computers while partitioning does not. **(you decide where to go)**
		- Partitioning is about grouping subsets of data within a single database instance. **(DB decides where to go)**
	- Search on creating partitioned tables (the syntax and method), It includes creating attach keywords
	- The course has a script for creating partitions automatically
	- DO NOT do sharding before you try everything else, It’s the last thing you want to try.




 
