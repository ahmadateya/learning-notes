# Notes that I took from [Fundamentals of Database Engineering Course](https://www.udemy.com/course/database-engines-crash-course/)

- **Exclusive locks(write locks) vs Shared locks (read locks)**
	- [What's the difference between an exclusive lock and a shared lock?](https://stackoverflow.com/questions/11837428/whats-the-difference-between-an-exclusive-lock-and-a-shared-lock)
	- [Difference between Shared Lock and Exclusive Lock - Article](https://www.geeksforgeeks.org/difference-between-shared-lock-and-exclusive-lock/)
- **2 phase locks** are very important to solve the double booking problem, Does MySQL have this feature? 
- Does deadlocks apply to the value itself, not the table? Does the resource in the term definitions is the value?
- Always try to avoid caches
	- it's hard to manage (cache invalidation)
	- make it the last performance improvement you want

- ### DB Replication
	- One master / multiple standby(follower) design
	- Multiple masters / multiple followers (complex implementation)
	- <img src="https://github.com/ahmadateya/learning-notes/blob/main/images/DBEngineering-image11.png" width="550" height="300">
	- Pagination without offsets by passing the id itself makes a huge improvement to performance
	- Connection pooling is good for decreasing the I/O headache

 
