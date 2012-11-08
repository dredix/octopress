---
layout: post
title: "Concatenating strings and other quirks of DB2 for i SQL"
date: 2012-11-09 00:10
comments: true
categories: 
 - SQL
 - iSeries
 - DB2
---
If you have ever used more than one database engine for any length of time you may be probably aware of the <s>not always</s> subtle syntax differences in their implementation of SQL. (Not that there's any reason for that. [That's what standards are for, right?](http://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=53681)).

In this regard, DB2 for i SQL does not disappoint. Even a simple search for the official online documentation can be daunting for a newcomer ([it's here, by the way](http://publib.boulder.ibm.com/infocenter/iseries/v6r1m0/index.jsp?topic=/db2/rbafzintro.htm)). If you are learning to program on iSeries, you might have been grateful when you found out there was support for SQL, until you actually started using it and stumbled upon some of those little but significant annoyances that tend to slow progress down to a crawl.

The good news is that most of what you already know from a different DB engine will work here. For the rest, I started compiling a simple list to get you through some of the most common, repetitive and especially non-obvious tasks. I plan to go back and continue expanding this list as I keep learning how to effectively use the query language on this platform (I may never come back, so [don't just sit around waiting.](http://i2.kym-cdn.com/photos/images/original/000/159/324/flash11752_Still-Waiting-on-OP.jpg))

### Concatenate strings

This one is deceptively simple, but there's a twist (so soon?). You can use the [<code>CONCAT</code> function](http://publib.boulder.ibm.com/infocenter/iseries/v6r1m0/index.jsp?topic=/db2/rbafzscaconcat.htm) or the [concatenation operator](http://publib.boulder.ibm.com/infocenter/iseries/v6r1m0/index.jsp?topic=/db2/rbafzscaconcat.htm), with different syntax (but ultimately yielding the same result) depending on the chosen form. Here are the three valid ways of concatenating strings:

Using the <code>CONCAT</code> function:

    SELECT CONCAT(CONCAT(FIRSTNAME, ' '), LASTNAME) AS FULLNAME FROM STUDENTS

Using the long form of the operator:

    SELECT FIRSTNAME CONCAT ' ' CONCAT LASTNAME AS FULLNAME FROM STUDENTS

Using the short form of the concatenation operator:
    SELECT FIRSTNAME || ' ' || LASTNAME AS FULLNAME FROM STUDENTS

Yes, <code>CONCAT</code> is a function AND an operator. It may look weird, but it works. I tend to prefer the compact syntax, as long as I'm not [writing C code](http://www.acm.uiuc.edu/webmonkeys/book/c_guide/1.5.html) at the same time.

### Create a table with the same data structure as an existing table or view

I wouldn't be surprised if the group of [CREATE](http://publib.boulder.ibm.com/infocenter/iseries/v6r1m0/topic/db2/rbafzhctabl.htm) statements were the most complex on SQL for i, so it would be understandable if, during your perusal of the official reference, you missed the syntax for copying the column definitions from an existing table or view to a new table. So here it is:

	CREATE TABLE MYLIB/STUDENTS_ARCHIVE LIKE PRDDTA/STUDENTS 

Simple, easy to remember. And yet I forget about it all the time (hence its inclusion on this list).

### Retrieve top N records

Retrieving the first N records of a query seems to be the quintessential I-don't-care-how-you-did-it-I'll-do-it-on-my-own-way query amongst DB engines. In SQL Server, for example, you have the [<code>TOP</code> clause](http://technet.microsoft.com/en-us/library/ms189463.aspx):

    SELECT TOP 100 * FROM STUDENTS

In Oracle you are supposed to use the [<code>ROWNUM</code> pseudocolumn](http://docs.oracle.com/cd/B19306_01/server.102/b14200/pseudocolumns009.htm) (as long as you don't need ordered results, in which case a nested query is necessary):

    SELECT * FROM STUDENTS WHERE ROWNUM < 100;

DB2 FOR i SQL gets the prize for the most verbose syntax:

    SELECT * FROM STUDENTS FETCH FIRST 100 ROW ONLY 

And so on and so forth. You get the point.

### Delete duplicate records

I spent a lot of time on this problem, until I remembered it's 2012 and these days you can Google your way out of any difficult situation. The issue was simple enough: A badly written program inserted duplicate records on a table lacking a unique identifier, and we needed to remove those duplicates via SQL. The solution was kindly provided by Vijayakumar Kannan on his appropriately titled ['Delete duplicate records using SQL in iSeries 400'](http://search400.techtarget.com/tip/Delete-duplicate-records-using-SQL-in-iSeries-400) post:

    DELETE FROM STUDENTS A WHERE RRN(A) > 
        (SELECT MIN(RRN(B)) FROM STUDENTS B WHERE
         A.FIRSTNAME = B.FIRSTNAME AND A.LASTNAME = B.LASTNAME AND
         A.DAYOFBIRTH = B.DAYOFBIRTH AND ...)

where [<code>RRN</code>](http://publib.boulder.ibm.com/infocenter/iseries/v6r1m0/topic/db2/rbafzscarrn.htm) is a function that "returns the relative record number of a row".

So there you have it. I like to think that just by having stopped to write this post I probably committed those snippets to memory, but the reality is that I will probably not remember anything the next time I need help. 

Finally if you are reading this, hi future me. Don't forget to leave a comment for future future me.
