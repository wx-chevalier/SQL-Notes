# From

The FROM clause is used to pick a table from which to select data. It accepts a single table name and returns the table from the database.

The implementation is very simple:

```javascript
function FROM(tableName) {
  return database.tables[tableName];
}
```

Tables in our system are stored by name in a `database.tables` object. The requested table is looked up and returned.

### What about FROM multiple tables?

In traditional SQL you can list multiple tables (and subqueries) after the FROM keyword. Multiple tables in the FROM clause are treated as if they were cross joined. For example in PostgreSQL:

```SQL
CREATE TABLE test1 (c char);
CREATE TABLE test2 (c char);
INSERT INTO test1 VALUES ('A');
INSERT INTO test1 VALUES ('B');
INSERT INTO test2 VALUES ('1');
INSERT INTO test2 VALUES ('2');
SELECT * FROM test1, test2;
+-----+-----+
| c   | c   |
|-----+-----|
| A   | 1   |
| A   | 2   |
| B   | 1   |
| B   | 2   |
+-----+-----+
```

In our database we support this with the CROSS_JOIN method. For example:

```javascript
let test1 = CREATE_TABLE('test1');
let test2 = CREATE_TABLE('test2');
INSERT_INTO('test1',{ c: 'A' });
INSERT_INTO('test1',{ c: 'B' });
INSERT_INTO('test2',{ c: '1' });
INSERT_INTO('test2',{ c: '2' });
result = CROSS_JOIN(test1, test2);
table(result);
```

```
┌─────────┬─────────┐
│ test1.c │ test2.c │
├─────────┼─────────┤
│    A    │    1    │
│    A    │    2    │
│    B    │    1    │
│    B    │    2    │
└─────────┴─────────┘
```

Subqueries are also possible by just joining on a separately prepared table.

***

### Next section: [CROSS_JOIN](https://github.com/weinberg/SQLToy/wiki/CROSS_JOIN)
