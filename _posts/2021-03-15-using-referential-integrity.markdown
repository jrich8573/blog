---
layout: post
title:  "Using Referential Integrity"
date:   2021-03-15 
categories:  
tags: featured 
image: /assets/article_images/2021-03-15-using-referential-integrity/db-rf.png
---
I am not sure about about you, but tax season is a busy time of year for my teams. With that, I have jumped in the mix to assist with code reviews, `PR` approvals and branch merging in order to free up some my `Senior Data Engineers` to do more time sensitive `ad hoc` work for the senior leadership. The one treading issue that I constantly see within my code reviews is the lack of `Referential Integrity` between database tables. For any in the [data management](https://en.wikipedia.org/wiki/Data_management) field, using [primary keys](https://en.wikipedia.org/wiki/Primary_key) are simply business as usual. However, the use of [foreign keys](https://en.wikipedia.org/wiki/Foreign_key) are, well, foreign. 
 

So that we are on the same page, let me start by defining referential integrity; how one might use referential integrity and then I provide some examples of referential integrity to drive home the point. 

Before we dive into the world of Referential integrity we must first discuss the premise of a `foreign key`. Also, one note, I loth capital letters in `SQL` (kind of an oxymoron since I just spelled `sql` with all caps!). Anytime I am writing `SQL`, say in `Oracle`, because it is case sensitive for quoted strings, I run run the following oneliners

```sql
-- ignore case
alter session set nls_comp=linguistic;
alter session set nls_sort=binary_ci;
set sqlblanklines on; 
```

Then I can query the database like this

```sql
select 
    student_last_name
   ,student_first_name
   ,student_id
   ,student_department
from students
where student_department = 'computer science'; 
```
Now that is out of the way, let get down to brass tax. What is a `foreign key`? According to my friends at `Wikipedia`, in the context of relational datbases, a foreign key is a set of attributes in a table that refer to the primary key of another table. Thus, a foreign key links two table. Said another way, a `foreign key` is a set of attributes subject to a certain kind of inclusion dependency constraints, specifically a that the `tuple` consisting of a `foreign key` attribute in one relation, R, must also exist in some other (not necessarily distinct) relation, S, and furthermore that those attributes must also be a `candidate key` (will discuss in a later post) in S. Therefore, a `foreign key` is set of attributes that __reference__ a candidate key. I should note at this point that neither the `foreign key` nor the `primary key` have to be names the same, the only requirement is the column (element or field) must contain the same values. Therefore, if student table contain the following student(student_last_name, student_first_name, student_id, student_department) and within those columns contain the following `Jordan, Michael, 62345,computer science `, and the student(student_id) is referenced by the department table(member_id), then said department table must contain the the member_id (62345). This conditional is the primary requirement for referential integrity.  

So, `foreign keys` tie two tables together, where the table containing the `foreign key` references the table containing the `primary key`. So how does link back to `referential integrity`? Glad you asked! According to [David Kroenke, David Auer, Scott Vandenberg, & Robert Yoder](https://www.amazon.com/gp/product/0135188148/ref=ppx_yo_dt_b_asin_image_o07_s00?ie=UTF8&psc=1), `referential integrity` is a constraint used within relational database to ensure that every value of a `foreign key` matches a value of the referenced tables `primary key`. To my earlier example, if 'student_id 62345' did not exist within the department table as `member_id`, the database execution plan (or policy depending on your database flavor) will through some form of a referential integrity constaint error. This is significant issue if your data model is designed with a relationship between to the student table and the deparment. 

The easiest and most straightforward way to create referential integrity within a database is to declare it withint he table definition. For reference, I am creating these tables within my `MySQL` dev environment within the latest official `MySQL` image from [Docker Hub](https://hub.docker.com/_/mysql). In a later post I will discuss how to create your own `MySQL` dev environment using `Docker`. 

Given two tables, 
-   tbl_fact_department
-   tbl_fact_student

```sql
-- create_department.sql
create table if not exists tbl_fact_departments(
            id smallint unsigned  not null auto_increment primary key
           ,dname varchar(30)
);
```

```sql
-- create_student.sql
create table if not exists tbl_fact_student(
        uin smallint unsigned not null auto_increment primary key
       ,sname varchar(50)
       ,depart_id smallint unsigned
       ,program ENUM('UG', 'MS', 'PHD')
       constraint fk_dpt_id foreign key (depart_id)
       references tbl_fact_department (id)
);
```

Reasonly straightforward. One thing to note here, when creating the tables above, you must create the `tbl_fact_department` table first. If it does not exist, and attempt to create the `tbl_fact_student` table, the execution plan will trough an error, and your table will not be created. 

Alternatively, if you realize after creating and populating a table that a `foreign key` is needed, you can simply run an `alter table` statement (I am assuming no foreign ket exist within the table):
```sql
alter table tbl_fact_student add constraint fk_dpt_id foreign key(depart_id) references tbl_fact_department(id);
```

Personally, I prefer the former, if for nothing else it makes the "true" table definition readable to anyone that may need to reuse or alter the code. Secondly, it may the table definition super user friendly. 

Well that's it for now. I hope you enjoyed reading about referential integrity. If you have specific things you want to read about, drop me a line on either [GitHub](https://github.com/jrich8573),  [slack](nadeblg.slack.com) or on [Twitter](https://twitter.com/realjasonrich).   


Thank you for reading. 

Cheers!         
Jason 