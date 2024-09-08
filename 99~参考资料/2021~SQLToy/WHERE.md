## Where

***Filtering a table by a predicate***

The where clause in a select query is used to reduce the resulting rows by a condition.

For example, using the following table:

```sql
employee
+------+--------+-----------------+
| id   | name   | department_id   |
|------+--------+-----------------|
| 1    | Josh   | 1               |
| 2    | Ruth   | 2               |
| 3    | Greg   | 5               |
| 4    | Pat    | 1               |
+------+--------+-----------------+
```

We can find all employees who have department_id of 1 by using a where clause.

In PostgreSQL:

```sql
SELECT * FROM employee WHERE department_id = 1;

+------+--------+-----------------+
| id   | name   | department_id   |
|------+--------+-----------------|
| 1    | Josh   | 1               |
| 4    | Pat    | 1               |
+------+--------+-----------------+
```

In SQLToy:

```javascript
employee = FROM('employee');
result = WHERE(employee, (c) => c["department_id"] === 1);
table(result);
```
Results:
```
┌────┬────────┬───────────────┐
│ id │  name  │ department_id │
├────┼────────┼───────────────┤
│ 1  │  Josh  │       1       │
│ 4  │  Pat   │       1       │
└────┴────────┴───────────────┘
```

### Where on Joined tables

The where clause is applied after any join operations according to the [SQL order of operations](https://github.com/weinberg/SQLToy/wiki/Two-Key-Concepts). Therefore the where clause can refer to columns from any joined tables. Join will prefix the table name onto the column name and the where clause should use the `table.column` syntax to reference the column.

For example this SQL query:

```sql
SELECT * FROM employee
  JOIN employee_club ON employee_club.a = employee.id
  JOIN club ON club.id = employee_club.b
  WHERE salary > 150000 AND club.name = 'Cat Lovers';
```

Can be executed in SQLToy like:

```javascript
const employee = FROM('employee');
const employee_club = FROM('employee_club');
const club = FROM('club');
result = INNER_JOIN(employee, employee_club, (c) => c["employee_club.a"] === c["employee.id"]);
result = INNER_JOIN(result, club, (c) => c["club.id"] === c["employee_club.b"]);
result = WHERE(result, r => r['employee.salary'] > 150000 && r['club.name'] === 'Cat Lovers');
table(result);
```

Producing the following output:

```
┌─────────────┬───────────────┬─────────────────┬────────────────────────┬─────────────────┬──────────────────────────┬──────────────────────────┬──────────────────┬────────────────────┐
│ employee.id │ employee.name │ employee.salary │ employee.department_id │ employee.status │ employee_club.a          │ employee_club.b          │ club.id          │ club.name          │
├─────────────┼───────────────┼─────────────────┼────────────────────────┼─────────────────┼──────────────────────────┼──────────────────────────┼──────────────────┼────────────────────┤
│      2      │     Jane      │     160000      │           2            │     active      │            2             │            1             │        1         │     Cat Lovers     │
│      4      │    Elliot     │     180000      │           1            │     active      │            4             │            1             │        1         │     Cat Lovers     │
└─────────────┴───────────────┴─────────────────┴────────────────────────┴─────────────────┴──────────────────────────┴──────────────────────────┴──────────────────┴────────────────────┘
```

The where clause references two columns from joined tables: `employee.salary` and `club.name`, using their values to filter the results.

***

Next section: [GROUP_BY](https://github.com/weinberg/SQLToy/wiki/GROUP_BY)
