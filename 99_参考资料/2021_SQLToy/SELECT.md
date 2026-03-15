# SELECT

***Specifying the columns to be returned by the query***

Contrary to what you might have thought, `SELECT` is one of the last operations in the [SQL Order of Operations](https://github.com/weinberg/SQLToy/wiki/Two-Key-Concepts). This is logical when you consider its role. `SELECT` basically just prepares a table for output. So any processing of data needs to happen before `SELECT` is invoked.

For example:

```SQL
SELECT * FROM games;
+-------------+------------+----------+------------------+
| player_id   | type       | result   | length_minutes   |
|-------------+------------+----------+------------------|
| 1           | Chess      | Win      | 23.5             |
| 1           | Chess      | Loss     | 26.5             |
| 2           | Checkers   | Loss     | 6.5              |
| 2           | Dominos    | Loss     | 9.1              |
| 1           | Battleship | Win      | 27.9             |
+-------------+------------+----------+------------------+
```

This table stores the results of some games. If we only want to view the type of game and the result we could use `SELECT` to include only those columns in the output:

```SQL
SELECT type, result;
+------------+----------+
| type       | result   |
|------------+----------|
| Chess      | Win      |
| Chess      | Loss     |
| Checkers   | Loss     |
| Dominos    | Loss     |
| Battleship | Win      |
+------------+----------+
```

In SQLToy we can do the same:

```javascript
const games = FROM('games');
const result = SELECT(games, ['type', 'result']);
table(result);
```

Resulting in:

```
┌───────────────┬────────┐
│     type      │ result │
├───────────────┼────────┤
│     Chess     │  Win   │
│     Chess     │  Loss  │
│   Checkers    │  Loss  │
│    Dominos    │  Loss  │
│  Battleship   │  Win   │
└───────────────┴────────┘
```

Select can pull from columns which were created by any previous operation. This includes from `JOIN` operations and Aggregate Functions. Joined tables' columns will have the original table name prefixed on the column. So when selecting, use the `table.column` syntax.

For example:

```javascript
const games = FROM('games');
const player = FROM('player');
let result = JOIN(games, player, c => c['games.player_id'] === c['player.id']);
result = SELECT(result, ['games.type', 'player.name', 'games.result']);
table(result);
```

will give us values from the joined `player` table:

```
┌───────────────┬─────────────┬──────────────┐
│  games.type   │ player.name │ games.result │
├───────────────┼─────────────┼──────────────┤
│     Chess     │    Josh     │     Win      │
│     Chess     │    Josh     │     Loss     │
│   Checkers    │    Ruth     │     Loss     │
│    Dominos    │    Ruth     │     Loss     │
│  Battleship   │    Josh     │     Win      │
└───────────────┴─────────────┴──────────────┘
```

## Aliases

A nice feature of the `SELECT` operation is aliasing. This allows you to re-name columns for output. To support aliases, `SELECT` in SQLToy accepts an optional third argument which is an object that maps table names.

For example, we could change the SELECT from the previous example to:

```javascript
result = SELECT(result, ['games.type', 'player.name', 'games.result'], {
                  'games.type': 'Game Type',
                  'player.name': 'Player Name',
                  'games.result': 'Win or Loss'
                });
```

And the result looks a bit more palatable:

```
┌───────────────┬─────────────┬─────────────┐
│   Game Type   │ Player Name │ Win or Loss │
├───────────────┼─────────────┼─────────────┤
│     Chess     │    Josh     │     Win     │
│     Chess     │    Josh     │    Loss     │
│   Checkers    │    Ruth     │    Loss     │
│    Dominos    │    Ruth     │    Loss     │
│  Battleship   │    Josh     │     Win     │
└───────────────┴─────────────┴─────────────┘
```

## Implementation

The SQLToy `SELECT` implementation builds a new table with only the columns given in the columns array argument.

```javascript
const SELECT = (table, columns, aliases = {}) => {
  const newrows = [];
  const colnames = {};

  for (const col of columns) {
    colnames[col] = aliases[col] ? aliases[col] : col;
  }

  for (const row of table.rows) {
    const newrow = {};
    for (let column of columns) {
      newrow[colnames[column]] = row[column];
    }
    newrows.push(newrow);
  }

  return {
    name: table.name,
    rows: newrows,
  };
}
```

First we set up a lookup table of aliased column names in `colnames`. This is keyed by the column name from the table and the value is the aliases name. If a column is not aliased then it is put in the lookup table with its original name.

Then for each row in the input table a new row is constructed with a column for each of `columns`, using the lookup table to name the column.

***

### Next Section: [DISTINCT](https://github.com/weinberg/SQLToy/wiki/DISTINCT)
