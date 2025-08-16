---
title: POSTGRESQL
date: 2023-12-24
tags:
  - seed
enableToc: true
---
# Notes on Postgresql

**Select statement:**

However, it is not a good practice to use the asterisk (`*`) in the `SELECT` statement when you embed SQL statements in the application code like [Python](https://www.postgresqltutorial.com/postgresql-python/), [Java](https://www.postgresqltutorial.com/postgresql-jdbc/), Node.js, or [PHP](https://www.postgresqltutorial.com/postgresql-php/) due to the following reasons:

1. Database performance. Suppose you have a table with many columns and a lot of data, the `SELECT` statement with the asterisk (`*`) shorthand will select data from all the columns of the table, which may not be necessary to the application.
2. Application performance. Retrieving unnecessary data from the database increases the traffic between the database server and application server. In consequence, your applications may be slower to respond and less scalable.

Because of these reasons, it is a good practice to explicitly specify the column names in the `SELECT` clause whenever possible to get only necessary data from the database.

More useful for large enterprise teams