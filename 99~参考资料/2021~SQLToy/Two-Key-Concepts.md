# 1. The database does not execute a query in the order it is written

SQL queries are made up of operations. These operations must be written in a specific order to be valid SQL. This is the ***lexical*** order of the query. The ***logical*** order is the order in which the database applies the operations when executing the query. This is a different order.

[[/images/query-vs-execution-order.png|Query vs Execution Order]]

Here is the logical sequence which is followed by a SQL database when processing a SELECT query:

1. FROM
2. JOIN
3. WHERE
4. GROUP BY
5. Aggregate Functions (COUNT, SUM, AVG, etc)
6. HAVING
7. SELECT
8. ORDER BY
9. OFFSET
10. LIMIT

In our database all the SQL operations are represented by different Javascript functions. We don't have a parser (which would convert an actual SQL query string into an execution plan) so we will have to call the functions ourselves in the right order.

Let's look at that query again. Here is the SQL:

##### SQL
```SQL
SELECT DISTINCT status, charity_group.name, COUNT(*) AS count FROM employee
  JOIN employee_charity_group 
    ON employee_charity_group.a = employee.id
  JOIN charity_group
    ON charity_group.id = employee_charity_group.b
  WHERE employee.salary > 150000
  GROUP BY status, charity_group.name
  ORDER BY count DESC;
```

And here is our implementation:

##### SQLToy
```javascript
let result;
result = JOIN(employee, employee_charity_group, (c) => c["employee_charity_group.A"] === c["employee.id"]);
result = JOIN(result, charity_group, (c) => c["employee_charity_group.B"] === c["charity_group.id"] );
result = WHERE(result, (row) => row['employee.salary'] > 150000);
result = GROUP_BY(result, ['employee.status', 'charity_group.name']);
result = COUNT(result, 'charity_group.name');
result = SELECT(result,['employee.status', 'charity_group.name','COUNT(charity_group.name)'],{'COUNT(charity_group.name)': 'count'})
result = DISTINCT(result, ['employee.status', 'charity_group.name', 'count'])
result = ORDER_BY(result, (a,b) => a.count < b.count ? 1 : -1);
```

See how we have re-ordered the operations? This is done for you in a "real" SQL database. In our case we do it manually.

For an excellent article on this topic see here: [A Beginner’s Guide to the True Order of SQL Operations](https://blog.jooq.org/a-beginners-guide-to-the-true-order-of-sql-operations/)

Now for our second Key Concept...

# 2. Logically speaking, SQL operations take tables as input and generate tables as output

This means that a JOIN between two tables can be considered to generate a new table on which the rest of the query's operations will act. A GROUP BY query will generate a new table with the grouped columns as its output. Essentially SQL processing can be thought of as a pipeline which takes tables in and puts tables out.

See how in the SQLToy query above we are taking the result of each operation and passing it in as an argument to the next? This is the table pipeline. Tables in, tables out.

With these two concepts under our belt we can now begin to actually implement our database.

***

### Next section: [FROM](https://github.com/weinberg/SQLToy/wiki/FROM)
