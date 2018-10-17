---
layout: post
title: Helpful psql commands
date: 2018-09-01 12:00:00 +0300
description: top 5 useful psql commands
img: postgres.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [PostgreSQL, Rails]
---

As a Ruby on Rails developer I often find myself using the `rails db` tool in order to troubleshoot issues with the application. Since my Rails application is using PostgreSQL `rails db` drops me into a `psql` session. Here are the top five `psql` commands that I have found helpful throughout the years.

## 1. The help commands: `\h`

Using this command you can get the documentation for all SQL commands. This can be helpful if you want to make sure you have the syntax correct for any SQL you are writing.

```
postgres=# \h CREATE INDEX
Command:     CREATE INDEX
Description: define a new index
Syntax:
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ [ IF NOT EXISTS ] name ] ON table_name [ USING method ]
    ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )
    [ WITH ( storage_parameter = value [, ... ] ) ]
    [ TABLESPACE tablespace_name ]
    [ WHERE predicate ]
```

## 2. The describe command: `\d`

This command can be used to describe table, view, sequence, or index. It can also be used by itself to list **all** tables, views, and sequences. It is helpful while exploring the schema. In this example I have one table named `dawgs`, and I use the `\d` command to describe all relations in my database. Then I use it again to specifically describe the `dawgs` table.

```
postgres=# \d
              List of relations
 Schema |     Name     |   Type   |  Owner
--------+--------------+----------+----------
 public | dawgs        | table    | postgres
 public | dawgs_id_seq | sequence | postgres
(2 rows)

postgres=# \d dawgs
                                   Table "public.dawgs"
 Column |         Type          | Collation | Nullable |              Default
--------+-----------------------+-----------+----------+-----------------------------------
 id     | integer               |           | not null | nextval('dawgs_id_seq'::regclass)
 name   | character varying(40) |           |          |
Indexes:
    "dawgs_pkey" PRIMARY KEY, btree (id)
```

## 3. Toggle expand output command: `\x`

This is an extremely helpful command. It toggles the output from being printed as a table to being printing one record at a time. This might not sound like a big deal, but once you have enough data to wrap your terminal the data becomes almost impossible to understand without turning on the 'Expanded' toggle.

```
postgres=# \x
Expanded display is off.
postgres=# SELECT * FROM dawgs LIMIT 3;
 id |      name
----+----------------
  1 | Dawg number: 1
  2 | Dawg number: 2
  3 | Dawg number: 3
(3 rows)

postgres=# \x
Expanded display is on.
postgres=# SELECT * FROM dawgs LIMIT 3;
-[ RECORD 1 ]--------
id   | 1
name | Dawg number: 1
-[ RECORD 2 ]--------
id   | 2
name | Dawg number: 2
-[ RECORD 3 ]--------
id   | 3
name | Dawg number: 3

postgres=#
```

## 4. Toggle timing command: `\timing`

This is an easy one. When timing is turned on `psql` will print how long each command took to run. It is nice while looking at the performance of your SQL.

```
postgres=# \timing
Timing is on.
postgres=# SELECT * FROM dawgs LIMIT 1;
 id |      name
----+----------------
  1 | Dawg number: 1
(1 row)

Time: 0.370 ms
```

## 5. List and connect to database commands: `\l` and `\c`

These commands are helpful if you use multiple databases. As a Rails developer I often have a 'development' database and a 'test' database running locally so that I can keep test and development data separated. We can use the `\l` and `\c` commands to list all databases and change our connection to a different database.

```
postgres=# \l
                                   List of databases
     Name      |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
---------------+----------+----------+------------+------------+-----------------------
 postgres      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres_test | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 template1     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \c postgres_test
You are now connected to database "postgres_test" as user "postgres".
postgres_test=#
```

## Bonus command: `\?`

The `\?` command shows the help text for all backslash commands in `psql`. So if you forget any of the commands we reviewed today you can always use `\?` to find them again.
