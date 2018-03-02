# SQL Recursive Common Table Expression

### Structure of the account table:

```
+----+----------+-----------+
| id | owner_id | username  |
+----+----------+-----------+
|  1 | 3        | Alexander |
|  2 | 9        | Rolf      |
|  3 | 10       | Elizabeth |
|  4 | 10       | Anna      |
|  5 | 3        | Uwe       |
|  6 | 9        | Sabine    |
|  7 | 5        | Wolfgang  |
|  8 | 2        | Andrea    |
|  9 | null     | Jürgen    |
| 10 | null     | Heinz     |
+----+----------+-----------+
```

### Query to the accounts table:

```sql
WITH RECURSIVE recursive_cte (id, owner_id, username, route) AS (
  SELECT
    a1.id,
    a1.owner_id,
    a1.username,
    array_append('{}' :: VARCHAR [], a1.username)
  FROM
    accounts a1
  WHERE
    a1.owner_id IS NULL
  UNION ALL
  SELECT
    a2.id,
    a2.owner_id,
    a2.username,
    array_append(recursive_cte.route, a2.username)
  FROM recursive_cte
    INNER JOIN accounts a2
      ON recursive_cte.id = a2.owner_id
) SELECT
    rc.username,
    array_to_string(rc.route, '.') AS route
  FROM
    recursive_cte rc
  ORDER BY
    rc.route;
```

### SQL Recursive CTE result (Depth First Search Algorithm):

```
+-----------+------------------------------+
| username  |            route             |
+-----------+------------------------------+
| Heinz     | Heinz                        |
| Anna      | Heinz.Anna                   |
| Elizabeth | Heinz.Elizabeth              |
| Alexander | Heinz.Elizabeth.Alexander    |
| Uwe       | Heinz.Elizabeth.Uwe          |
| Wolfgang  | Heinz.Elizabeth.Uwe.Wolfgang |
| Jürgen    | Jürgen                       |
| Rolf      | Jürgen.Rolf                  |
| Andrea    | Jürgen.Rolf.Andrea           |
| Sabine    | Jürgen.Sabine                |
+-----------+------------------------------+
```

### SQL Recursive CTE result (Breadth First Search Algorithm):

###### Without using:

```sql
ORDER BY rc.route;
```
```
+-----------+------------------------------+
| username  |            route             |
+-----------+------------------------------+
| Jürgen    | Jürgen                       |
| Heinz     | Heinz                        |
| Rolf      | Jürgen.Rolf                  |
| Elizabeth | Heinz.Elizabeth              |
| Anna      | Heinz.Anna                   |
| Sabine    | Jürgen.Sabine                |
| Alexander | Heinz.Elizabeth.Alexander    |
| Uwe       | Heinz.Elizabeth.Uwe          |
| Andrea    | Jürgen.Rolf.Andrea           |
| Wolfgang  | Heinz.Elizabeth.Uwe.Wolfgang |
+-----------+------------------------------+
```
