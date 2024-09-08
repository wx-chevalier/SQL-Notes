## Right Join

***Filtering a cross join to include rows which match a condition. Every row from the right table will be included at least once in the output even if it does not match the condition. In the non-matching case, nulls will be included for the left column values.***

A right join operation is exactly the same as a left join except the right column values are always included and left column values are null when there is no match.

See LEFT_JOIN for a detailed description of how this works.

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

Note that the department table has a value (Engineering) which is not referenced by any `employee.department_id`.

#### PostgreSQL example

In PostgreSQL a RIGHT JOIN on these tables looks like:

```sql
SELECT * FROM employee RIGHT JOIN department on employee.department_id = department.id

+--------+--------+-----------------+------+-------------+
| id     | name   | department_id   | id   | name        |
|--------+--------+-----------------+------+-------------|
| 1      | Josh   | 1               | 1    | Sales       |
| 2      | Ruth   | 2               | 2    | Marketing   |
| <null> | <null> | <null>          | 3    | Engineering |
+--------+--------+-----------------+------+-------------+
```

Since there is no match for the condition on `Engineering` the output includes a row with null values from the `employee` table for that row.

#### SQLToy example

This same call in our database looks like this:

```javascript
employee = FROM('employee');
department = FROM('department');
result = RIGHT_JOIN( employee, department, (c) => c["employee.department_id"] === c["department.id"] );
table(result);
```

The results are:

```
┌───────────────┬─────────────────┬─────────────┬───────────────┬────────────────────────┐
│ department.id │ department.name │ employee.id │ employee.name │ employee.department_id │
├───────────────┼─────────────────┼─────────────┼───────────────┼────────────────────────┤
│       1       │      Sales      │      1      │     Josh      │           1            │
│       2       │    Marketing    │      2      │     Ruth      │           2            │
│       3       │   Engineering   │    null     │     null      │          null          │
└───────────────┴─────────────────┴─────────────┴───────────────┴────────────────────────┘
```

The columns are differently ordered from PostgreSQL but the result is the same.

## RIGHT_JOIN Implementation

`RIGHT_JOIN(a,b, pred)`: right join takes two tables and a predicate. Result will be a table which includes the cross
join of all rows which satisfy the predicate and which has no nulls in table b.

```javascript
const RIGHT_JOIN = (a, b, pred) => {
  return LEFT_JOIN(b, a, pred);
};
```

The implementation just calls LEFT_JOIN with the tables reversed.

***

### Next Section: [Join Tables](https://github.com/weinberg/SQLToy/wiki/Join-Tables)

