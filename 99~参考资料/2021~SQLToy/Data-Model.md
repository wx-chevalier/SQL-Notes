# Data Model

SQLToy has a very simple internal data model. We have no persistence so we do not have to worry about writing anything
to disk; everything is stored in memory.

Our database is stored in a variable called `database`. It is initialized like:

```javascript
function initSQLToy() {
  database = {
    tables: {}
  }
}
```

`database` has one property: `tables`. This is an object which stores all the tables in the database indexed by name. You can see how a
table is created in the `CREATE_TABLE` method:

```javascript
function CREATE_TABLE(name) {
  database.tables[name] = {
    name,
    rows: []
  }

  return database.tables[name];
}
```

This function takes a name of a table and adds a new empty table to the `database.tables[]` array. Note that we do not
do any error checking here. If the table name already exists we overwrite it. This is in keeping with the SQLToy
philosophy of simplicity. The purpose of SQLToy is not to make a database which is actually useful! The purpose is to
teach SQL.

In the same vein we do not create a schema for our table. In a real database `CREATE TABLE` takes a list of columns and
types. This way the database can typecheck the values being inserted into the table. We forgo all that as well. We want
to keep the focus on the SQL operations without worrying about error handling.

The object which is inserted into the `tables` map is our new table. This object has two properties: `name` and `rows`.
The `name` is the name of the table. This is used for output and for creating column names when doing joins.

The second property is an array of rows. Each row is represented by an object. We can see how they are created
in `INSERT_INTO`:

```javascript
function INSERT_INTO(tableName, r) {
  let rows;
  if (Array.isArray(r)) {
    rows = r;
  } else {
    rows = [r];
  }
  const table = database.tables[tableName];
  table.rows = [...table.rows, ...rows];
}
```

This function accepts a `tableName` and a second argument which can be either a single row object or an array of row
objects. The new row(s) are then appended to the existing rows.

Rows are just objects which have the column name as the key and the value for the column as the value. For example:

```
{
    id: 1,
    name: 'Josh',
    department_id: 1,
}
```

All together, a table to hold data about stories might be created like this:

```javascript
initSQLToy();
CREATE_TABLE('stories');
INSERT_INTO('stories', [
  {id: 1, name: 'The Elliptical Machine that ate Manhattan', author_id: 1},
  {id: 2, name: 'Queen of the Bats', author_id: 2},
  {id: 3, name: 'ChocoMan', author_id: 3},
]);
```

The resulting table object looks like:

```JSON
{
  "name": "stories",
  "rows": [
    {
      "id": 1,
      "name": "The Elliptical Machine that ate Manhattan",
      "author_id": 1
    },
    {
      "id": 2,
      "name": "Queen of the Bats",
      "author_id": 2
    },
    {
      "id": 3,
      "name": "ChocoMan",
      "author_id": 3
    }
  ]
}
``` 

### Output helper `table`

To print a table in a nice format you can use the `table()` helper. The above table looks like:

```
┌────┬─────────────────────────────────────────────┬───────────┐
│ id │                    name                     │ author_id │
├────┼─────────────────────────────────────────────┼───────────┤
│ 1  │  The Elliptical Machine that ate Manhattan  │     1     │
│ 2  │              Queen of the Bats              │     2     │
│ 3  │                  ChocoMan                   │     3     │
└────┴─────────────────────────────────────────────┴───────────┘
```

***

Now you are ready for the next section: [Key Concepts](https://github.com/weinberg/SQLToy/wiki/Two-Key-Concepts) 
