# Left Join

***Filtering a cross join to include rows which match a condition. Every row from the left table will be included at least once in the output even if it does not match the condition. In the non-matching case, nulls will be included for the right column values.***

A left join operation is similar to an inner join in that it filters the cross join with a condition. The difference is that a left join will always return at least one row for every row in the left table even if it does not match the condition.

For example, using the following tables:

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

Note that the employee table has an entry (Greg) with a department_id that does not exist in the department table.

##### Left join starts with a cross join

Remember that all joins start with a cross join (logically speaking, in a real database this is not literally true for performance reasons).

Here is the cross join of the above two tables (with full column names):

```
+---------------+-----------------+--------------------------+-----------------+-------------------+
| employee.id   | employee.name   | employee.department_id   | department.id   | department.name   |
|---------------+-----------------+--------------------------+-----------------+-------------------|
| 1             | Josh            | 1                        | 1               | Sales             |
| 1             | Josh            | 1                        | 2               | Marketing         |
| 1             | Josh            | 1                        | 3               | Engineering       |
| 2             | Ruth            | 2                        | 1               | Sales             |
| 2             | Ruth            | 2                        | 2               | Marketing         |
| 2             | Ruth            | 2                        | 3               | Engineering       |
| 3             | Greg            | 5                        | 1               | Sales             |
| 3             | Greg            | 5                        | 2               | Marketing         |
| 3             | Greg            | 5                        | 3               | Engineering       |
+---------------+-----------------+--------------------------+-----------------+-------------------+
```

If we were to do an inner join now using the condition `employee.department_id = department.id` then only two rows would come back (see INNER JOIN). The left join however will include every row from the left table (employee) at least once with nulls for the right table if nothing can be found to match.

#### PostgreSQL example

In PostgreSQL a LEFT JOIN on these tables looks like:

```sql
select * from employee left join department on employee.department_id = department.id
```

The `ON` clause is used to specify that we only want to see the records from the cross join where the employee's department id matches the department id.

The result is:
```
+------+--------+-----------------+--------+-----------+
| id   | name   | department_id   | id     | name      |
|------+--------+-----------------+--------+-----------|
| 1    | Josh   | 1               | 1      | Sales     |
| 2    | Ruth   | 2               | 2      | Marketing |
| 3    | Greg   | 5               | <null> | <null>    |
+------+--------+-----------------+--------+-----------+
```

In the inner join, Greg is shown in this output even though the condition did not match any rows from the cross join. `<null>` is shown for all columns in the right column in this case.

#### SQLToy example

This same call in our database looks like this:

```javascript
employee = FROM('employee');
department = FROM('department');
result = LEFT_JOIN( employee, department, (c) => c["employee.department_id"] === c["department.id"] );
table(result);
```

We load the two tables into variables and pass them to LEFT_JOIN which applies CROSS_JOIN and then filters the result by the condition. The condition is a function which takes a single row of the cross joined tables and returns true if the column `employee.department_id` is equal to `department.id`.

The result is:

```
┌─────────────┬───────────────┬────────────────────────┬───────────────┬─────────────────┐
│ employee.id │ employee.name │ employee.department_id │ department.id │ department.name │
├─────────────┼───────────────┼────────────────────────┼───────────────┼─────────────────┤
│      1      │     Josh      │           1            │       1       │      Sales      │
│      2      │     Ruth      │           2            │       2       │    Marketing    │
│      3      │     Greg      │           5            │     null      │      null       │
└─────────────┴───────────────┴────────────────────────┴───────────────┴─────────────────┘
```

## LEFT_JOIN Implementation

`LEFT_JOIN(a,b, pred)`: leftJoin takes two tables and a predicate. Result will be a table which includes the cross join of all rows which satisfy the predicate or are from table a (with table b columns set to null).

```javascript
const LEFT_JOIN = (a, b, pred) => {
  // Start by taking the cross join of a,b and creating a result table.
  const cp = CROSS_JOIN(a, b);
  let result = {
    name: '',
    rows: [],
  }

  // for each row in a either return matching rows from the cross product
  // or a row with nulls for b if there are no matches
  for (let aRow of a.rows) {
    // find all rows in cross product which come from this row in table a using the _tableRows array
    const cpa = cp.rows.filter((cpr) => cpr._tableRows.includes(aRow));

    // apply the filter
    const match = cpa.filter(pred);

    if (match.length) {
      // we found at least one match so add to result rows
      result.rows.push(...match);
    } else {
      // we did not find a match so create a row with values from a and nulls for b

      let aValues = {};
      let bValues = {};

      // values from a
      for (const key in aRow) {
        aValues[`${a.name}.${key}`] = aRow[key];
      }

      // nulls for b
      for (const key in b.rows[0]) {
        bValues[`${b.name}.${key}`] = null;
      }

      result.rows.push({...aValues, ...bValues});
    }
  }

  return result;
};
```

We start by getting the cross join of `a` and `b`. Then for each row of `a` we determine if any rows in the cross join match the predicate. If they do then add those rows to the output. If not then construct a new row with values from `a` and `null` for all the columns in `b`.

We make use of our helper array `_tableRows` which is stored on the cross join. This makes it easier for us to process the cross join in chunks so we can get a count of matching rows for each `a`.

***

### Next Section: [RIGHT_JOIN](https://github.com/weinberg/SQLToy/wiki/RIGHT_JOIN)
