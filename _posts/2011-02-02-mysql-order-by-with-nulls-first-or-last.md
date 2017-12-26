---
layout: post
title: "MySQL ORDER BY with nulls first or last (at bottom or top)"
date: 2011-02-02
---

Want to create a MySQL query where a column is sorted in ascending order but with nulls last (at the bottom), or where a column is sorted in descending order but with nulls first (at the top)? You can use `ISNULL(column)` to do this.

Normally, a query with `ORDER BY column ASC` will display nulls first (at the top):

``` sql
SELECT product, number FROM products
ORDER BY number ASC;
```

| product | number |
| ------- | ------ |
| Bread   | NULL   |
| Sausage | NULL   |
| Bacon   | 0      |
| Cheese  | 2      |
| Milk    | 2      |
| Butter  | 3      |
| Eggs    | 7      |

and a query with `ORDER BY column DESC` will display nulls last (at the bottom):

``` sql
SELECT product, number FROM products
ORDER BY number DESC;
```

| product | number |
| ------- | ------ |
| Eggs    | 7      |
| Butter  | 3      |
| Cheese  | 2      |
| Milk    | 2      |
| Bacon   | 0      |
| Bread   | NULL   |
| Sausage | NULL   |

If you want to change the position of nulls in the results, you can use `ISNULL(column)`. The following query displays the number column in ascending order but with nulls last (at the bottom). I have included `ISNULL(number)` in the results table so that you can see its effect on the sorting, but you don't normally need to include it in the list of columns after `SELECT`, just in the `ORDER BY` section.

``` sql
SELECT product, number, ISNULL(number) FROM products
ORDER BY ISNULL(number) ASC, number ASC;
```

| product | number | isnull(number) |
| ------- | ------ | -------------- |
| Bacon   | 0      | 0              |
| Cheese  | 2      | 0              |
| Milk    | 2      | 0              |
| Butter  | 3      | 0              |
| Eggs    | 7      | 0              |
| Bread   | NULL   | 1              |
| Sausage | NULL   | 1              |

and the following query displays the number column in descending order but with nulls first (at the top):

``` sql
SELECT product, number, ISNULL(number) FROM products
ORDER BY ISNULL(number) DESC, number DESC;
```

| product | number | isnull(number) |
| ------- | ------ | -------------- |
| Bread   | NULL   | 1              |
| Sausage | NULL   | 1              |
| Eggs    | 7      | 0              |
| Butter  | 3      | 0              |
| Cheese  | 2      | 0              |
| Milk    | 2      | 0              |
| Bacon   | 0      | 0              |

Note the position of the zeros in the results tables; zero is not the same as null!
