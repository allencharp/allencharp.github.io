---
layout: post
title:  "SQL Select Skills"
date:   2018-07-30 21:34:34 +0800
categories: jekyll update
---

# optional parameter to compose where parameter
If our store procedure accepts some parameters to compose the SQL where clause, user might enter any of the field values all are optional.
How to write the where clause with optional parameters

{% highlight sql %}
CREATE PROCEDURE
{
@LastName VARCHAR(100),
@Address VARCHAR(100)
}
as
select * from TBS_Names 
where lastname = @LastName or @LastName is null 
and address = @Address or @Address is null
go
{% endhighlight %}

# sum(case(..)) syntax
How to select the rate of girls in one class ?
{% highlight sql %}
select (sum(case when gender="M" then 1 end) / count(*)) as 'num' 
from studentTable;
{% endhighlight %}