---
layout: post
title:  "SQL Select Skills"
date:   2018-07-30 21:34:34 +0800
author: allencharp
tags: [database, sql, tips]
---

## Optional parameters to compose the WHERE clause

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

> Without the parentheses, `AND` binds tighter than `OR`, which can silently change the query logic.

## SUM(CASE WHEN ... END) syntax

Use `SUM(CASE ... END)` to turn rows into columns. For example, to get the percentage of boys in a class:

{% highlight sql %}
SELECT (SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)) AS boy_percent
FROM studentTable;
{% endhighlight %}

## Select the Nth highest salary

A simple way to find the Nth highest salary in a table:

{% highlight sql %}
SELECT MIN(EmpSalary)
FROM Salary
WHERE EmpSalary IN (
    SELECT TOP N EmpSalary FROM Salary ORDER BY EmpSalary DESC
);
{% endhighlight %}
