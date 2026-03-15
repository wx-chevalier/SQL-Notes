## Join Tables

It is common in relational databases to use a 'Join Table' to represent many-to-many relationships. A join table is a table which just holds the IDs of rows in two other tables.

For example given these tables:

```sql
people
┌────┬───────────┐
│ id │   name    │
├────┼───────────┤
│ 1  │   Josh    │
│ 2  │   Jane    │
│ 3  │   Ruth    │
│ 4  │  Elliot   │
│ 5  │  Michael  │
│ 6  │   Garth   │
└────┴───────────┘

club
┌────┬──────────────────────┐
│ id │         name         │
├────┼──────────────────────┤
│ 1  │      Cat Lovers      │
│ 2  │    House Builders    │
│ 3  │      Book Drive      │
│ 4  │  Carbon Offset Club  │
│ 5  │   Asian Languages    │
│ 6  │    Weekly Potluck    │
└────┴──────────────────────┘

people_club
┌───┬───┐
│ A │ B │
├───┼───┤
│ 1 │ 1 │
│ 1 │ 4 │
│ 2 │ 1 │
│ 3 │ 4 │
│ 3 │ 5 │
│ 4 │ 1 │
│ 4 │ 2 │
│ 4 │ 3 │
│ 4 │ 4 │
└───┴───┘
```

We have a `people` table with people and their names, a `club` table with club names and a `people_club` table which stores which person is in which club. Since we model this in a separate table (and not alongside the data in either people or club) then we can handle the many-to-many relationship where:
- a person is in zero or more clubs
- a club has zero or more people

To query the join table we use two separate joins. First from people to the people_club table and then from people_club to club.

For example in PostgreSQL:

```sql
SELECT people.name, club.name FROM people
  JOIN people_club ON people.id = people_club."A"
  JOIN club ON club.id = people_club."B"
  ORDER BY people.name;
  
+--------+--------------------+
| name   | name               |
|--------+--------------------|
| Elliot | Cat Lovers         |
| Elliot | House Builders     |
| Elliot | Book Drive         |
| Elliot | Carbon Offset Club |
| Jane   | Cat Lovers         |
| Josh   | Cat Lovers         |
| Josh   | Carbon Offset Club |
| Ruth   | Carbon Offset Club |
| Ruth   | Asian Languages    |
+--------+--------------------+
```

We selected just the two name columns (and sorted by people.name to make the output nicer).

In SQLToy this is done the same way:

```javascript
employee = FROM('employee');
club = FROM('club');
employee_club = FROM('employee_club');
result = INNER_JOIN(employee, employee_club, (c) => c["employee_club.A"] === c["employee.id"]);
result = INNER_JOIN(result, club, (c) => c["employee_club.B"] === c["club.id"]);
result = SELECT(result, ['employee.name', 'club.name']);
result = ORDER_BY(result, (a, b) => a['employee.name'].localeCompare(b['employee.name']));
table(result);
```

The result:

```
┌───────────────┬──────────────────────┐
│ employee.name │      club.name       │
├───────────────┼──────────────────────┤
│    Elliot     │      Cat Lovers      │
│    Elliot     │    House Builders    │
│    Elliot     │      Book Drive      │
│    Elliot     │  Carbon Offset Club  │
│     Jane      │      Cat Lovers      │
│     Josh      │      Cat Lovers      │
│     Josh      │  Carbon Offset Club  │
│     Ruth      │  Carbon Offset Club  │
│     Ruth      │   Asian Languages    │
└───────────────┴──────────────────────┘
```

***

### Next Section: [WHERE](https://github.com/weinberg/SQLToy/wiki/WHERE)
