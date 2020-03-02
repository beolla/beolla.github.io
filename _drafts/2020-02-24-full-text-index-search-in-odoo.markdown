---
layout: post
title:  "Full text search in Odoo Ecommerce Website"
date:   2020-02-29 00:16:33 +0700
categories: odoo, search, performance, postgresql
---

Helping customers find the right products quickly is crucial to running a successful ecommerce website. Therefore, product searching deserves attention to its performance. Odoo, a batteries included solution to managing enterpries, comes with its own off-the-shelf search functionality. It works out of the box with reasonable speed for small inventory. For larger product collection, it leaves something to be desired. In this post, we are looking at how Odoo is implementing its search function and possible methods to speed it up. 

INSERT SCREENSHOT OF PRODUCT SEARCH HERE.

### 1. How odoo is doing search currently

When an user enters a query into Odoo's ecommerce search box, Odoo is using Sql's `ilike` operator to find relevant products. `ilike` does case-insensitive substring matching. Products that contain the search query in theirs name or description are returned. To increase search recall, i.e returns as many related results as possible, Odoo tokenizes a search query into words and do `ilike` match for each word separately. That way, positions of matched words in resulted products' names can be flexible. A query of "Apple keyboard" will be able to match with a product named "Apple wireless keyboard".  

In order to find the result, Postgres would need to run pretty exhaustive search. It would look through every product in store, and for each loop through `name` and `description` fields to find matched substrings. Needless to say, this approach will be more costly as database grows.

Let say an user searches for `office lamp`, the equivalent sql query is roughly as below. (for ease of reasoning, this version of the sql query is simplified)

```sql
select id from product_template where name ilike '%office%' and name ilike '%lamp%'
```

Running the same query through Postgres query planner, surprisingly, a sequential scan is done on `product_template` table, even though there exists an name_index on `name` column.

![Sequential scan is required to search for products](/content/images/seq_scan_search.png)

INSERT SCREEN SHOT OF EXISTING INDEX

Turns out, the current index is of BTree type, which is only made use of if the search operator `like` and query is matched at the beginning of the text. This is understandable, given how BTree is implemented. A record would be added to BTree index by comparing it lexicographically with existing nodes. Lexicographic order starts from the beginning. A little modification to the above query would allow Postgres to use the existing index:

```sql
select id from product_template where name like 'office%' and name like 'lamp%'
```

![BTree index is used to search for products](/content/images/btree_scan.png)

Though using BTree index significantly improves search time, only matching queries at the beginning of a product's name is not helpful. Let see if we can do better. 

### 2. Speed up using index

PosgresSQL comes with different types of indices out of the box. The default index created by `CREATE INDEX` is BTree. In order to overcome the limitation of BTree, which only matches at start of the string, we will try to create index of GIN or GIST type. 

Let first try with GIN

THEN GIST

### 3. Full-text search.

Bes

Show query planner here 
What is full text search. 

How does it help 

Performance comparison





