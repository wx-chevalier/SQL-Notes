## OFFSET and LIMIT

***Return a subset of the rows in a table starting from a given row number***

Sometimes your query returns too many rows to be useful. The user probably does not want to see _all_ of their bank transactions going back to the start of their account. Also for performance reasons it is impractical to return entire tables of data from the database all the time. For these reasons it is common to request data in groups known as pages. 

This approach (called pagination) is done by requesting the starting row and a row count to return. `OFFSET` is used to specify the starting row. `LIMIT` says how many rows to return in the resulting table.

In SQL if we have a table of filenames we could request the entire table:

```sql
SELECT * FROM repo_files
+----------------------------------------+
| filename                               |
|----------------------------------------|
| 00-INDEX                               |
| 07                                     |
| 1040.bin.ihex                          |
| 11d.c                                  |
| 11d.h                                  |
| 1200.bin.ihex                          |
| 12160.bin.ihex                         |
| 1232ea962bbaf0e909365f4964f6cceb2ba8ce |
| 1280.bin.ihex                          |
| 15562512ca6cf14c1b8f08e09d5907118deaf0 |
| 17                                     |
| 1d                                     |
| 1.Intro                                |
| 21142.c                                |
| 21285.c                                |
| 2860_main_dev.c                        |
| 2860_rtmp_init.c                       |
| 2870_main_dev.c                        |
| 2870_rtmp_init.c                       |
| 2.Process                              |

...entire table...

+----------------------------------------+
SELECT 9981
```

9,981 filenames is too many. Let's take one page of them starting at 0 and going to 10:

```sql
SELECT * FROM filenames LIMIT 10;

+----------------------------------------+
| filename                               |
|----------------------------------------|
| 00-INDEX                               |
| 07                                     |
| 1040.bin.ihex                          |
| 11d.c                                  |
| 11d.h                                  |
| 1200.bin.ihex                          |
| 12160.bin.ihex                         |
| 1232ea962bbaf0e909365f4964f6cceb2ba8ce |
| 1280.bin.ihex                          |
| 15562512ca6cf14c1b8f08e09d5907118deaf0 |
+----------------------------------------+
SELECT 10
```

You don't have to specify `OFFSET 0` if you want to start at the first record. But once we get to the second page of data we need to specify both `OFFSET` and `LIMIT`:

```sql
SELECT * FROM filenames OFFSET 10 LIMIT 10

+------------------+
| filename         |
|------------------|
| 17               |
| 1d               |
| 1.Intro          |
| 21142.c          |
| 21285.c          |
| 2860_main_dev.c  |
| 2860_rtmp_init.c |
| 2870_main_dev.c  |
| 2870_rtmp_init.c |
| 2.Process        |
+------------------+
SELECT 10
```

Now we have retrieved the second set of 10 filenames from the table. Note that `OFFSET` and `LIMIT` happen at the very end of the [SQL Order of Operations](https://github.com/weinberg/SQLToy/wiki/Two-Key-Concepts#1-the-database-does-not-execute-a-query-in-the-order-it-is-written) so any `ORDER BY` operation has already happened and the offset will implicitly respect that.

Now to do pagination in SQLToy. We just call `OFFSET(row)` and `LIMIT(count)` with the appropriate values:

```javascript
const filenames = FROM('filenames');
result = LIMIT(result, 10);
table(result);
```

```
┌──────────────────────────────────────────┐
│                 filename                 │
├──────────────────────────────────────────┤
│                 00-INDEX                 │
│                    07                    │
│              1040.bin.ihex               │
│                  11d.c                   │
│                  11d.h                   │
│              1200.bin.ihex               │
│              12160.bin.ihex              │
│  1232ea962bbaf0e909365f4964f6cceb2ba8ce  │
│              1280.bin.ihex               │
│  15562512ca6cf14c1b8f08e09d5907118deaf0  │
└──────────────────────────────────────────┘
```

Again we don't need to specify an offset of 0 to start at the beginning.

Getting the next page we do:

```javascript
const filenames = FROM('filenames');
let result = OFFSET(filenames, 10);
result = LIMIT(result, 10);
table(result);
```

```
┌────────────────────┐
│      filename      │
├────────────────────┤
│         17         │
│         1d         │
│      1.Intro       │
│      21142.c       │
│      21285.c       │
│  2860_main_dev.c   │
│  2860_rtmp_init.c  │
│  2870_main_dev.c   │
│  2870_rtmp_init.c  │
│     2.Process      │
└────────────────────┘
```

## OFFSET and LIMIT implementation

Both `OFFSET()` and `LIMIT()` make use of built-in Javascript array methods so the implementation is very straightforward.

`OFFSET()` uses `slice()` to return the rows after a given offset:

```javascript
const OFFSET = (table, offset) => {
  return {
    name: table.name,
    rows: table.rows.slice(offset),
  }
}
```

`LIMIT()` also uses `slice()`. In this case it returns the first `limit` rows from the table.

```javascript
const LIMIT = (table, limit) => {
  return {
    name: table.name,
    rows: table.rows.slice(0, limit),
  }
}
```

Both functions return a new table with the subset of the rows from the original table.

***

### Next Section: [Conclusion](https://github.com/weinberg/SQLToy/wiki/Finally)
