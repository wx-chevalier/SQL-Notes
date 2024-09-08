# DISTINCT

***Reducing rows based on the uniqueness of certain columns***

`DISTINCT` acts much like the `GROUP BY` operation in that it reduces rows of the output table by collapsing rows which have common values for the `columns` provided. In the case of `GROUP BY` these collapsed rows are made available to Aggregate Functions which follow. `DISTINCT` however just drops these rows completely.

For example in this table we have a list of friends and their home city and state:

```
+------+------------------+----------+
| id   | city             | state    |
|------+------------------+----------|
| 1    | Denver           | Colorado |
| 2    | Colorado Springs | Colorado |
| 3    | South Park       | Colorado |
| 4    | Corpus Christi   | Texas    |
| 5    | Houston          | Texas    |
| 6    | Denver           | Colorado |
| 7    | Corpus Christi   | Texas    |
+------+------------------+----------+
```

If we want to know which states our friends are in we can use distinct on state. In PostgreSQL:

```sql
SELECT DISTINCT state FROM friends;

+----------+
| state    |
|----------|
| Colorado |
| Texas    |
+----------+
```

In SQLToy:

```javascript
const friends = FROM('friends');
const result = DISTINCT(friends, ['state']);
table(result);
```

```
┌────────────┐
│   state    │
├────────────┤
│  Colorado  │
│   Texas    │
└────────────┘
```

Now if we want to know which city and state we have friends in we can do `DISTINCT` on multiple columns. In PostgreSQL:

```sql
SELECT DISTINCT state, city FROM friends;

+----------+------------------+
| state    | city             |
|----------+------------------|
| Texas    | Houston          |
| Texas    | Corpus Christi   |
| Colorado | South Park       |
| Colorado | Colorado Springs |
| Colorado | Denver           |
+----------+------------------+
```

Note that we have two friends in Denver, Colorado and two friends in Corpus Christi, Texas but those cities are only listed once each.

In SQLToy:

```javascript
const friends = FROM('friends');
const result = DISTINCT(friends, ['city','state']);
table(result);
```

```
┌────────────────────┬────────────┐
│        city        │   state    │
├────────────────────┼────────────┤
│       Denver       │  Colorado  │
│  Colorado Springs  │  Colorado  │
│     South Park     │  Colorado  │
│   Corpus Christi   │   Texas    │
│      Houston       │   Texas    │
└────────────────────┴────────────┘
```

# DISTINCT Implementation

`DISTINCT` loops over the rows of the input table and creates a [Composite Key](https://github.com/weinberg/SQLToy/wiki/GROUP_BY#composite-keys) on the values of each of the columns in `columns[]`. It uses the key to keep track of rows from the input table which share that key. Sharing the same key means they have the same values for the distinct columns and thus should be grouped together. Unlike `GROUP_BY`, `DISTINCT` does not need to store the list of rows in each group so it just stores a single row and for simplicity's sake just overwrites it if another row has the same key.

Once the distinct rows have been collected in `_distinct` it loops over the keys of that table and creates one row in the output table using only the requested columns.

```javascript
const DISTINCT = (table, columns) => {
  const _distinct = {}
  for (const row of table.rows) {
    // make composite key
    let key = columns.map(column => row[column]).join(US);
    _distinct[key] = row;
  }

  const newRows = [];
  for (let key in _distinct) {
    const newRow = {};
    for (let column of columns) {
      newRow[column] = _distinct[key][column];
    }
    newRows.push(newRow);
  }

  return {
    name: table.name,
    rows: newRows,
  };
}
```

***

### Next section: [ORDER_BY](https://github.com/weinberg/SQLToy/wiki/ORDER_BY)
