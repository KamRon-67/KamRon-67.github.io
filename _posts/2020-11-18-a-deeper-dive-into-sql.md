---
title: "A Deeper Dive Into SQL"
description: "We are going to talk about some of the inner workings of sql"
tags: [SQL, Microsoft, SQL Server, Relational Model]
categories: [Blogging]
excerpt_image: https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/pexels-gilberto-olimpio.jpg
---

To me, SQL was a strange language at first. The basics will get you far. I am the kind of person who needs to know how the insides of something works. Tends to help me utilize the tool better sometimes. For the longest, I was able to do simple selects, queries, updates, queries, and some joins. That was more than enough to get my common daily tasks done.  

The longer I worked in development, the expectations of my skills increased as well. I quickly found out working with relational databases required a different thought process than my frontend work or server-side middleware work did. When I found out breakpoints did not stop the query and show all of the information at hand. I figured it was time to learn SQL. So, when in doubt, always go back to the basics. I went all the way back to my college training. I wanted to know the full SQL process.  

I did not go over Relational Algebra or Relational Calculus. That seemed too technical. Learning the life cycle of a SQL query seemed like a good start.   

The SQL query logical processing order  
The query processing order has a unique flow as it is not top-down like a lot of other languages. The order is as follows 

1. From (Source Data) 
2. Where (Row filters) 
3. Group By (Grouping) 
4. Having (Group Filter) 
5. Select (Return Expressions) 
6. Order By (Presentation Order) 
7. Offset Fetch (Paging) 

This was my first step in understanding the flow of a query and how to get the correct data back. Moving away from ad hoc SQL query writing was a breath of fresh air. Now I had gotten the foundation laid out. The next thing on my list was joining.  

## How Joins work under the hood 

Cartesian products 
Every join begins with a cartesian product. A cartesian product is where each row from one set is paired with each row from the other. This will give you a [set](https://www.sqlshack.com/mathematics-sql-server-fast-introduction-set-theory/) that consists of all columns from both sources.  

Qualified joins 
Inner and outer joins are both qualified joins. The qualification predicate is specified using the ON keyword. So, each row from the cartesian product is evaluated using the predicate. Only when the predicate evaluates to true will move on. All the other rows are eliminated from the set. With inner joins, the processing stops at this point and the rows are passed to the next clause. If the requested join type is an outer join, the qualified rows move on to the reservation stage. With outer joins, one or both source sets are designated as reserved. Reserved sets get to have all of the rows added to the join even those that did not pass the qualification step. So, an outer join reserves the set on the left side. So, the values on the left come back, and the nulls are placed on the right side. With right outer joins the same thing happens just flipped. Full outer joins simply reserve both sets. (NOTE this may be off or under-explained but it's the way I worded it for myself) A more correct way would be to say Right and Left joins are different in the data that is displayed based on the ON keyword. Left joins will use the left table and empty values from the right side show as null. Vice versa for the Right joins. What I was able to find out was using the left outer join as the example. A left outer join reserves the set on the left side. here it is set a, so all its rows that failed to qualify are reintroduced back into the join result since these rows had no match in set b there is no value to show for it and a null indicator is used for this inapplicable data. 

## Join Order

When joining multiple data sets, if you are exclusively working with inner joins, the order they are joined doesn't matter. This is not the case with outer joins, the order matter. To play with join the order we have the concept of [chiastic order](https://www.itprotoday.com/sql-server/take-control-joins). 
 


