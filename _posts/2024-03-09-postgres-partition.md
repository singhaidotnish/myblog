---
layout: post
title: Partitioning Tables in PostgreSQL
date: 2024-03-09 12:00:00 +0300
description: Partitioning Tables in PostgreSQL
img: partition/wood_posts.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [PostgresSQL, Partitioning]
---

Table partitioning is a powerful technique that involves breaking down a large table into smaller physical tables or partitions. This partitioning process is particularly advantageous when dealing with datasets where queries frequently target specific subsets of data rather than scanning the entire table. For instance, in multi-tenant SaaS (Software as a Service) applications, partitioning based on the tenant_id can significantly enhance performance. By partitioning the data according to tenants, each query can operate exclusively on the relevant subset of data, leading to faster response times. Additionally, partitioning allows for the isolation of each tenant's data, enabling efficient management such as archiving or deleting entire tenants without affecting others.

PostgreSQL currently supports three types of partitioning: **list**, **range**, and **hash**. In this blog post we will explore a hypothetical scenario involving the development of a multi-tenant SaaS application tailored for blogging. We'll delve into how each type of partitioning could be implemented within this application.

## No Partitioning

Without using partitioning all of our data will end up in one large table. When we query the table, PostgreSQL will need to scan the full table or utilize indexes that help filter down to one tenant.
```
CREATE TABLE tenants (
    identity  varchar(40),
    name      varchar(40),
    CONSTRAINT unique_identity UNIQUE (identity)
);

CREATE TABLE posts (
    tenant_identity  varchar(40),
    title            varchar(40),
    date             date,
    CONSTRAINT fk_tenent
      FOREIGN KEY(tenant_identity)
        REFERENCES tenants(identity)
);

```

We can insert data like normal without needing to think about the values for the tenants' identity.

```
INSERT INTO tenants (identity, name) VALUES
  ('TenantOne', 'Cool Blogger Biz'),
  ('TenantTwo', 'Blogger Inc.');


INSERT INTO posts (title, date, tenant_identity) VALUES
  ('post 1 from tenant 1', '2020-06-01', 'TenantOne'),
  ('post 2 from tenant 1', '2023-06-01', 'TenantOne'),
  ('post 1 from tenant 2', '2023-06-01', 'TenantTwo');

SELECT * FROM tenants;
 identity  |       name
-----------+------------------
 TenantOne | Cool Blogger Biz
 TenantTwo | Blogger Inc.
(2 rows)


SELECT * FROM posts;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
 TenantTwo       | post 1 from tenant 2 | 2023-06-01
(3 rows)

```


## List partitioning

**List partitioning** assigns each distinct value in the partitioning column to its own partition. This approach is ideal for scenarios like multi-tenant SaaS applications, where each tenant can reside in its own partition. List partitioning is particularly advantageous for applications with a relatively small number of tenants since the number of partitions scales linearly with the number of tenants.

```
CREATE TABLE posts (
    tenant_identity  varchar(40),
    title            varchar(40),
    date             date,
    CONSTRAINT fk_tenent
      FOREIGN KEY(tenant_identity)
        REFERENCES tenants(identity)
) PARTITION BY LIST(tenant_identity);
```
Before inserting data into the partitioned table we have to make sure the partition for a given tenant is created.

```
CREATE TABLE posts_TenantOne PARTITION OF posts FOR VALUES IN ('TenantOne');
CREATE TABLE posts_TenantTwo PARTITION OF posts FOR VALUES IN ('TenantTwo');

INSERT INTO posts (title, date, tenant_identity) VALUES
  ('post 1 from tenant 1', '2020-06-01', 'TenantOne'),
  ('post 2 from tenant 1', '2023-06-01', 'TenantOne'),
  ('post 1 from tenant 2', '2023-06-01', 'TenantTwo');
```

We are still able to select all the data from the partitioned data `posts` and we are able to select from any one partition directly so that the other partitions do not need to get scanned.

```
SELECT * FROM posts;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
 TenantTwo       | post 1 from tenant 2 | 2023-06-01

SELECT * FROM posts_TenantOne;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
(2 rows)
```

When using a  `WHERE`  clause that filters the data to one partition we can see that PostgreSQL's query planner will not bother scanning other partitions, saving us query time. In a multi tenant application most queries will have a `WHERE` clause specifying the tenant.

```
SELECT * FROM posts WHERE tenant_identity = 'TenantOne';
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
(2 rows)

EXPLAIN SELECT * FROM posts WHERE tenant_identity = 'TenantOne';
                               QUERY PLAN
------------------------------------------------------------------------
 Seq Scan on posts_tenantone posts  (cost=0.00..14.38 rows=2 width=200)
   Filter: ((tenant_identity)::text = 'TenantOne'::text)
(2 rows)
```

## Hash partitioning

**Hash partitioning** involves specifying the total number of partitions desired for the table. The partitioning column undergoes a hashing process, which is then modulo by the number of partitions to determine the partition for each row. This method is beneficial in multi-tenant SaaS applications with a large number of tenants, as the number of partitions remains static even as more tenants are acquired.

```
CREATE TABLE posts (
    tenant_identity  varchar(40),
    title            varchar(40),
    date             date,
    CONSTRAINT fk_tenent
      FOREIGN KEY(tenant_identity)
        REFERENCES tenants(identity)
) PARTITION BY HASH(tenant_identity);
```
Before inserting data into the partitioned table we have to make sure the partitions are created.
```
CREATE TABLE posts_0 PARTITION OF posts FOR VALUES WITH (modulus 4, remainder 0);
CREATE TABLE posts_1 PARTITION OF posts FOR VALUES WITH (modulus 4, remainder 1);
CREATE TABLE posts_2 PARTITION OF posts FOR VALUES WITH (modulus 4, remainder 2);
CREATE TABLE posts_3 PARTITION OF posts FOR VALUES WITH (modulus 4, remainder 3);

INSERT INTO posts (title, date, tenant_identity) VALUES
  ('post 1 from tenant 1', '2020-06-01', 'TenantOne'),
  ('post 2 from tenant 1', '2023-06-01', 'TenantOne'),
  ('post 1 from tenant 2', '2023-06-01', 'TenantTwo');
```

Like list partitioning we are able to query the full table to see all the data, and we are able to query specific partitions directly.
```
SELECT * FROM posts;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
 TenantTwo       | post 1 from tenant 2 | 2023-06-01
(3 rows)

SELECT * FROM posts_0;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
(2 rows)
```
When using a  `WHERE`  clause that filters to one tenant, the query planner will only target one partition. However the hashing partition allows more than on tenant per partition so we will still be filtering on other tenants' data.

```
SELECT * FROM posts WHERE tenant_identity = 'TenantOne';
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
(2 rows)

EXPLAIN SELECT * FROM posts WHERE tenant_identity = 'TenantOne';
                                QUERY PLAN
--------------------------------------------------------------------------
 Seq Scan on posts_0 posts  (cost=0.00..14.38 rows=2 width=200)
   Filter: ((tenant_identity)::text = 'TenantOne'::text)
(2 rows)
```
## Range partitioning

**Range partitioning** involves defining a starting and ending value for the partitioned column. This type of partitioning is particularly useful for managing data based on date ranges, facilitating the archival of old partitions and efficient data maintenance.

```
CREATE TABLE posts (
    tenant_identity  varchar(40),
    title            varchar(40),
    date             date,
    CONSTRAINT fk_tenent
      FOREIGN KEY(tenant_identity)
        REFERENCES tenants(identity)
) PARTITION BY RANGE(date);
```
Before inserting data into the partitioned table we have to make sure the partitions are created.
```
CREATE TABLE posts_y2020 PARTITION OF posts FOR VALUES FROM ('2020-01-01') TO ('2021-01-01');
CREATE TABLE posts_y2021 PARTITION OF posts FOR VALUES FROM ('2021-01-01') TO ('2022-01-01');
CREATE TABLE posts_y2022 PARTITION OF posts FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');
CREATE TABLE posts_y2023 PARTITION OF posts FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

INSERT INTO posts (title, date, tenant_identity) VALUES
  ('post 1 from tenant 1', '2020-06-01', 'TenantOne'),
  ('post 2 from tenant 1', '2023-06-01', 'TenantOne'),
  ('post 1 from tenant 2', '2023-06-01', 'TenantTwo');
```

Like previous forms of partitioning we are able to query the full table to see all the data, and we are able to query specific partitions directly.
```
SELECT * FROM posts;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
 TenantOne       | post 2 from tenant 1 | 2023-06-01
 TenantTwo       | post 1 from tenant 2 | 2023-06-01
(3 rows)

SELECT * FROM posts_y2020;
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
(1 row)
```

When using a  `WHERE`  clause that filters on a date range the query planner will only search the relative partitions, and aggregate the results together.
```
SELECT * FROM posts WHERE date > '2020-01-01' and date < '2022-01-01';
 tenant_identity |        title         |    date
-----------------+----------------------+------------
 TenantOne       | post 1 from tenant 1 | 2020-06-01
(1 row)

EXPLAIN SELECT * FROM posts WHERE date > '2020-01-01' and date < '2022-01-01';
                                  QUERY PLAN
-------------------------------------------------------------------------------
 Append  (cost=0.00..30.52 rows=4 width=200)
   ->  Seq Scan on posts_y2020 posts_1  (cost=0.00..15.25 rows=2 width=200)
         Filter: ((date > '2020-01-01'::date) AND (date < '2022-01-01'::date))
   ->  Seq Scan on posts_y2021 posts_2  (cost=0.00..15.25 rows=2 width=200)
         Filter: ((date > '2020-01-01'::date) AND (date < '2022-01-01'::date))
(5 rows)
```
