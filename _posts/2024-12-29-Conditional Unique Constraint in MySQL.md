---
title: Conditional Unique Constraint in MySQL
date: 2024-12-29 23:52:00 +0800
categories: [ Technology ]
tags: [ mysql ]  
---

Assume each user must have a unique username, which can be reused after it has been deleted.

## 1. Create Database and Table

```shell
mysql> create database test;
Query OK, 1 row affected (0.01 sec)

mysql> use test;
Database changed

mysql> CREATE TABLE Persons (
    ->     UserName int NOT NULL,
    ->     LastName varchar(255) NOT NULL,
    ->     FirstName varchar(255),
    ->     IsDeleted tinyint
    -> );
Query OK, 0 rows affected (0.08 sec)

mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| Persons        |
+----------------+
1 row in set (0.01 sec)
```

## 2. No Unique Constraint

Successfully insert 2 same entries.

```shell
mysql> INSERT INTO Persons(UserName, LastName, IsDeleted) VALUES('001', 'name-1', 0),('001', 'name-1', 0);
Query OK, 2 rows affected (0.04 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM Persons;
+----------+----------+-----------+-----------+
| UserName | LastName | FirstName | IsDeleted |
+----------+----------+-----------+-----------+
|        1 | name-1   | NULL      |         0 |
|        1 | name-1   | NULL      |         0 |
+----------+----------+-----------+-----------+
2 rows in set (0.00 sec)

mysql> Truncate Persons;
Query OK, 0 rows affected (0.08 sec)
```

## 3. Unique Constraint

Fail to insert 2 same entries.

```shell
mysql> ALTER TABLE Persons ADD CONSTRAINT UserName_udx UNIQUE (UserName);
Query OK, 0 rows affected (0.20 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> INSERT INTO Persons(UserName, LastName, IsDeleted) VALUES('001', 'name-1', 0),('001', 'name-1', 0);
ERROR 1062 (23000): Duplicate entry '1' for key 'persons.UserName_udx'

mysql> Truncate Persons;
Query OK, 0 rows affected (0.11 sec)

mysql> ALTER TABLE Persons DROP CONSTRAINT UserName_udx;
Query OK, 0 rows affected (0.17 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

## 4. Conditional Unique Constraint

Successfully insert 2 same inactive entries.

```shell
mysql> ALTER TABLE Persons ADD CONSTRAINT UserName_udx UNIQUE (UserName, (CASE WHEN IsDeleted IN(0) THEN IsDeleted ELSE NULL END));
Query OK, 0 rows affected (0.18 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> INSERT INTO Persons(UserName, LastName, IsDeleted) VALUES('001', 'name-1', 1),('001', 'name-1', 1);
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM Persons;
+----------+----------+-----------+-----------+
| UserName | LastName | FirstName | IsDeleted |
+----------+----------+-----------+-----------+
|        1 | name-1   | NULL      |         1 |
|        1 | name-1   | NULL      |         1 |
+----------+----------+-----------+-----------+
2 rows in set (0.00 sec)
```

Only one active entry can be inserted.

```shell
mysql> INSERT INTO Persons(UserName, LastName, IsDeleted) VALUES('001', 'name-1', 0);
Query OK, 1 row affected (0.03 sec)

mysql> INSERT INTO Persons(UserName, LastName, IsDeleted) VALUES('001', 'name-1', 0);
ERROR 1062 (23000): Duplicate entry '1-0' for key 'persons.UserName_udx'
mysql> SELECT * FROM Persons;
+----------+----------+-----------+-----------+
| UserName | LastName | FirstName | IsDeleted |
+----------+----------+-----------+-----------+
|        1 | name-1   | NULL      |         1 |
|        1 | name-1   | NULL      |         1 |
|        1 | name-1   | NULL      |         0 |
+----------+----------+-----------+-----------+
3 rows in set (0.00 sec)
```

## 5. Clean Up

```shell
mysql> DROP TABLE Persons;
Query OK, 0 rows affected (0.07 sec)

mysql> DROP DATABASE test;
Query OK, 0 rows affected (0.04 sec)
```
