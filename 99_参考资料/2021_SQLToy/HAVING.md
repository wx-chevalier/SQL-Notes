# HAVING

***A WHERE clause for GROUP BY***

The HAVING clause is used to filter the results of a GROUP BY operation. It is functionally equivalent to the `WHERE` operation and in SQLToy is implemented the same way.

```javascript
const HAVING = (table, pred) => {
  return {
    name: table.name,
    rows: table.rows.filter(pred)
  }
}
```

The resulting table only includes rows which satisfy the predicate. Since it is run after the `GROUP BY` and Aggregate Functions the predicate can reference columns created by them like `AVG()` or `MAX()`.

For a discussion of filtering using a predicate refer to [WHERE](https://github.com/weinberg/SQLToy/wiki/WHERE).

***

### Next section: [SELECT](https://github.com/weinberg/SQLToy/wiki/SELECT)
