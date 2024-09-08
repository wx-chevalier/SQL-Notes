# Group By

***Grouping rows in preparation for aggregate functions***

(If you already know how grouping works you can jump right to the [GROUP_BY Implementation](#GROUP_BY-Implementation))

Group By and Aggregate Functions are used to summarize data. `GROUP BY` takes a table and one or more columns as input. A new table is returned where each individual row represents a group of rows from the original table. Once grouped, aggregate functions can then be used to summarize and report on the data in each group.

For example:

```SQL
SELECT * FROM employee;

+------+--------+-----------------+---------+
| id   | name   | department_id   | role    |
|------+--------+-----------------+---------|
| 1    | Josh   | 1               | Manager |
| 2    | Ruth   | 2               | Worker  |
| 3    | Jake   | 1               | Worker  |
| 4    | John   | 2               | Worker  |
| 5    | Alice  | 2               | Manager |
| 6    | Dan    | 1               | Manager |
| 7    | Janet  | 1               | Manager |
+------+--------+-----------------+---------+
```

This table shows multiple employees in each department. We'd like to know how many are in each. Let's have the database group the rows where the employees are in the same department:

```SQL
SELECT * FROM employee GROUP BY department_id;

+-----------------+
| department_id   |
|-----------------+
| 1               |
| 2               |
+-----------------+
```

##### Group By reduces rows and columns
This `GROUP BY` operation returns a new table with a single row for every different `department_id`. All rows with `department_id: 1` are collapsed down into one row. All rows with `department_id: 2` are collapsed down into another row. So the resulting table has two rows.

In addition to collapsing rows, the columns are reduced as well. The new table only includes the columns which we told `GROUP BY` to... group by. In this case that means there is only one column: `department_id`. Grouping can be done on multiple columns as well. In that case, each row in the resulting table would represent all the rows which have the same values for the given columns. An example of this is shown below.

##### Aggregate Functions

But before we get to that let's talk about Aggregate Functions. What we really want to know in our query is not just that there are two departments but how many employees are in each. Once rows are grouped, however, we can no longer see the contents of the group directly. To "look inside" the group we must use Aggregate Functions.

An Aggregate Function summarizes the data in the group into a single value and it puts that value into a new column.

For example, the `COUNT(*)` operation is an aggregate function which summarizes the group by counting how many rows were collapsed to make it. `COUNT(*)` adds a new column onto the table to report the result of this aggregation and calls it `count`.

How many employees are in each department? Once they are grouped we can find out with `COUNT(*)`:

```SQL
SELECT department_id, COUNT(*) FROM employee GROUP BY department_id;

+-----------------+---------+
| department_id   | count   |
|-----------------+---------|
| 1               | 4       |
| 2               | 3       |
+-----------------+---------+
```

##### Group By in SQLToy

In SQLToy we must reorder the query and call the operations explicitly. It looks like this:

```Javascript
employee = FROM('employee');
result = GROUP_BY(employee, ['department_id']);
result = COUNT(result, '*');
table(result)
```

Result:

```
┌───────────────┬──────────┐
│ department_id │ COUNT(*) │
├───────────────┼──────────┤
│       1       │    4     │
│       2       │    3     │
└───────────────┴──────────┘
```

Instead of just showing the count, say we wanted to see the names of all the employees in each group. We can find this with another aggregate function: `ARRAY_AGG()`. Instead of counting the number of rows which went into the group it will create a column with all the values from a column in the group and show it like an array:

```SQL
SELECT department_id, ARRAY_AGG(name) FROM employee GROUP BY department_id;

+-----------------+----------------------------------+
| department_id   | array_agg                        |
|-----------------+----------------------------------|
| 1               | ['Jake', 'Josh', 'Dan', 'Janet'] |
| 2               | ['John', 'Alice', 'Ruth']        |
+-----------------+----------------------------------+
```

In SQLToy we would do:

```javascript
employee = FROM('employee');
result = GROUP_BY(employee, ['department_id']);
result = ARRAY_AGG(result, 'name');
table(result)
```

Result:

```
┌───────────────┬─────────────────────────────────┐
│ department_id │         ARRAY_AGG(name)         │
├───────────────┼─────────────────────────────────┤
│       1       │  ["Josh","Jake","Dan","Janet"]  │
│       2       │     ["Ruth","John","Alice"]     │
└───────────────┴─────────────────────────────────┘
```

This shows us the values from the name column of each row which went into in the group.

There are many aggregate functions but they all have the same purpose: to summarize data, most often from a `GROUP BY` operation. Likewise the `GROUP BY` operation is used to prepare groups of rows for summarization by aggregate functions.

#### Multiple Group By Columns

As mentioned before, `GROUP BY` can operate on multiple columns at once. It looks at the values in each of the grouping columns when deciding how to group rows together.

For example:

```SQL
SELECT role, COUNT(*) FROM employee GROUP BY role;
+---------+---------+
| role    | count   |
|---------+---------|
| Manager | 4       |
| Worker  | 3       |
+---------+---------+
```

This query is grouping the data by a different column: `role`. In this case we are asking the database to return a new table with a single row for each distinct value of `role`. Then the `COUNT(*)` aggregate function is adding a column with the count of how many rows were condensed into that group. So we end up with a table showing how many Managers and Workers there are in the whole table.

This organization looks a little top heavy! It might be time for a reorganization. But we'll need to know how many managers and workers are in each group and we'll need to know their names. To do this we can `GROUP BY` both the role and the department_id and supply multiple aggregate functions as well like this:

```SQL
SELECT department_id, role, COUNT(*), ARRAY_AGG(name) FROM employee GROUP BY department_id, role;

+-----------------+---------+---------+--------------------------+
| department_id   | role    | count   | array_agg                |
|-----------------+---------+---------+--------------------------|
| 1               | Manager | 3       | ['Josh', 'Dan', 'Janet'] |
| 1               | Worker  | 1       | ['Jake']                 |
| 2               | Manager | 1       | ['Alice']                |
| 2               | Worker  | 2       | ['John', 'Ruth']         |
+-----------------+---------+---------+--------------------------+
```

When `GROUP BY` is given more than one column it groups rows based on the values of all the columns. So rows with department_id: 1 and role: Manager are put into one group. Rows with `department_id: 1` and `role: Worker` go into a separate group.

Now we can see that there are 3 Managers and 1 Worker in department 1. Look out Josh, I think you are about to get reassigned!

In SQLToy this works the same way:

```javascript
employee = FROM('employee');
result = GROUP_BY(employee, ['department_id','role']);
result = COUNT(result, '*');
result = ARRAY_AGG(result, 'name');
result = SELECT(result, ['department_id','role','COUNT(*)','ARRAY_AGG(name)']);
```

Note that we passed both `department_id` and `role` into `GROUP_BY`. With the result:

```
┌───────────────┬───────────┬──────────┬──────────────────────────┐
│ department_id │   role    │ COUNT(*) │     ARRAY_AGG(name)      │
├───────────────┼───────────┼──────────┼──────────────────────────┤
│       1       │  Manager  │    3     │  ["Josh","Dan","Janet"]  │
│       1       │  Worker   │    1     │         ["Jake"]         │
│       2       │  Manager  │    1     │        ["Alice"]         │
│       2       │  Worker   │    2     │     ["Ruth","John"]      │
└───────────────┴───────────┴──────────┴──────────────────────────┘
```

## GROUP_BY Implementation

Like all SQLToy operations `GROUP_BY` takes a table in as an argument. Additionally it accepts an array of grouping columns and it returns a new table with one row for each group. In each row of the output it includes a "hidden" property `_groupRows` which holds a list of copies of the rows from the input table which were grouped into the row. That hidden property is used by the aggregate functions when they need to summarize the grouped data.

For example, here is a the output table which `GROUP_BY` generates for the call shown above:

```JSON
{
  "name": "employee",
  "rows": [
    {
      "_groupRows": [
        {
          "id": 1,
          "name": "Josh",
          "department_id": 1,
          "role": "Manager"
        },
        {
          "id": 6,
          "name": "Dan",
          "department_id": 1,
          "role": "Manager"
        },
        {
          "id": 7,
          "name": "Janet",
          "department_id": 1,
          "role": "Manager"
        }
      ],
      "department_id": 1,
      "role": "Manager"
    },
    {
      "_groupRows": [
        {
          "id": 2,
          "name": "Ruth",
          "department_id": 2,
          "role": "Worker"
        },
        {
          "id": 4,
          "name": "John",
          "department_id": 2,
          "role": "Worker"
        }
      ],
      "department_id": 2,
      "role": "Worker"
    },
    {
      "_groupRows": [
        {
          "id": 3,
          "name": "Jake",
          "department_id": 1,
          "role": "Worker"
        }
      ],
      "department_id": 1,
      "role": "Worker"
    },
    {
      "_groupRows": [
        {
          "id": 5,
          "name": "Alice",
          "department_id": 2,
          "role": "Manager"
        }
      ],
      "department_id": 2,
      "role": "Manager"
    }
  ]
}
```

The table is named `employee` which is copied from the input table. There is one row for each group and the grouping keys for that row are standard column properties on the row. So the first row has `department_id: 1` and `role: Manager` because all the rows in that group have that department_id and role. In addition you can see the `_groupRows` property which is an array of the source rows which went into that group.

Implementation:

```javascript
const GROUP_BY = (table, groupBys) => {
  const keyRows = {};
  for (const row of table.rows) {
    let key = groupBys.map(groupBy => row[groupBy]).join(US); // composite key
    if (!keyRows[key]) {
      keyRows[key] = [];
    }
    keyRows[key].push({...row}); // push a copy of the current row into it's group
  }

  const resultRows = [];
  for (const key in keyRows) {
    const resultRow = {
      _groupRows: keyRows[key]
    };
    for (const groupBy of groupBys) {
      resultRow[groupBy] = keyRows[key][0][groupBy]; // take the grouped value from the first entry in keyrows
    }
    resultRows.push(resultRow);
  }

  return {
    name: table.name,
    rows: resultRows
  }
}
```

The implementation uses two consecutive loops. The first loops over the input table rows and groups them based on the groupBy columns. The second goes over the groups and builds the output table with a row for each group and the `_groupRows` property which contains a copy of all rows in the group.

##### Composite keys

While building groups we make use of an object to store the groups of rows. The index into this object is a composite key made up of the values from the grouping columns in the current row. A composite key is a key made up of multiple values. This is how we handle multiple group by keys.

When using composite keys on a javascript object we must handle the following case. Say we are grouping by two columns and in one column the value is `A` and in the second the value is `BC`. The easiest way to make a composite key is just to concatenate them and use `ABC` as our key. This fails in the case where another row has `AB` for the first column and `C` for the second. This concatenation approach will generate the same composite key `ABC` for both columns and the resulting collision will group rows which are not supposed to be grouped.

To handle collisions we concatenate keys but separate them with a [Unit Separator](https://en.wikipedia.org/wiki/Unit_separator). This is a special Unicode character specifically reserved for this purpose. This character is invisible but if we represent it with `␟` then the two composite keys from above would be `A␟BC` and `AB␟C` which will not collide.

Once the second loop completes the output table is built and we just return it.

***

### Next section: [Aggregate Functions](https://github.com/weinberg/SQLToy/wiki/Aggregate-Functions)
