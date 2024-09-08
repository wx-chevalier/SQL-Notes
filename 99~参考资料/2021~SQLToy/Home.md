# SQLToy

A great way to learn how something works is to build it yourself. In this case you will learn how a SQL database works by building one!

## What is SQL?

SQL stands for Structured Query Language. It is the standard interface of most relational databases.

When you want to put some data in or get some data out of a SQL Database you write a Query. Queries are written in a kind of quasi-English language which lets you tell the database what you want (but leaves the how-to-do-it up to the database).

Here’s an example:

```sql
SELECT name FROM employee;
```

This query will return the names of all the employees in a database table called `employee`.

## What are we going to implement?

A very simple SQL database which in fact has no parser and therefore cannot even understand the above query directly. What we will be building are Javascript methods for each of the most common SQL operations like `SELECT` and `JOIN`. Then we will manually call them (in code) in the correct order to execute the query.

For example the above query will look like this:

```javascript
SELECT(employee, ['name']);
```

## What can it do?

As a preview, here’s a more complex query in both PostgreSQL and SQLToy.

##### PostgreSQL
```SQL
SELECT DISTINCT status, charity_group.name, COUNT(*) AS count FROM employee
  JOIN employee_charity_group 
    ON employee_charity_group.a = employee.id
  JOIN charity_group
    ON charity_group.id = employee_charity_group.b
  WHERE employee.salary > 150000
  GROUP BY status, charity_group.name
  ORDER BY count DESC;

+----------+--------------------+---------+
| status   | name               | count   |
|----------+--------------------+---------|
| active   | Cat Lovers         | 2       |
| active   | Environmentalists  | 1       |
| active   | Food for the Needy | 1       |
| active   | House Builders     | 1       |
| inactive | Education for Kids | 1       |
| inactive | Environmentalists  | 1       |
+----------+--------------------+---------+|
```

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

Output:

```
┌─────────────────┬──────────────────────┬───────┐
│ employee.status │  charity_group.name  │ count │
├─────────────────┼──────────────────────┼───────┤
│     active      │      Cat Lovers      │   2   │
│     active      │  Environmentalists   │   1   │
│     active      │  Food for the Needy  │   1   │
│     active      │    House Builders    │   1   │
│    inactive     │  Education for Kids  │   1   │
│    inactive     │  Environmentalists   │   1   │
└─────────────────┴──────────────────────┴───────┘
```

Save for some formatting, the results are the same.


# License

The code in SQLToy is licensed under [The MIT License](https://github.com/weinberg/SQLToy/blob/main/LICENSE).

The contents of the Wiki are licensed separately under the [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License](https://github.com/weinberg/SQLToy/wiki/LICENSE).

***

### Next section: [Data Model](https://github.com/weinberg/SQLToy/wiki/Data-Model)
