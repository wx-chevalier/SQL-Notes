## ORDER_BY

***Re-order the rows in a table via a compare function***

`ORDER BY` is a simple operation which orders the rows in a table by some criteria. In SQL we can `ORDER BY` a column and the type of the column is used to determine how to sort:

```SQL
SELECT * FROM employee
+------+--------+-----------------+---------+
| id   | name   | department_id   | role    |
|------+--------+-----------------+---------|
| 3    | Jake   | 1               | Worker  |
| 4    | John   | 2               | Worker  |
| 1    | Josh   | 1               | Worker  |
| 5    | Alice  | 2               | Manager |
| 6    | Dan    | 1               | Manager |
| 2    | Ruth   | 2               | Worker  |
| 7    | Janet  | 1               | Manager |
+------+--------+-----------------+---------+

SELECT * FROM employee ORDER BY name ASC;
+------+--------+-----------------+---------+
| id   | name   | department_id   | role    |
|------+--------+-----------------+---------|
| 5    | Alice  | 2               | Manager |
| 6    | Dan    | 1               | Manager |
| 3    | Jake   | 1               | Worker  |
| 7    | Janet  | 1               | Manager |
| 4    | John   | 2               | Worker  |
| 1    | Josh   | 1               | Worder  |
| 2    | Ruth   | 2               | Worker  |
+------+--------+-----------------+---------+
```

Note that all the rows and columns of the table are in the output but the order of the rows has changed. SQL allows the use of an `ASC` or `DESC` modifier to change the direction of the sort to ascending or descending.

In SQLToy we pass a [compare function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) into the `ORDER_BY()` call which is used to do the comparison. There is no separate parameter for ascending or descending sort, this is just done by modifying the compare function appropriately.

```javascript
employee = FROM('employee');
result = ORDER_BY(employee, (a, b) => a['name'].localeCompare(b['name']));
console.log('Order by name:');
table(result);
```

This gives us:

```
┌────┬─────────┬───────────────┬───────────┐
│ id │  name   │ department_id │   role    │
├────┼─────────┼───────────────┼───────────┤
│ 5  │  Alice  │       2       │  Manager  │
│ 6  │   Dan   │       1       │  Manager  │
│ 3  │  Jake   │       1       │  Worker   │
│ 7  │  Janet  │       1       │  Manager  │
│ 4  │  John   │       2       │  Worker   │
│ 1  │  Josh   │       1       │  Worker   │
│ 2  │  Ruth   │       2       │  Worker   │
└────┴─────────┴───────────────┴───────────┘
```

# ORDER_BY Implementation

The implementation could not be simpler as we just make use of Javascript's built-in array `sort` method to sort the rows.

```javascript
const ORDER_BY = (table, rel) => {
  return {
    name: table.name,
    rows: table.rows.sort(rel),
  }
}
```

***

### Next Section: [OFFSET and LIMIT](https://github.com/weinberg/SQLToy/wiki/OFFSET_LIMIT)
