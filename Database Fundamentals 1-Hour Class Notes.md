# Database Fundamentals: 1-Hour Class Notes

Oct 12, 2026 · @El Rando

## Overview and timeline

This session covers what a database is, how relational databases are designed and queried, and when to use something else. By the end, students can:

- Explain what a DBMS does and why files and spreadsheets fall short
- Read and design simple tables with primary and foreign keys
- Write basic SQL: create, insert, select, update, delete, join, group
- Normalize a table to third normal form (3NF)
- Explain ACID transactions, indexes, and when NoSQL fits

| Minutes | Topic | Activity |
| --- | --- | --- |
| 0-10 | What is a database? | Lecture, quick poll |
| 10-20 | The relational model | Draw tables from a scenario |
| 20-35 | SQL essentials | Live queries |
| 35-45 | Normalization | Fix a messy table |
| 45-52 | Transactions and indexes | Bank transfer example |
| 52-58 | NoSQL and choosing a database | Discussion |
| 58-60 | Wrap-up | Quiz questions |

## Part 1 (0-10 min): What is a database?

A database is an organized collection of data stored electronically so it can be searched, updated, and shared. A database management system (DBMS) is the software that sits between users or apps and the stored data. Examples: MySQL, PostgreSQL, SQLite, SQL Server, Oracle, MongoDB.

**Why not just use files or spreadsheets?**

- Redundancy: the same data copied in many places
- Inconsistency: copies drift apart after edits
- Concurrency: two people editing at once overwrite each other
- Weak security: all-or-nothing access
- Slow search: no efficient way to query millions of rows

**What a DBMS provides**

- Storage and fast retrieval
- Integrity rules (no duplicate IDs, no orphan records)
- Concurrent access without corruption
- Security and permissions
- Backup and recovery

**Main types of databases**

| Type | Stores data as | Examples | Best for |
| --- | --- | --- | --- |
| Relational (SQL) | Tables with rows and columns | PostgreSQL, MySQL | Structured data, strong consistency |
| Document | JSON-like documents | MongoDB | Flexible, changing structure |
| Key-value | Key to value pairs | Redis | Caching, sessions |
| Wide-column | Column families | Cassandra | Huge write volumes |
| Graph | Nodes and edges | Neo4j | Relationships, recommendations |

## Part 2 (10-20 min): The relational model

The relational model stores data in tables (relations) and links them using keys.

| Term | Also called | Meaning |
| --- | --- | --- |
| Table | Relation | A set of rows about one kind of thing |
| Row | Tuple, record | One instance, e.g. one student |
| Column | Attribute, field | One property, e.g. email |
| Domain | Data type | Allowed values for a column |

**Keys**

| Key | Purpose |
| --- | --- |
| Primary key (PK) | Uniquely identifies each row; never null |
| Foreign key (FK) | Column that references another table's primary key |
| Candidate key | Any column set that could serve as the primary key |
| Composite key | A key made of two or more columns |
| Unique constraint | Prevents duplicate values in a column |

**Relationships**

- One-to-one (1:1): one person has one passport
- One-to-many (1:N): one department has many students
- Many-to-many (M:N): students take many courses and courses have many students. This needs a junction table.

**Running example used in this class**

students

| student\_id (PK) | name | dept\_id (FK) |
| --- | --- | --- |
| 1 | Amina | 10 |
| 2 | Brian | 20 |
| 3 | Chloe | 10 |

enrollments (junction table)

| student\_id (FK) | course\_id (FK) | grade |
| --- | --- | --- |
| 1 | 101 | A |
| 1 | 102 | B |
| 2 | 101 | C |

The enrollments table has a composite primary key: (student\_id, course\_id).

## Part 3 (20-35 min): SQL essentials

SQL (Structured Query Language) is the standard language for relational databases. Its commands fall into groups:

| Group | Full name | Commands |
| --- | --- | --- |
| DDL | Data Definition | CREATE, ALTER, DROP |
| DML | Data Manipulation | INSERT, UPDATE, DELETE |
| DQL | Data Query | SELECT |
| DCL | Data Control | GRANT, REVOKE |
| TCL | Transaction Control | COMMIT, ROLLBACK |

**Create a table**

```sql
CREATE TABLE students (
  student_id INT PRIMARY KEY,
  name       VARCHAR(50) NOT NULL,
  dept_id    INT,
  FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

**Insert, update, delete**

```sql
INSERT INTO students (student_id, name, dept_id) VALUES (4, 'Dina', 20);
UPDATE students SET dept_id = 10 WHERE student_id = 4;
DELETE FROM students WHERE student_id = 4;
```

Always include WHERE with UPDATE and DELETE, or every row is changed.

**Select and filter**

```sql
SELECT name FROM students WHERE dept_id = 10 ORDER BY name;
SELECT * FROM students WHERE name LIKE 'A%';
```

**Join tables**

```sql
SELECT s.name, e.course_id, e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id;
```

| Join | Returns |
| --- | --- |
| INNER JOIN | Only rows that match in both tables |
| LEFT JOIN | All left rows, plus matches from the right (NULL if none) |
| RIGHT JOIN | All right rows, plus matches from the left |
| FULL JOIN | All rows from both tables |

**Group and aggregate**

```sql
SELECT dept_id, COUNT(*) AS total
FROM students
GROUP BY dept_id
HAVING COUNT(*) > 1;
```

Aggregate functions: COUNT, SUM, AVG, MIN, MAX. Use WHERE to filter rows before grouping and HAVING to filter groups after.

**Order the query is executed:** FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY.

## Part 4 (35-45 min): Normalization

Normalization organizes tables to remove redundancy and prevent anomalies. The three anomalies:

- Insert anomaly: cannot add a course until a student enrolls
- Update anomaly: changing a teacher's name means editing many rows
- Delete anomaly: deleting the last student of a course erases the course

**Starting table (unnormalized)**

| student\_id | student\_name | courses | teacher |
| --- | --- | --- | --- |
| 1 | Amina | Math, Physics | Dr. Eto, Dr. Fon |
| 2 | Brian | Math | Dr. Eto |

**1NF: one value per cell, no repeating groups**

| student\_id | student\_name | course | teacher |
| --- | --- | --- | --- |
| 1 | Amina | Math | Dr. Eto |
| 1 | Amina | Physics | Dr. Fon |
| 2 | Brian | Math | Dr. Eto |

Key: (student\_id, course).

**2NF: 1NF, and no partial dependency.** Every non-key column must depend on the whole key. student\_name depends only on student\_id, so split it out.

- students(student\_id, student\_name)
- enrollments(student\_id, course, teacher)

**3NF: 2NF, and no transitive dependency.** Non-key columns must not depend on other non-key columns. teacher depends on course, so split it out.

- students(student\_id, student\_name)
- courses(course, teacher)
- enrollments(student\_id, course)

**Rule of thumb:** every non-key column depends on the key, the whole key, and nothing but the key.

Denormalizing (adding some redundancy back) is sometimes done on purpose to speed up read-heavy reports.

## Part 5 (45-52 min): Transactions and indexes

A transaction is a group of operations that must succeed or fail together. Example: transferring 100 from account A to account B.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;   -- or ROLLBACK; if anything failed
```

**ACID properties**

| Property | Meaning | In the example |
| --- | --- | --- |
| Atomicity | All steps happen or none do | Never debit A without crediting B |
| Consistency | Data stays valid under the rules | Total money is unchanged |
| Isolation | Concurrent transactions do not interfere | Another transfer sees consistent balances |
| Durability | Committed data survives a crash | The transfer persists after power loss |

**Indexes**

An index is a lookup structure (usually a B-tree) that lets the database find rows without scanning the whole table, like a book's index.

```sql
CREATE INDEX idx_students_name ON students(name);
```

- Pro: much faster searches, joins, and sorting on indexed columns
- Con: uses extra storage and slows INSERT, UPDATE, and DELETE
- Index columns you filter or join on often. Primary keys are indexed automatically.

## Part 6 (52-58 min): NoSQL and choosing a database

NoSQL databases trade some relational features for flexible structure and easy scaling across many servers. Many follow BASE (basically available, soft state, eventually consistent) instead of strict ACID, though some now support transactions too.

|  | Relational (SQL) | NoSQL |
| --- | --- | --- |
| Structure | Fixed schema, defined up front | Flexible or schema-less |
| Query language | SQL | Varies by database |
| Relationships | Joins | Embedded data or application code |
| Scaling | Mostly vertical, some horizontal | Designed for horizontal scaling |
| Consistency | Strong (ACID) | Often eventual |
| Typical use | Banking, inventory, school records | Social feeds, caching, logs, catalogs |

**How to choose**

- Data is structured and correctness matters most: relational
- Data shape varies or changes often: document
- Need very fast simple lookups or caching: key-value
- Relationships are the main question (friends of friends): graph
- When unsure, start with PostgreSQL or MySQL. It handles most needs.

## Wrap-up (58-60 min): Review and practice

**Key takeaways**

- A DBMS manages data safely for many users at once
- Relational databases use tables linked by primary and foreign keys
- SQL: SELECT to read, INSERT, UPDATE, DELETE to change, JOIN to combine
- Normalize to 3NF to avoid redundancy and anomalies
- ACID keeps transactions reliable; indexes make reads fast
- Pick the database type that matches the data and the workload

**Quick quiz**

1. What is the difference between a primary key and a foreign key?
2. Which join keeps all rows from the left table even with no match?
3. What does the A in ACID guarantee?
4. Why is student\_name in an enrollments table a 2NF problem?
5. Name one downside of adding an index.

**Answers:** (1) a primary key identifies a row in its own table; a foreign key points to another table's primary key. (2) LEFT JOIN. (3) Atomicity: all steps happen or none do. (4) It depends only on student\_id, part of the key. (5) Slower writes and extra storage.

**Practice exercises**

- [ ] Design tables for a library (books, members, loans) and mark all keys
- [ ] Write a query listing each student's name with their courses
- [ ] Count students per department and show only departments with 2 or more
- [ ] Normalize an orders table that stores customer name, address, and product list in one row
- [ ] Try the queries free at an online SQL sandbox or with SQLite
