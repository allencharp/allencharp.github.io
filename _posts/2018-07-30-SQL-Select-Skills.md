---
layout: post
title:  "SQL Select Skills"
date:   2018-07-30 21:34:34 +0800
categories: jekyll update
---

# optional parameter to compose where condition
If our store procedure accepts some parameters to compose the SQL where clause, user might enter any of the field values all are optional.
How to write the where clause with optional parameters

{% highlight sql %}
CREATE PROCEDURE
{
@LastName VARCHAR(100),
@Address VARCHAR(100)
}
AS
SELECT * FROM NamesTable 
WHERE lastname = @LastName OR @LastName IS null 
AND address = @Address or @Address IS null
GO
{% endhighlight %}

# sum(case(..) end) syntax
Suppose to use a SELECT to get the percentage of boys rate in one class ?

{% highlight sql %}
SELECT (SUM(CASE WHEN gender="M" THEN 1 END) / count(*)) AS 'num' 
FROM studentTable;
{% endhighlight %}

# select Nth high salary in the office
{% hightlight sql %}
SELECT MIN(EmpSalary)
FROM Salary
WHERE EmpSalary IN(SELECT TOP N EmpSalary FROM Salary ORDER BY EmpSalary DESC) 
{% endhighlight %}