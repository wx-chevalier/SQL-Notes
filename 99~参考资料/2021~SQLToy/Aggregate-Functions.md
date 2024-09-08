# Aggregate Functions

***Summarizing data***

An aggregate function is capable of taking multiple pieces of data and summarizing them with a single value.

For example, `AVG()` is an aggregate function which returns the average of the values in a grouped column. Take the following table of test scores:

```SQL
SELECT * FROM test_scores;

+--------------+-----------+---------+
| student_id   | test_id   | score   |
|--------------+-----------+---------|
| 1            | 1         | 100     |
| 1            | 2         | 90      |
| 2            | 1         | 85      |
| 2            | 2         | 80      |
| 3            | 1         | 75      |
| 3            | 2         | 99      |
+--------------+-----------+---------+
```

In this table we have test scores for three students across two tests. If we want to know the average score per student per test we can use group by and `AVG()`.

```SQL
SELECT student_id, AVG(score) FROM test_scores GROUP BY student_id;

+--------------+---------------------+
| student_id   | avg                 |
|--------------+---------------------|
| 1            | 95.0000000000000000 |
| 2            | 82.5000000000000000 |
| 3            | 87.0000000000000000 |
+--------------+---------------------+
```

Here we are using a `GROUP BY` as described in [GROUP_BY](https://github.com/weinberg/SQLToy/wiki/GROUP_BY).

In SQLToy this query would look like:

```javascript
const testScores = FROM('test_scores');
result = GROUP_BY(testScores, ['student_id']);
result = AVG(result, 'score');
result = SELECT(result, ['student_id','AVG(score)']);
table(result)
```

Resulting in:

```
┌────────────┬────────────┐
│ student_id │ AVG(score) │
├────────────┼────────────┤
│     1      │     95     │
│     2      │    82.5    │
│     3      │     87     │
└────────────┴────────────┘
```

### SQLToy Aggregate Functions

SQLToy currently has the following aggregate functions implemented:

- `ARRAY_AGG(column)` - Display the values from a grouped column in array form.
- `AVG(column)` - Average the values in the grouped column.
- `MAX(column)` - Returns the max value from the grouped column.
- `MIN(column)` - Returns the min value from the grouped column.
- `COUNT(column)` - Returns the count of rows in the grouped column.

## Implementation

Since all the aggregate functions take the same approach to computing their result we have an `aggregateHelper()` function which extracts the common code. The individual aggregate functions call this helper with an `aggFunc` which is used to do the aggregation.

```javascript
const aggregateHelper = (table, column, aggName, aggFunc) => {
  for (const row of table.rows) {
    const pick = row._groupRows.map(gr => gr[column])
    row[`${aggName}(${column})`] = aggFunc(pick);
  }
  return table;
}

const ARRAY_AGG = (table, column) => {
  return aggregateHelper(table, column, 'ARRAY_AGG', values => JSON.stringify(values));
}
```

Taking `ARRAY_AGG` as an example we see it calls `aggregateHelper` with the table and column to be aggregated. The third parameter is a label for the resulting column name. The last argument is the `aggFunc` which takes an array of values and summarizes them. In this case it just stringifies the array.

The `aggregateHelper` loops over all the rows of the input table and for each row creates a `pick` array which contains the values for the requested column from all the rows in the group. This makes use of the `_groupRows` "hidden" column as described in [GROUP_BY](https://github.com/weinberg/SQLToy/wiki/GROUP_BY). Once this array is built it then creates a new output column using the `aggName` and `(column)` for the column name. This ends up looking like `ARRAY_AGG(score)` in our example. The value for this column is the result of calling the `aggFunc()` callback.

All told this results in a new table being returned which looks like this (for `ARRAY_AGG(score)`):

```JSON
{
  "name": "test_scores",
  "rows": [
    {
      "_groupRows": [
        {
          "student_id": 1,
          "test_id": 1,
          "score": 100
        },
        {
          "student_id": 1,
          "test_id": 2,
          "score": 90
        }
      ],
      "student_id": 1,
      "ARRAY_AGG(score)": "[100,90]"
    },
    {
      "_groupRows": [
        {
          "student_id": 2,
          "test_id": 1,
          "score": 85
        },
        {
          "student_id": 2,
          "test_id": 2,
          "score": 80
        }
      ],
      "student_id": 2,
      "ARRAY_AGG(score)": "[85,80]"
    },
    {
      "_groupRows": [
        {
          "student_id": 3,
          "test_id": 1,
          "score": 75
        },
        {
          "student_id": 3,
          "test_id": 2,
          "score": 99
        }
      ],
      "student_id": 3,
      "ARRAY_AGG(score)": "[75,99]"
    }
  ]
}
```

Note the new column `ARRAY_AGG(score)` which has been added. The internal `_groupRows` data is still in the table which allows other aggregate functions to be called on the results.

The result in table format looks like:

```
┌────────────┬──────────────────┐
│ student_id │ ARRAY_AGG(score) │
├────────────┼──────────────────┤
│     1      │     [100,90]     │
│     2      │     [85,80]      │
│     3      │     [75,99]      │
└────────────┴──────────────────┘
```

The other aggregate functions are implemented similarly, using the `arrayAgg` callback to reduce the array of input to a single output value.

```javascript
// AVG reduces the values into a single average
const AVG = (table, column) => {
  return aggregateHelper(table, column, 'AVG', values => {
    const total = values.reduce((p, c) => p + c, 0);
    return total / values.length;
  });
}

// MAX aggregate function
const MAX = (table, column) => {
  const getMax = (a, b) => Math.max(a, b);
  return aggregateHelper(table, column, 'MAX', values => values.reduce(getMax));
}

// MIN aggregate function
const MIN = (table, column) => {
  const getMin = (a, b) => Math.min(a, b);
  return aggregateHelper(table, column, 'MIN', values => values.reduce(getMin));
}

// COUNT aggregate function
const COUNT = (table, column) => {
  return aggregateHelper(table, column, 'COUNT', values => values.length);
}
```

***

### Next Section: [HAVING](https://github.com/weinberg/SQLToy/wiki/HAVING)
