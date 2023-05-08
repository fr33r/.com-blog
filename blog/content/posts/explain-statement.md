+++
title = "Explaining the `EXPLAIN`"
date = "2023-05-06T22:47:06-07:00"
author = "fr33r"
cover = ""
tags = ["sql", "database"]
keywords = ["explain", "sql", "database", "performance", "analyze"]
description = ""
showFullContent = false
readingTime = true
hideComments = false
draft = false
+++

When using an relational database management system (RDBMS), also called a
"relational database" or "SQL database", it's only a matter of time before
you want to gain more insight into the performance of a subset of your queries.
These databases are equipped with the ability for engineers to examine how the
queries are planned to be executed, referred to as a "query execution plan",
and in some cases, able to compare that plan to what actually occurred when
the query is run.

This process of asking the database to provide these insights is technically
referred to as "explaining" or "describing" the query, and commonly utilize
the keyword `EXPLAIN` to invoke that process.

## `EXPLAIN` Statement

---

The `EXPLAIN` statement supported by various relational databases exposes
how the the database plans on executing the query provided to it. It can be
used to gain these insights on most query statement types, such as `SELECT`,
`INSERT`, `DELETE`, `UPDATE`, and potentially more, but in practice is mainly
used with `SELECT`.

> ℹ️  The component within a relational database responsible for
determinining the query execution plan is often called the "optimizer" or
"planner".

When providing the `EXPLAIN` statement with a query, that query is not actually
executed. Instead, the query execution plan that would have been used is
retrieved and provided in the output. This is helpful to know, because queries
that have been known to be problematic and utilize a large portion of system
resources can be safely specified to `EXPLAIN` without the side-effects
caused by actually executing that query.

> ⚠️ Most relational databases require that you have permissions to run
the query provided to `EXPLAIN`, and a subset have an explicit permission
associated to the `EXPLAIN` command. Make sure you have the proper permissions.

### Example Syntax

Although various relational databases have differing alternative syntaxes
and options that can be provided to further refine how `EXPLAIN` behaves,
most support a common (canonical) syntax. The examples laid out here
demonstrate both the common and alternative syntaxes for several popular
relational databases.

> **FUN FACT**: The cited relational databases together represent `97.96%`
market share; PostgreSQL has [`17.32%`][postgresql-market-share], MySQL has
[`44.68%`][mysql-market-share], SQLite has [`3.32%`][sqlite-market-share],
and SQL Server has [`32.64%`][sql-server-market-share].

#### PostgreSQL

{{< code language="sql" title="PostgreSQL" id="1" expand="Show" collapse="Hide" isCollapsed="true" >}}
-- canonical syntax.
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

> **NOTE**: PostgreSQL supports a vast array of options that can provided to
`EXPLAIN`, such as `VERBOSE`, `COSTS`, `BUFFERS`, `WAL`, etc. For more
information on these options, check out [this][postgres-explain-options]
documentation.

#### MySQL

{{< code language="sql" title="MySQL" id="2" expand="Show" collapse="Hide" isCollapsed="true" >}}
-- canonical syntax.
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;

-- alternative syntax.
DESCRIBE SELECT name FROM players WHERE players.games_played > 10;

-- alternative syntax.
DESC SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

#### SQLite

{{< code language="sql" title="SQLite" id="3" expand="Show" collapse="Hide" isCollapsed="true" >}}
-- canonical syntax.
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;

-- alternative syntax.
EXPLAIN QUERY PLAN SELECT name FROM players WHERE players.games_played > 10;

{{< /code >}}

#### SQL Server

{{< code language="sql" title="SQL Server" id="4" expand="Show" collapse="Hide" isCollapsed="true" >}}
-- canonical syntax.
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

> **NOTE**: SQL Server supports a `WITH_RECOMMENDATIONS` option, which results in
the output containing recommendations to optimize the provided query. For more
information on this option, check out [this][with-recommendations-docs]
documentation.

### Output

Now that we know how to issue the `EXPLAIN` statement, it's time to dive into
the output. In contrast to the common syntax across various relational databases,
the output format, along with the level of configurability of that format,
deviates significantly.

#### PostgreSQL

{{< code language="sql" title="PostgreSQL" id="1" expand="Show" collapse="Hide" isCollapsed="true" >}}
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

#### MySQL

{{< code language="sql" title="MySQL" id="2" expand="Show" collapse="Hide" isCollapsed="true" >}}
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

#### SQLite

{{< code language="sql" title="SQLite" id="3" expand="Show" collapse="Hide" isCollapsed="true" >}}
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

#### SQL Server

{{< code language="sql" title="SQL Server" id="4" expand="Show" collapse="Hide" isCollapsed="true" >}}
EXPLAIN SELECT name FROM players WHERE players.games_played > 10;
{{< /code >}}

## When should I use `EXPLAIN`?

## Comparing plans with actual

## Any tools to help me read the output?


OUTLINE

- what circumstances would lead to you wanting to use `EXPLAIN`?
- what is `EXPLAIN`?
  - talk about how it is an estimate.
  - what is a query execution plan?
  - what is the optimizer?
  - can be used with other things besides `SELECT`, but let's focus on that.
- what does the output look like?
  - MySQL?
  - SQLLite?
  - PostgreSQL?
- what about `EXPLAIN ANALYZE`?
  - talk about how it actually runs the query and compares.
- optimizer hints?
- what does the output mean?
- example of how `EXPLAIN` output can help improve query performance.
  - create tables related to hockey:
    - players
      - name
      - position
    - teams
      - name
    - players_teams
      - player_id
      - team_id
      - season_id
      - games_played
      - icetime
      - shifts
    - season
      - year

RESOURCES

- https://www.postgresql.org/docs/current/sql-explain.html
- https://dev.mysql.com/doc/refman/8.0/en/using-explain.html
- https://dev.mysql.com/doc/refman/8.0/en/explain.html
- https://www.sqlite.org/eqp.html
- https://www.sqlite.org/optoverview.html
- https://www.postgresql.org/docs/current/planner-optimizer.html
- https://dev.mysql.com/doc/refman/8.0/en/controlling-query-plan-evaluation.html
- https://learn.microsoft.com/en-us/sql/t-sql/queries/explain-transact-sql?view=azure-sqldw-latest

---


Perhaps you noticed a slow query when developing a new feature, or maybe
there is application timing out in your production environment. May

[with-recommendations-docs]: https://learn.microsoft.com/en-us/sql/t-sql/queries/explain-transact-sql?view=azure-sqldw-latest#with_recommendations
[postgres-explain-options]: https://www.postgresql.org/docs/current/sql-explain.html
[postgresql-market-share]: https://6sense.com/tech/relational-databases/postgresql-market-share
[mysql-market-share]: https://6sense.com/tech/relational-databases/mysql-market-share
[sqlite-market-share]: https://6sense.com/tech/relational-databases/sqlite-market-share
[sql-server-market-share]: https://6sense.com/tech/database/microsoft-sql-server-market-share
