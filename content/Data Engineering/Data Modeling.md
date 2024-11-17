---
date: 2024-11-16
modified: 2024-11-17T22:40:51+01:00
---

Here are some notes on one of the most important topics in Data Engineering: **Data Modeling**. I collected these notes (an pictures too!) from different sources and will keep adding more as I use new ones. For now, the sources are:
* ***Fundamentals of Data Engineering*** book (by Joe Reis & Matt Housely)
* ***Deciphering Data Architecture*** book (by James Serra)
* [Learn Database Normalization - 1NF, 2NF, 3NF, 4NF, 5NF](https://www.youtube.com/watch?v=GFQaEYEc8_8&t=125s&ab_channel=Decomplexify) video on YouTube (by Decomplexify channel)
# Introduction
>[!definition]
> **Data Modeling** is a high-level conceptual technique used to design a database. This process involves identifying how the data must be structured and standardized to best reflect the organizations' processes, specifically how data that needs to be stored, and then creating a structured representation of that data and the relationships among the data.

Creating a coherent structure for data is a critical step to make data useful for the business and a well-constructed data architectures must reflect the goals and business logic of the organization that relies on this data.

Data Modeling has been used for decades in various forms:
+ some Normalization techniques since the creation of RDBMSs;
+ Data Warehousing modeling techniques since 1990s.

The lack of rigorous Data Modeling creates data swamps, along with lots of redundant, mismatched, or simply wrong data.
# Conceptual, Logical, and Physical Data Models
When modeling data, the idea is to move from **abstract modeling concepts** to **concrete implementations**. Following this flow, we can distinguish between three main concepts of data models:
+ **Conceptual Data Model**. It contains business logic and rules and describes the system's data, such as schemas, tables, and fields. It's helpful to visualize a conceptual model with the **Entity-Relationship** (**ER**) **Diagram**, which is a standard tool for visualizing the relationship among various entities in the data.
+ **Logical Data Model**. Details are added to the conceptual model, such as customer ID, customer names. In addition, we would map out primary and foreign keys.
+ **Physical Data Model**. It defines how the logical model will be implemented in a database system. We would add specific databases, schemas, and tables.

Another important consideration for data modeling is the **grain** of the data, which is the resolution at which data is stored and queried. Say you received a request for a report that summarized daily customer orders; specifically, the report should list all customers who ordered, the number of orders they placed that dat, and the total amount they spent. This report is coarse-grained because it contains no details, such as the cost per order or the items in each order, so the data engineer could think to ingest data from production database and reduce it to a report table with only the basic aggregate data needed for the report. However, this would result in the need to start over when the request for a report with a finer-grained data aggregation arrives. In general, the idea is to try to model data at the **lowest level of grain possible** because it's easy to aggregate this to coarse-grained report, but the reverse is not possible.
# Relational Modeling
**Relational modeling** was developed by Edgar F. Codd in 1970 and it's a data modeling technique used to design a database. It involves organizing the data into tables and defining the relationship between the tables. In Relational Databases and Relational Data Warehouses:
+ each **table** consists of rows (aka records or tuples) and columns (aka fields or attributes);
+ each **row** represents a unique instance of the data;
+ each **column** represents a specific piece of information about the data.
## Keys
In relational modeling relationship between tables, rows and columns are defined by:
+ **Primary Keys**. A primary key is a unique identifier for each record in a table, ensuring that no two rows of data have the same key value.
+ **Foreign Keys**. A foreign key is a column in a table that refers to the primary key in another table. It's used to establish a relationship between two tables in order to ensure the integrity of the data.

There is Natural Keys as well: a natural key is a field that already exists in a table and is unique to each row and it is usually used as the primary key because it represents something specific in the real world, such as a person's Social Security number or a product's serial number.
## Entity-Relationship Diagrams
Usually, you start relational modeling with an **Entity-Relationship** (**ER**) **diagram**: a high-level visual structure of the database that represents the *entities* (data) and the *relationship* between them. Here's an example:
![](Data%20Engineering/attachments/Pasted%20image%2020241117160636.png)

Once it's completed you can go ahead and move to Logical and Physical data modeling.
## Normalization
**Normalization** is a database data modeling practice that enforces strict control over the relationship of tables and columns within a database. Specifically, it's a way of decomposing a complex database into smaller, simpler tables with the purpose of minimizing redundancy and dependency, improving data integrity and makes the database more efficient and easier to maintain and manage.

This technique was first introduced by the pioneer of relational database Edgar Codd and he outlined four main objectives of normalization:
1. To free the collection of relations from undesirable insertion, update, and deletion dependencies.
2. To reduce the need for reconstructing the collection of relations, as new types of data are introduces, and thus increase the lifespan of application programs.
3. To make the relational model more informative to users.
4. To make the collection of relations neutral to the query statistics, where these statistics are liable to change as time goes by.

*(I just pasted these 4 points from the book, but I need to understand better one by one, maybe with some examples.)*

Let's list what Codd called **Normal Forms** and the conditions each of them needs to be satisfied:
+ **Denormalized**. No normalization, so nested and redundant data is allowed.
+ **First Normal Form (1NF)**:
	+ the table has a primary key (i.e. `StudentID` column in `Students` table);
	+ each attribute in the table contains a single value, not a list of values (i.e. `Name` column);
	+ the table has no repeating groups of columns (i.e. in the `Studends` table you should not have multiple "Course" columns, such as `Course1`, `Course2`, etc.). Instead, each student-course pairing should have its own row.
+ **Second Normal Form (2NF)**:
	+ all 1NF conditions;
	+ every detail (non-key attribute) in the database record must rely entirely on its unique identifier (primary key) and not on any other detail (i.e. in the `Students` table with `StudentID`, `Name`, and `Major` columns, the students' `Major` (non-key attribute) must be determined solely by the `StudentID` (primary key), not by the `Name` or any other attribute in the table). In other words, partial dependencies need to be removed.
+ **Third Normal Form (3NF)**:
	+ all 2NF conditions;
	+ every non-key detail in the table should relate directly to the main identifier (primary key) and not through another detail (i.e. in the `Students` table with `StudentID`, `Major` , and `DepartmentHead` (which is the head of the `Major`), the `DepartmentHead` should not depend on the `Major`, which in turn should not depend on the `StudentID`).

Most relational models, especially for **OLTP** databases, are in **3NF**.

A relational model uses *normalized database schema*, in which data is stored in one place and organized into multiple tables with strictly defined relationship between them. This helps to ensure **integrity**, but it can also make querying **more time-consuming** because the database will likely need to be join multiple tables together in order to retrieve the desired data.

Here's an example of a Relational Data Model for sales data:
![](Data%20Engineering/attachments/Pasted%20image%2020241117181143.png)
### Normalization from a Practical Perspective
This example is taken from [this video](https://www.youtube.com/watch?v=GFQaEYEc8_8&t=125s&ab_channel=Decomplexify) on YouTube.

Generally speaking, even a good database can't protect against bad data. However, a good database design can protect against some of them. These are cases where the data is telling us something that logically cannot possibly be true, such as a customer with two different dates of birth, which is logically impossible. It's what we might call a failure of **Data Integrity** and the data can't be trusted because it disagree with itself. When data disagree with itself, that's more than just a problem of bad data because it's a problem of **bad database design**, specifically it's what happens when a database design **isn't properly normalized**.

When you **Normalize a Database**, you structure it in such a way that can't express redundant information. For example, in a normalized table, you wouldn't be able to give two birth dates to the same customer. Normalized database tables are not only protected from contradictory data, they're also:
+ easier to understand
+ easier to enhance and extend
+ protected from insertion anomalies, update anomalies, deletion anomalies.

How do we determine whether a table isn't normalized enough? In other words, how do we determine if there's a danger that redundant data could creep into the table? It turns out that there are sets of criteria we can use to access the level of danger. These sets of criteria have names like *First Normal Form*, *Second Normal Form*, *Third Normal Form*, and so on, where each of these forms is given more stringent conditions than the previous one:
![](Data%20Engineering/attachments/Pasted%20image%2020241117190405.png)

Think of these normal forms as kind of **safety assessments**: if we discover that a table meets the requirements of first normal form, that's a bare minimum safety guarantee; if we further discover that the table meets the requirements of second normal form, that's an even greater safety guarantee, and so on.
#### First Normal Form (1NF)
There are various ways to violate First Normal Form (1NF).

Say you want to get the list of the Beatles member from the tallest to shortest: Paul, John, George, Ringo. Lists like this are totally comprehensible to us, but they're not normalized: remember, there's no such thing as **row order** within a relational database table. So, here we have our first *violation of First Normal Form*.
>[!attention] Violation of 1NF: Using Row Order to convey information
> When we use row order to convey information, we're violating First Normal Form.

The solution is very simple: be explicit by dedicating a separate column to this information:
![](Data%20Engineering/attachments/Pasted%20image%2020241117193218.png)

A second way of violating 1NF involves **mixing data types**. Suppose our database has data like:
![](Data%20Engineering/attachments/Pasted%20image%2020241117193401.png)

If you use spreadsheets, they typically will not stop you from having more than one datatype within a single column, but in a relational database you're not allowed to be ambiguous about a column's data type, so you cannot have, for example, a mix of strings and integers values within a single column. Once you defined a column as an integer column, then every value that goes into that column will be an integer - no string, no timestamps, no any other data types other than integers.
>[!attention] Violation of 1NF: Mixing data types within the same column
> Once you defined a column as an integer column, then every value that goes into that column will be an integer - no string, no timestamps, no any other data types other than integers. Indeed, the DB platform won't let you do it anyway.

A third way of violating 1NF is by designing a **table without a primary key**. A Primary Key is a column, or a combination of columns, that uniquely identifies a row in the table. For example, in the `Beatle_Height` table, our intention is that each row should tell us about one particular Beatle, so we designate `Beatle` column as the primary key of this table. The DB platform will need to know about our choice of primary key, so we'll want to get the primary key into the database by doing:
```sql
ALTER TABLE Beatle_Height
ADD PRIMARY KEY (Beatle);
```

>[!attention] Violation of 1NF: Having a table without Primary Key
> With the primary key in place, the database platform prevents multiple entries for the same Beatle because multiple rows for the same Beatle would be nonsensical and contradictory.

The last way of failing to achieve 1NF involves the notion of "***repeating groups***". Suppose we're designing a database for an online multiplayer game. At a given time, each player has a number of items of different types, like arrows, shields, and copper coins. We might represent the situation like this:
![](Data%20Engineering/attachments/Pasted%20image%2020241117203958.png)
Each `Inventory` entry can contains potentially many different types of items and that's what we call "repeating group". We could design a database table that represents the `Inventory` as a string of text:
![](Data%20Engineering/attachments/Pasted%20image%2020241117204431.png)
However, this is a terrible design because there's no easy way of querying it (i.e. if we want to know which players currently have more than 10 copper coins, it would be very impractical to write a query that gives us the answer).

We might be tempted to represent the data like this:
![](Data%20Engineering/attachments/Pasted%20image%2020241117204523.png)
This is still a bad choice because a player might have hundreds of different types of items, so a table would have hundreds of columns.
>[!attention] Violation of 1NF: "*Repeating groups*"
> Storing a repeating group of data items on a single row violates 1NF.

Here's a design that respects 1NF:
![](Data%20Engineering/attachments/Pasted%20image%2020241117204827.png)

With this design, because each row in the table tells us about one unique combination of `Player_ID`-`Item_Type`, the **Primary Key** is the combination of `Player_ID`-`Item_Type`.
#### Second Normal Form (2NF)
Let's look again at the `Player_Inventory` table:
![](Data%20Engineering/attachments/Pasted%20image%2020241117205207.png)

This table is fully normalized, but suppose we want to enhance the table slightly. Let's image we want to record the current rating (beginner, intermediate or advanced) of each player. To do so, we simply include in the table an extra column called `Player_Rating`:
![](Data%20Engineering/attachments/Pasted%20image%2020241117205950.png)
We have some reasons why this is not a good design:
+ `jdog21` and `trev73` have more than one rows in the table, so all of them are marked with the same `Player_Rating` value, resulting in redundant information.
+ Suppose `gila19` player loses all her copper coins leaving with nothing in her inventory:
	![](Data%20Engineering/attachments/Pasted%20image%2020241117211030.png)
	The unique entry for `gila19` is now gone. If we try to query the database to find out what is her rating, we can no longer access this information. This problem is known as **Deletion Anomaly**.
+ Suppose `jdog21` improves his rating from intermediate to advanced and to capture this new information, we run an update on his two records. But let's imagine the update goes wrong and, by accident, only one of his records gets updated, and the other record gets left alone:
	![](Data%20Engineering/attachments/Pasted%20image%2020241117210944.png)
	This player has now both intermediate and advanced rating at the same time. This problem is called **Update Anomaly**.
* Suppose a new player comes along and she's a beginner and she doesn't have anything in her inventory yet. We want to record the fact that she's a beginner but, because she has nothing in her inventory, we can't insert this player into the table. So, her rating goes unrecorded. This problem is known as **Insertion Anomaly**:
	![](Data%20Engineering/attachments/Pasted%20image%2020241117211327.png)

The reason our design is vulnerable to this problems is that isn't in **Second Normal Form** (**2NF**). This normal form is about how a table's *non-key columns* (or *non-key attributes*) relate to the primary key. In our table, the non-key columns are `Item_Quantity` and `Player_Rating`:
![](Data%20Engineering/attachments/Pasted%20image%2020241117212156.png)

The definition we're going to give to 2NF is an informal one, which leaves out some nuances - but for most practical purposes, that shouldn't matter.

> [!info] Second Normal Form (2NF)
> Informally, the **Second Normal Form** (**2NF**) says that **each non-key attribute in the table must be dependent on the entire primary key**.

Let's analyze our non-key attributes:
+ does `Item_Quantity` depend on the entire primary key? Yes, because an `Item_Quantity` value represents a specific `Item_Type` owned by specific `Player_ID`. We can express is like this:
	![](Data%20Engineering/attachments/Screenshot%202024-11-17%20at%2021.28.08.png)
	where the arrow signifies a **dependency**, or better a **functional dependency**. This simply means that each value of the thing on the left of the arrow is associated with exactly one value of the thing on the right side of the arrow. As far as 2NF is concerned, this dependency is fine because it's a dependency on the entire primary key.
+ does `Player_Rating` depend on the entire primary key? No, because it's a property of `Player_ID` only. In other words, for any given player, there's one `Player_Rating`:
	![](Data%20Engineering/attachments/Pasted%20image%2020241117213433.png)
	This dependency on `Player_ID` is the problem because `Player_ID` is part of the primary key, but it's not the whole key. That's why the table isn't in Second Normal Form and why it's vulnerable to problems.

The design of this database went wrong when we chose to add a `Player_Rating` column to a table where it didn't really belong. The fact that a `Player_Rating` is a property of a player should have helped us to realize that a player is an important concept in its own right. So, `Player_ID` deserves its own table, which will contain one row per player, and in it we can include as columns the ID of the player, the rating of the player, the rating of the player, and so on:
![](Data%20Engineering/attachments/Pasted%20image%2020241117214532.png)

The previous `Player_Inventory` table can stay as it was. For both tables, we can say that there are no part-key dependencies. In other words, it's always the case that every attribute depends on the whole primary key, not just part of it:
![](Data%20Engineering/attachments/Pasted%20image%2020241117214734.png)

**These two tables are in Second Normal Form** (**2NF**).
#### Third Normal Form (3NF)
Suppose we decide to enhance the `Player` table with a new column `Player_Skill_Level`. Image that in this particular multiplayer game, there's a nine-point scale for skill level (level 1 as beginner and at the opposite extreme level 9 as expert) and let's say we've defined exactly how `Player_Skill_Level` relates to `Player_Rating`:
![](Data%20Engineering/attachments/Pasted%20image%2020241117220025.png)

This is the updated table:
![](Data%20Engineering/attachments/Pasted%20image%2020241117220104.png)

In this updated table a problem can arise. Let's say tomorrow, player `gila19`'s skill level increases from 3 to 4. If that happens, we'll update her row the `Player_Skill_Level` column and we should also update her `Player_Rating` as well from beginner to intermediate. However, suppose that something goes wrong and we fail to update the `Player_rating`: now, we've got a data inconsistency where `gila19` is still a beginner but has 4 as skill level:
![](Data%20Engineering/attachments/Pasted%20image%2020241117220546.png)

How did the design allow this happens? Second Normal Form didn't flag up any problems because there's no non-key attribute that depends only partially on the primary key. But, in what way are they dependent on it? Let's look more closely:
+ `Player_Skill_Level` is dependent on `Player_ID`:
	![](Data%20Engineering/attachments/Pasted%20image%2020241117221027.png)
+ `Player_Rating` is dependent on `Player_ID` too, but only indirectly:
	![](Data%20Engineering/attachments/Pasted%20image%2020241117221100.png)
	A dependency of this type is called **Transitive Dependency**: `Player_Rating` depends on `Player_Skill_Level`, which in turn depends on the primary key `Player_ID`.

The problem is just located in this transitive dependency because what **Third Normal Form** **forbids** is exactly **Transitive Dependency**: the dependency of a non-key attribute on another non-key attribute. Based on that, `Player` table is not in 3NF. The way of making this table in 3NF is to remove `Player_Rating` from `Player` table and we introduce a new table called `Player_Skill_Levels` that tells us everything we need to know about how to translate a player skill level into a player rating:
![](Data%20Engineering/attachments/Pasted%20image%2020241117221843.png)

> [!info] Third Normal Form (3NF)
> **Third Normal Form** (**3NF**) says that **every non-key attribute in a table should depend on the key, the whole key, and nothing but the key**.

If you keep these rules in ming, then the 99% of the time you will end up with **Fully Normalized Tables**. It's even possible to shorten this normal form by knocking out the phrase "*non-key*". This represents a slightly stronger flavour of 3NF known as **Boyce-Codd Normal Form** (**BCNF**):
> [!info] Boyce-Codd Normal Form
> **Third Normal Form** (**3NF**) says that **every attribute in a table should depend on the key, the whole key, and nothing but the key**.

In practice the difference between BCNF and 3NF is extremely small and the chances of you ever encountering a real-life 3NF table that doesn't meet BCNF are almost zero. Any such table would have to have what we call multiple overlapping candidate keys - which gets us into realms of obscurity and theoretical rigor that are little beyond the scope of this lesson. So, as a practical matter, just follow the guideline of **Boyce-Codd Normal Form** and you can be confident that the table will be in both 3NF and BCNF.

In almost all cases, once you've normalized a table this far, you've **Fully Normalized** it. There are some instances where this level of normalization isn't enough and these rare instance are dealt with by **Fourth** and **Fifth Normal Form** (**4NF** and **5NF**).