# Cross Join

***A cross product (aka cartesian product) of the two tables without any filtering condition.***

The first Join to consider in our database is a cross join because it is the starting point for all other joins. What is a cross join? It's every possible combination of rows from the first table with rows from the second table. 

For example if two tables look like:

```
employee
+------+--------+-----------------+
| id   | name   | department_id   |
|------+--------+-----------------|
| 1    | Josh   | 1               |
| 2    | Ruth   | 2               |
| 3    | Greg   | 5               |
+------+--------+-----------------+

department
+------+-------------+
| id   | name        |
|------+-------------|
| 1    | Sales       |
| 2    | Marketing   |
| 3    | Engineering |
+------+-------------+
```

Then the cross join of these two tables would look like:

```
+------+--------+-----------------+------+-------------+
| id   | name   | department_id   | id   | name        |
|------+--------+-----------------+------+-------------|
| 1    | Josh   | 1               | 1    | Sales       |
| 1    | Josh   | 1               | 2    | Marketing   |
| 1    | Josh   | 1               | 3    | Engineering |
| 2    | Ruth   | 2               | 1    | Sales       |
| 2    | Ruth   | 2               | 2    | Marketing   |
| 2    | Ruth   | 2               | 3    | Engineering |
| 3    | Greg   | 5               | 1    | Sales       |
| 3    | Greg   | 5               | 2    | Marketing   |
| 3    | Greg   | 5               | 3    | Engineering |
+------+--------+-----------------+------+-------------+
```

Every entry in employee is paired with every entry in department without any concern for semantic meaning. Note there are rows which associate Josh with the Marketing department and Ruth with the Sales department even though they aren't "in" those departments. This is because a cross join just provides every possible combination. This is in contrast to all other joins which accept a condition and will only return rows which "make sense" according to that condition. More on that later!

For now just remember that a cross join has no filter and therefore returns all possible combinations of rows and is the starting point for other joins which do have a filter.

For a good discussion of this see https://medium.com/basecs/set-theory-the-method-to-database-madness-5ec4b4f05d79 .

## CROSS_JOIN Implementation

`CROSS_JOIN(a,b)` takes two tables and returns a table which includes a cross join of all rows

```javascript
const CROSS_JOIN = (a, b) => {
  const result = {
    name: '',
    rows: []
  }
  for (const x of a.rows) { // loop over rows in table a
    for (const y of b.rows) { // loop over rows in table b
      const row = {};

      // for each column in a, create a column in the output
      for (const k in x) {
        const columnName = a.name ? `${a.name}.${k}` : k;
        row[columnName] = x[k];
      }

      // for each column in b, create a column in the output
      for (const k in y) {
        const columnName = b.name ? `${b.name}.${k}` : k;
        row[columnName] = y[k];
      }

      // Store an array of the two rows used to make up this new row.
      // This is used in LEFT_JOIN and RIGHT_JOIN.
      row._tableRows = [x, y];

      result.rows.push(row);
    }
  }
  return result;
};
```

Since in our database all of our operations take in tables and generate new tables as output we start by creating the empty output table `result`.

The implementation to populate this result table is simply a nested loop. We loop over the first table's rows and for every row we then loop over every row in the second table. We then add all the columns from each table into a new row which gets pushed onto the output table. For column names in the new table we concatenate the table name and the column name like `table.column` so that we can refer to these columns when filtering.

***

### Next Section: [INNER_JOIN](https://github.com/weinberg/SQLToy/wiki/INNER_JOIN)
