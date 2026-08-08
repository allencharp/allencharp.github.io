---
layout: post
title:  "SQL Select Skills"
date:   2018-07-30 21:34:34 +0800
author: allencharp
tags: [database, sql, tips]
---

# Introduction

Three practical SQL patterns I use frequently: composing dynamic `WHERE` clauses with optional parameters, pivoting rows into a single aggregate with `SUM(CASE ... END)`, and selecting the Nth value in an ordering.

# Optional parameters to compose the WHERE clause

If a stored procedure accepts optional parameters to build the WHERE clause, the user might provide any subset of the field values. The trick is to pair each filter with an `OR <column> IS NULL` condition, and wrap each pair in parentheses:

{% highlight sql %}
CREATE PROCEDURE GetNames
(
  @LastName VARCHAR(100),
  @Address  VARCHAR(100)
)
AS
SELECT * FROM NamesTable
WHERE (lastname = @LastName OR @LastName IS NULL)
  AND (address = @Address OR @Address IS NULL)
GO
{% endhighlight %}

How it works: when the caller supplies `@LastName`, the `OR @LastName IS NULL` branch is false, so the filter `lastname = @LastName` is enforced; when the parameter is NULL, the filter branch is ignored.

> Without the parentheses, `AND` binds tighter than `OR`, which silently changes the query logic (the classic "optional parameter" bug). Always parenthesize each `(col = @p OR @p IS NULL)` pair.

# SUM(CASE WHEN ... END) syntax

Use `SUM(CASE ... END)` to turn rows into columns (conditional aggregation). For example, to get the percentage of boys in a class:

{% highlight sql %}
SELECT (SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)) AS boy_percent
FROM studentTable;
{% endhighlight %}

`SUM(CASE ... THEN 1 ELSE 0 END)` counts the rows matching the condition. Multiplying by `100.0` (not `100`) keeps the division as floating point.

# Select the Nth highest salary

A simple way to find the Nth highest salary in a table:

{% highlight sql %}
SELECT MIN(EmpSalary)
FROM Salary
WHERE EmpSalary IN (
    SELECT TOP N EmpSalary FROM Salary ORDER BY EmpSalary DESC
);
{% endhighlight %}

This takes the top N salaries and returns the smallest of them. In SQL Server and modern databases, window functions are often cleaner:

{% highlight sql %}
SELECT EmpSalary
FROM (
    SELECT EmpSalary, ROW_NUMBER() OVER (ORDER BY EmpSalary DESC) AS rn
    FROM Salary
) t
WHERE rn = N;
{% endhighlight %}

`ROW_NUMBER()` also handles ties predictably (each row gets a distinct number).

# Best practice: parameterized queries

All the patterns above assume user input arrives through **parameters**. Never concatenate user input into SQL strings — use parameterized queries / prepared statements:

{% highlight python %}
# Python (psycopg2 / mysql-connector style)
cursor.execute("SELECT * FROM NamesTable WHERE lastname = %s", (lastname,))
{% endhighlight %}

Parameterized queries eliminate the entire class of SQL injection vulnerabilities and are mandatory for any application-facing SQL.

# Summary

Dynamic WHERE with `(col = @p OR @p IS NULL)` pairs, conditional aggregation with `SUM(CASE ... END)`, and `ROW_NUMBER()` for Nth-value queries cover a large share of everyday SQL needs — and always pair them with parameterized queries for security.
