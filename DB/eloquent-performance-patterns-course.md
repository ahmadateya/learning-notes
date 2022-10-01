# Notes that I took from [Eloquent Performance Patterns](https://eloquent-course.reinink.ca/)


- Minimize the memory usage by selecting only what you need
- If you need one column from a `hasMany` relationship don't use eager loading it will bring to you all the records which are bad.
- Instead use the subquery => `addSelect()` Note that the subquery
- The subquery returns only one record
- Almost whenever you run operations in line with your queries the DB will not be able to use indexes, unless you have an index for this expression
- You can add a virtual column in your laravel migrations by this method `virtualAs()`
- The mini-series of search is very helpful (7 - 11)
    - The indexes are not working with the wildcard at the beginning where like statement ex => ‘%bill’




