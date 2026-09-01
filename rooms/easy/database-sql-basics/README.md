# Database SQL Basics

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Databases

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included. Code examples below are generic
> reference examples, not captures from a completed session.

## Overview

SQL (Structured Query Language) is the standard language for defining and manipulating data in
relational databases — systems like MySQL, PostgreSQL, Microsoft SQL Server, and SQLite that organize
data into tables of rows and columns related to one another by keys. Fundamentals rooms in this space
teach the core data-retrieval statements (`SELECT`, `WHERE`, `JOIN`) because they are the same
statements an attacker abuses when a web application builds SQL queries by concatenating untrusted
input — making SQL literacy a prerequisite for understanding SQL injection, one of the
longest-running and most damaging web vulnerability classes.

## Core Concepts

### Tables and a Working Example Schema

Assume two tables that a small web application might use — `users` and `orders`:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'customer'
);

CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL,
    item VARCHAR(100) NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

`orders.user_id` is a foreign key referencing `users.id` — this is what allows the two tables to be
joined later.

### SELECT and WHERE

`SELECT` retrieves rows; `WHERE` filters which rows are returned.

```sql
SELECT username, email
FROM users
WHERE role = 'admin';
```

```
username  | email
----------+---------------------
adminuser | admin@example.com
```

`WHERE` accepts comparison operators (`=`, `!=`, `<`, `>`), logical operators (`AND`, `OR`, `NOT`),
pattern matching (`LIKE '%term%'`), and set membership (`IN (...)`). This is also precisely the
clause where SQL injection vulnerabilities usually surface, since `WHERE` conditions are the most
common place a web application inserts user-supplied values (a login form's username/password, a
search box, a URL parameter).

### JOIN

`JOIN` combines rows from two or more tables based on a related column — typically a foreign key.

```sql
SELECT users.username, orders.item, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id
WHERE orders.amount > 50.00;
```

```
username | item          | amount
---------+---------------+--------
alice    | Mechanical KB | 89.99
bob      | Monitor Stand | 54.50
```

An `INNER JOIN` returns only rows that have a match in both tables. `LEFT JOIN` would additionally
return every row from `users` even when there is no matching row in `orders` (with `NULL`s filling
the `orders` columns), which matters when writing reports that must not silently drop unmatched
records.

### Aggregation

```sql
SELECT users.username, COUNT(orders.id) AS order_count, SUM(orders.amount) AS total_spent
FROM users
INNER JOIN orders ON users.id = orders.user_id
GROUP BY users.username
ORDER BY total_spent DESC;
```

```
username | order_count | total_spent
---------+-------------+------------
alice    | 3           | 214.97
bob      | 1           | 54.50
```

`GROUP BY` collapses rows sharing a value into a single summary row per group, and aggregate
functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) compute over each group. `ORDER BY ... DESC` sorts the
result set, here by total spend, descending.

## Why It Matters for Security

**SQL injection (SQLi)** is the canonical example of what happens when application code builds a SQL
statement by directly concatenating untrusted input, rather than treating the input as data. OWASP
lists Injection (of which SQLi is the most prominent form) among its Top 10 web application security
risks. Consider a naive login check built as a Python string:

```python
# VULNERABLE — do not use: string concatenation into a SQL statement
username = user_input          # attacker-controlled
query = f"SELECT * FROM users WHERE username = '{username}'"
```

If an attacker supplies `' OR '1'='1` as the username, the resulting statement becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

Because `'1'='1'` is always true, the `WHERE` clause matches every row in `users`, potentially
bypassing authentication entirely or dumping the whole table depending on how the application uses
the result. More advanced payloads use `UNION SELECT` to pull data out of unrelated tables, or
stacked/boolean/time-based techniques to exfiltrate data blind, one bit at a time, when the
application doesn't directly display query results.

The fix is **parameterized queries (prepared statements)**, which send the query structure and the
user-supplied values to the database separately, so the database engine never interprets the values
as SQL syntax:

```python
# SAFE — parameterized query, using a DB-API-style placeholder
cursor.execute(
    "SELECT * FROM users WHERE username = %s",
    (username,)
)
```

Here `username` is passed as a bound parameter, not spliced into the query text, so even a value like
`' OR '1'='1` is treated as a literal string to compare against, not executable SQL. This single
change — parameterize, never concatenate — is OWASP's primary defense recommendation against
injection, supplemented by least-privilege database accounts, strict input validation/allow-listing,
and escaping when parameterization genuinely isn't available (e.g. for dynamic identifiers, which
parameters cannot substitute for).

## Common Pitfalls / Misconfigurations

- **String-concatenated queries** anywhere user input reaches a database call — the root cause of
  nearly all SQL injection.
- **Overly privileged database accounts** for the web application — an app account with `DROP TABLE`
  or cross-schema access turns a read-only injection into full data destruction.
- **Verbose error messages** that leak database structure (table/column names, DBMS version) back to
  the client, which materially assists an attacker crafting an injection payload.
- **Blacklisting instead of parameterizing** — trying to filter out keywords like `SELECT` or `UNION`
  is trivially bypassed with case variation, comments, or encoding, and is not a substitute for
  parameterized queries.
- **Trusting client-side validation alone** — JavaScript form validation is a UX convenience, not a
  security control, since it can be bypassed by calling the backend endpoint directly.
- **Missing indexes on join/foreign-key columns**, which is a performance rather than security issue
  but frequently appears alongside the same schemas used to teach `JOIN`.

## Related TryHackMe Rooms in This Series

This room commonly precedes dedicated SQL injection rooms in the same learning path. See
`../python-simple-demo/README.md` for the scripting fundamentals (including exception handling around
user input) that underpin safe database-access code, such as the parameterized-query pattern shown
above.

## References

- OWASP, "SQL Injection": https://owasp.org/www-community/attacks/SQL_Injection
- OWASP Top 10 (2021), A03 Injection: https://owasp.org/Top10/A03_2021-Injection/
- OWASP SQL Injection Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- PostgreSQL documentation, `SELECT`: https://www.postgresql.org/docs/current/sql-select.html
- MDN Web Docs, "SQL injection": https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/SQL_injection
- Python `sqlite3` module — parameterized queries: https://docs.python.org/3/library/sqlite3.html#how-to-use-placeholders-to-bind-values-in-sql-queries
