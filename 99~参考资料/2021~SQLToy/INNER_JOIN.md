# Inner Join

***Filtering a cross join by a condition***

An inner join operation returns the join of two tables using a condition to pick rows from the cross join. In SQL this condition is specified with an `ON` clause.

For example, take the following two tables:

```sql
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

Note that the employee table has an entry (Greg) with a department_id that does not exist in the department table. And the department table has a department (Engineering) which is not referenced by any employee.

##### Inner Join starts with a Cross Join

Remember that all joins start with a cross join (logically speaking, in a real database this is not literally true for performance reasons).

Here is the cross join of the above two tables (with full column names):

```sql
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

The inner join takes all rows from the cross join which match the condition.

If we want to know the name of the department each employee is in then we want our condition to select only those rows where the `employee.department_id` column is equal to the `department.id` column. This condition provides meaning to our data by associating the data in the department table with the data in the employee table.

Before you go on, look at the above table and pick out the rows where `employee.department_id` matches the `department.id` column.

#### PostgeSQL example

In PostgreSQL an INNER JOIN on these tables looks like:

```sql
SELECT * FROM employee JOIN department ON department.id = employee.department_id;
```

The `ON` clause is used to specify that we only want to see the records from the cross join where the employee's department id matches the department id.

The result is:
```
+------+--------+-----------------+------+-----------+
| id   | name   | department_id   | id   | name      |
|------+--------+-----------------+------+-----------|
| 1    | Josh   | 1               | 1    | Sales     |
| 2    | Ruth   | 2               | 2    | Marketing |
+------+--------+-----------------+------+-----------+
```


#### SQLToy example

This same call in our database looks like this:

```javascript
employee = FROM('employee');
department = FROM('department');
result = INNER_JOIN( employee, department, (c) => c["employee.department_id"] === c["department.id"] );
table(result);
```

We load the two tables into variables and pass them to INNER_JOIN which applies CROSS_JOIN and then filters the result by the condition. The condition is a function which takes a single row of the cross joined tables and returns true if the column `employee.department_id` is equal to `department.id`.

The result is:

```
┌─────────────┬───────────────┬────────────────────────┬───────────────┬─────────────────┐
│ employee.id │ employee.name │ employee.department_id │ department.id │ department.name │
├─────────────┼───────────────┼────────────────────────┼───────────────┼─────────────────┤
│      1      │     Josh      │           1            │       1       │      Sales      │
│      2      │     Ruth      │           2            │       2       │    Marketing    │
└─────────────┴───────────────┴────────────────────────┴───────────────┴─────────────────┘
```

## INNER_JOIN Implementation

`INNER_JOIN(a,b, pred)` takes two tables and a predicate. The result will be a table which includes the cross join of all rows which satisfy the predicate. The predicate has the signature `(a,b) => boolean`, taking two rows as input and returning true for rows which should be in the resulting table.

```
const INNER_JOIN = (a, b, pred) => {
  return {
    name: '',
    rows: CROSS_JOIN(a, b).rows.filter(pred),
  }
};
```

The implementation returns a new table with rows from the cross join filtered using the standard `Array.prototype.filter()` method.

***

### Next Section: [LEFT_JOIN](https://github.com/weinberg/SQLToy/wiki/LEFT_JOIN)
