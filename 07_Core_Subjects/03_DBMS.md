# DBMS + SQL — Complete Interview Guide

> Goal: after reading this one file you can clear a **core DBMS + SQL round** at Amazon, Google, Flipkart, Zomato/Eternal and similar companies.
> All SQL is **PostgreSQL**. ⭐ = very commonly asked. **Project** lines link a concept to your real work (LocalArena, TTL Chat) — use them as stories.

How to use this file:
1. Read sections 1–16 once, slowly. Run the SQL in `psql` if you can (sample DB is in Section 1).
2. Write the 25 practice queries (Section 7) **by hand** on paper or a whiteboard.
3. Day before the interview: Section 17 (Top 40 questions) + Section 18 (last-minute revision).

Every topic follows the same pattern: **Simple explanation → Key points → Example → Code → Interview tip / trap**.

---

## Table of Contents

1. [Sample Database (used everywhere)](#1-sample-database-used-everywhere)
2. [DBMS Basics](#2-dbms-basics)
3. [ER Model](#3-er-model)
4. [Relational Model, Keys and Integrity Constraints](#4-relational-model-keys-and-integrity-constraints)
5. [Relational Algebra](#5-relational-algebra)
6. [SQL in Depth](#6-sql-in-depth)
7. [SQL Practice: 25 Most-Asked Interview Queries](#7-sql-practice-25-most-asked-interview-queries)
8. [Functional Dependencies and Normalization](#8-functional-dependencies-and-normalization)
9. [Transactions](#9-transactions)
10. [Concurrency Control](#10-concurrency-control)
11. [Recovery](#11-recovery)
12. [Storage and Indexing](#12-storage-and-indexing)
13. [Query Processing and Optimization](#13-query-processing-and-optimization)
14. [Scaling Databases](#14-scaling-databases)
15. [NoSQL and Redis](#15-nosql-and-redis)
16. [ORMs and the N+1 Problem](#16-orms-and-the-n1-problem)
17. [Most Asked DBMS Interview Questions (Top 40)](#17-most-asked-dbms-interview-questions-top-40)
18. [Last-minute Revision](#18-last-minute-revision)

---

## 1. Sample Database (used everywhere)

**Simple explanation:** One small company that has employees **and** sells products online. Every SQL example in this file uses these tables, so you can run them and check the outputs.

```
departments 1───* employees *───1 employees (manager, self-reference)
customers   1───* orders 1───* order_items *───1 products
accounts (bank transfer demo) · logs, contacts (practice-query tables)
```

```sql
-- ============ CLEAN START ============
DROP TABLE IF EXISTS order_items, orders, products, customers,
                     employees, departments, accounts, logs, contacts CASCADE;

-- ============ HR PART ============
CREATE TABLE departments (
    dept_id   INT PRIMARY KEY,
    name      VARCHAR(50) NOT NULL UNIQUE,
    location  VARCHAR(50)
);

CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    name       VARCHAR(50)  NOT NULL,
    email      VARCHAR(100) UNIQUE,
    dept_id    INT REFERENCES departments(dept_id) ON DELETE SET NULL,
    manager_id INT REFERENCES employees(emp_id),          -- self-reference
    salary     NUMERIC(10,2) NOT NULL CHECK (salary > 0),
    hire_date  DATE NOT NULL DEFAULT CURRENT_DATE
);

-- ============ E-COMMERCE PART ============
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name        VARCHAR(50) NOT NULL,
    city        VARCHAR(50),
    email       VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name       VARCHAR(50) NOT NULL,
    category   VARCHAR(30) NOT NULL,
    price      NUMERIC(10,2) NOT NULL CHECK (price >= 0),
    stock      INT NOT NULL CHECK (stock >= 0),
    version    INT NOT NULL DEFAULT 1                      -- for optimistic locking
);

CREATE TABLE orders (
    order_id     INT PRIMARY KEY,
    customer_id  INT NOT NULL REFERENCES customers(customer_id),
    order_date   DATE NOT NULL,
    status       VARCHAR(20) NOT NULL
                 CHECK (status IN ('PLACED','SHIPPED','DELIVERED','CANCELLED')),
    total_amount NUMERIC(12,2) NOT NULL
);

CREATE TABLE order_items (
    order_id   INT REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT REFERENCES products(product_id),
    quantity   INT NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL,
    PRIMARY KEY (order_id, product_id)                     -- composite key
);

-- ============ DEMO / PRACTICE TABLES ============
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    owner      VARCHAR(50) NOT NULL,
    balance    NUMERIC(12,2) NOT NULL CHECK (balance >= 0)
);

CREATE TABLE logs     (id INT PRIMARY KEY, num INT);        -- "consecutive numbers"
CREATE TABLE contacts (id INT PRIMARY KEY, email VARCHAR(100)); -- "duplicates"

-- ============ DATA ============
INSERT INTO departments VALUES
 (1,'Engineering','Bengaluru'), (2,'Sales','Mumbai'),
 (3,'HR','Delhi'),              (4,'Marketing','Pune');   -- Marketing has nobody

INSERT INTO employees VALUES
 (1,'Asha',  'asha@co.in',  1, NULL, 150000,'2019-01-10'),
 (2,'Ravi',  'ravi@co.in',  1, 1,    120000,'2020-03-15'),
 (3,'Neha',  'neha@co.in',  1, 1,    160000,'2021-07-01'),
 (4,'Karan', 'karan@co.in', 2, NULL,  90000,'2018-11-20'),
 (5,'Priya', 'priya@co.in', 2, 4,     95000,'2022-02-01'),
 (6,'Vikram','vikram@co.in',2, 4,     95000,'2023-05-10'),
 (7,'Meera', 'meera@co.in', 3, 1,     60000,'2023-08-01'),
 (8,'Arjun', 'arjun@co.in', NULL, 1,  50000,'2024-01-15'); -- no department

INSERT INTO customers VALUES
 (1,'Rohan','Delhi','rohan@mail.com'),   (2,'Sara','Mumbai','sara@mail.com'),
 (3,'Imran','Bengaluru','imran@mail.com'),(4,'Divya','Pune','divya@mail.com'); -- Divya never orders

INSERT INTO products VALUES
 (1,'Laptop','Electronics',60000,10,1), (2,'Phone','Electronics',20000,25,1),
 (3,'Headphones','Electronics',2000,100,1), (4,'Book','Books',500,200,1),
 (5,'Pen','Stationery',20,1000,1);                      -- Pen never ordered

INSERT INTO orders VALUES
 (101,1,'2026-01-05','DELIVERED',62000),
 (102,2,'2026-01-20','DELIVERED',20000),
 (103,1,'2026-02-10','SHIPPED',   1000),
 (104,3,'2026-02-11','CANCELLED',20000),
 (105,2,'2026-03-01','PLACED',    2500),
 (106,3,'2026-03-15','DELIVERED',64000);

INSERT INTO order_items VALUES
 (101,1,1,60000),(101,3,1,2000),
 (102,2,1,20000),
 (103,4,2,500),
 (104,2,1,20000),
 (105,3,1,2000),(105,4,1,500),
 (106,1,1,60000),(106,3,2,2000);

INSERT INTO accounts VALUES (1,'Rohan',5000),(2,'Sara',3000);

INSERT INTO logs VALUES (1,1),(2,1),(3,1),(4,2),(5,1),(6,2),(7,2);
INSERT INTO contacts VALUES
 (1,'a@x.com'),(2,'b@x.com'),(3,'a@x.com'),(4,'c@x.com'),(5,'b@x.com'),(6,'a@x.com');
```

Quick picture of `employees` (you will need this for outputs):

| emp_id | name | dept | manager | salary |
|---|---|---|---|---|
| 1 | Asha | Engineering | — | 150000 |
| 2 | Ravi | Engineering | Asha | 120000 |
| 3 | Neha | Engineering | Asha | 160000 |
| 4 | Karan | Sales | — | 90000 |
| 5 | Priya | Sales | Karan | 95000 |
| 6 | Vikram | Sales | Karan | 95000 |
| 7 | Meera | HR | Asha | 60000 |
| 8 | Arjun | (NULL) | Asha | 50000 |

---

## 2. DBMS Basics

### 2.1 What is a DBMS? DBMS vs File System

**Simple explanation:** A **database** is an organised collection of data. A **DBMS** is the software that stores it, lets many users read/write it safely, and protects it (PostgreSQL, MySQL, Oracle, MongoDB). Storing data in plain files (CSV, text) is like keeping accounts in loose notebooks; a DBMS is like a bank with rules, a vault and a ledger.

**Key points — problems with file systems that a DBMS solves:**

| Problem with files | How DBMS fixes it |
|---|---|
| Data redundancy and inconsistency (same address in 3 files, all different) | Normalised tables, one source of truth |
| Hard to query (write a new C++ program for each question) | SQL — declarative queries |
| No integrity rules (age = -5 is accepted) | Constraints: NOT NULL, CHECK, FK |
| No atomicity (crash in middle of transfer = money lost) | Transactions (ACID) + logs |
| Concurrent access breaks data (two writers overwrite) | Locking / MVCC |
| No security per user | GRANT/REVOKE, roles |
| No crash recovery / backup | WAL, checkpoints, replication |

**RDBMS** = DBMS based on the relational model (tables + keys + SQL). PostgreSQL, MySQL, Oracle, SQL Server.

**Interview tip:** When asked "why not just use files?", list 4–5 rows of this table: redundancy, integrity, atomicity, concurrency, security, recovery.

### 2.2 Three-Schema Architecture (ANSI/SPARC)

**Simple explanation:** The DB is described at 3 levels, like a building: what each visitor sees (view), the floor plan (logical), and the bricks and pipes (physical).

```
   View 1     View 2     View 3        <- EXTERNAL level (what each user/app sees)
       \        |        /
        CONCEPTUAL / LOGICAL level     <- all tables, columns, keys, constraints
                 |
          INTERNAL / PHYSICAL level    <- files, pages, indexes, B+ trees on disk
```

| Level | What it describes | Example |
|---|---|---|
| External (view) | Part of DB a user sees | HR view hides `salary` from interns |
| Conceptual (logical) | Whole DB structure | `employees(emp_id, name, dept_id, salary)` |
| Internal (physical) | How data is stored | heap file, 8 KB pages, B+ tree index on `emp_id` |

### 2.3 Data Independence

**Simple explanation:** You can change one level without breaking the level above it.

- **Physical data independence:** change storage (add an index, move to SSD, change file layout) **without** changing the logical schema or queries. Easy to achieve.
- **Logical data independence:** change the logical schema (add a column, split a table) **without** changing apps/views. Harder to achieve, because apps depend on table structure. Views help.

**Interview trap:** "Which is harder?" → **Logical** data independence.

### 2.4 Data Models

**Simple explanation:** A data model is the "shape" in which you describe data.

| Model | Idea | Example |
|---|---|---|
| Hierarchical | Tree, one parent per child | IBM IMS, XML, file system folders |
| Network | Graph, many parents allowed | CODASYL (old) |
| Relational | Tables (relations) with keys | PostgreSQL, MySQL |
| ER (Entity-Relationship) | Design-level diagram of entities and relationships | used before creating tables |
| Object / Object-relational | Objects, inheritance, custom types | PostgreSQL supports some (arrays, JSONB, custom types) |
| Document / Key-value / Column / Graph (NoSQL) | Flexible schemas | MongoDB, Redis, Cassandra, Neo4j |

### 2.5 Other basic terms

- **Schema** = structure (design, changes rarely). **Instance** = data at a moment (changes all the time).
- **Metadata / data dictionary / catalog** = data about data (PostgreSQL: `information_schema`, `pg_catalog`).
- **DBA** = person who manages the DB (users, backups, tuning).
- **Database languages:** DDL (define), DML (manipulate), DCL (control access), TCL (transactions).
- **DBMS architecture tiers:** 1-tier (DB on same machine), 2-tier (client → DB), 3-tier (client → app server → DB). Web apps are 3-tier.

---

## 3. ER Model

### 3.1 Entities and Attributes

**Simple explanation:** An ER diagram is the "drawing" you make before creating tables. **Entity** = a real-world thing (Student, Order). **Attribute** = a property (name, price). **Relationship** = how entities connect (Customer *places* Order).

**Key points:**
- **Entity** (one row, e.g. "Rohan") vs **Entity set** (all customers — becomes a table).
- **Strong entity:** has its own key (Customer has customer_id).
- **Weak entity:** cannot be identified by its own attributes alone; depends on an **owner** (identifying) entity. Example: `Dependent(name)` of an Employee; `OrderItem(line_no)` of an Order. It has a **partial key / discriminator** (dashed underline). Shown with double rectangle; its identifying relationship with double diamond. Always **total participation**.

**Attribute types:**

| Type | Meaning | Example | ER symbol |
|---|---|---|---|
| Simple (atomic) | Cannot be divided | `age` | ellipse |
| Composite | Made of parts | `name` = first + last; `address` = street, city, pin | ellipse with child ellipses |
| Single-valued | One value | `dob` | ellipse |
| Multi-valued | Many values | `phone_numbers` | double ellipse |
| Derived | Computed from others | `age` from `dob` | dashed ellipse |
| Key attribute | Uniquely identifies | `roll_no` | underlined |
| Stored | Actually stored | `dob` | ellipse |
| Complex | Composite + multi-valued | multiple addresses each with parts | — |

### 3.2 Relationships, Degree, Cardinality, Participation

- **Degree:** number of entity sets in a relationship. Unary (Employee *manages* Employee), Binary (Customer–Order), Ternary (Doctor–Patient–Medicine *prescribes*).
- **Cardinality (mapping constraint):**

| Cardinality | Example |
|---|---|
| 1 : 1 | Person — Passport |
| 1 : N | Department — Employees; Customer — Orders |
| M : N | Student — Course; Order — Product |

- **Participation:**
  - **Total** (double line): every entity must participate. Every Order *must* have a Customer.
  - **Partial** (single line): some may not. A Customer may have zero orders (Divya).
- Relationships can have their own attributes: `Enrolls(Student, Course)` has `grade`; `Contains(Order, Product)` has `quantity`.

### 3.3 Extended ER (EER) features

- **Specialization** (top-down): Employee → Engineer, Manager.
- **Generalization** (bottom-up): Car, Bike → Vehicle.
- **Inheritance:** sub-entity gets attributes of super-entity.
- **Aggregation:** treat a relationship as an entity to relate it further (e.g., `Works_On(Employee, Project)` is *monitored by* Manager).

### 3.4 ER → Tables (conversion rules) with example

**Example ER (our shop):**

```
[Customer]──1────places────N──[Order]──M────contains(quantity)────N──[Product]
 customer_id (PK)              order_id (PK)                         product_id (PK)
 name                          order_date                            name, price
 phones {multi-valued}         ║ total participation (every order has a customer)
 address(street, city)
```

**Rules:**

| ER element | Becomes |
|---|---|
| Strong entity | Table; key attribute → PK |
| Composite attribute | Store only the leaf parts: `street`, `city` (no `address` column) |
| Multi-valued attribute | **Separate table**: `customer_phones(customer_id FK, phone, PK(customer_id, phone))` |
| Derived attribute | Usually not stored (compute `age` from `dob`) |
| Weak entity | Table with PK = owner's PK + partial key; FK to owner with `ON DELETE CASCADE` |
| 1:1 relationship | FK on either side (prefer the side with total participation), add UNIQUE |
| 1:N relationship | FK on the **N side** (`orders.customer_id`) — no extra table |
| M:N relationship | **New junction table** with both FKs; PK = (FK1, FK2); relationship attributes go here (`order_items.quantity`) |
| Unary (self) 1:N | FK to same table (`employees.manager_id`) |
| Ternary | Table with 3 FKs |
| Specialization | Either one table per subclass (with FK to parent), one big table with a `type` column, or only subclass tables |

**Result for the example:** `customers`, `customer_phones`, `orders(customer_id FK NOT NULL — total participation)`, `products`, `order_items(order_id, product_id, quantity)`. Apart from `customer_phones` and the address parts, this is our sample schema.

**Interview tip:** M:N always needs a third table. 1:N never does. Total participation on the N side → FK is `NOT NULL`.

---

## 4. Relational Model, Keys and Integrity Constraints

### 4.1 Relational model terms

**Simple explanation:** Data lives in **tables**. Formal names:

| Formal term | Everyday term |
|---|---|
| Relation | Table |
| Tuple | Row / record |
| Attribute | Column / field |
| Domain | Allowed values of a column (e.g., INT > 0) |
| Degree | Number of columns |
| Cardinality | Number of rows |
| Relation schema | `employees(emp_id, name, dept_id, ...)` |
| Relation instance | Rows at this moment |

**Properties of a relation:** each cell atomic; no two identical rows (in theory — SQL tables can have duplicates without a key); order of rows and columns does not matter; column names unique.

**NULL** = unknown / not applicable. `NULL = NULL` is **not true** (it's UNKNOWN). Use `IS NULL`. SQL uses **3-valued logic**: TRUE, FALSE, UNKNOWN.

### 4.2 Keys ⭐

**Simple explanation:** A key is a set of columns that lets you find exactly one row — like a roll number in a class.

Use this table: `student(roll_no, email, aadhaar, name, phone, dept)`.

| Key | Definition | Example |
|---|---|---|
| **Super key** | Any set of columns that uniquely identifies a row (may have extra columns) | `{roll_no}`, `{roll_no, name}`, `{email, dept}` |
| **Candidate key** | **Minimal** super key (remove any column → no longer unique) | `{roll_no}`, `{email}`, `{aadhaar}` |
| **Primary key** | The candidate key you choose. Unique + NOT NULL. One per table | `roll_no` |
| **Alternate key** | Candidate keys not chosen as primary | `email`, `aadhaar` |
| **Composite key** | Key with 2+ columns | `order_items(order_id, product_id)` |
| **Foreign key** | Column that refers to a PK/unique key of another (or same) table | `orders.customer_id → customers.customer_id` |
| **Unique key** | UNIQUE constraint; allows NULLs (Postgres allows multiple NULLs) | `employees.email` |
| **Surrogate key** | Artificial key with no business meaning | `id BIGINT GENERATED ALWAYS AS IDENTITY`, UUID |
| **Natural key** | Real-world meaning | PAN number, email |
| **Prime attribute** | Attribute that is part of *some* candidate key | `roll_no`, `email` |

Relationship: every candidate key is a super key; PK is one candidate key. Super ⊇ Candidate ⊇ Primary.

> **Project:** "My idempotency table in LocalArena uses a **composite unique key `(player_id, idempotency_key)`**. The same key from a different player is fine; the same player retrying the same request is rejected by the DB itself."

**Interview trap:** "Can a PK be NULL?" No. "Can a unique key have NULL?" Yes (in Postgres, multiple NULLs; use `UNIQUE NULLS NOT DISTINCT` in PG 15+ to allow only one). "Can a FK be NULL?" Yes (Arjun has no dept).

### 4.3 Integrity Constraints

**Simple explanation:** Rules the DB enforces so bad data can never get in.

| Constraint | Rule | SQL |
|---|---|---|
| Domain | Value must be of right type/range | `salary NUMERIC CHECK (salary > 0)` |
| Entity integrity | PK cannot be NULL | `PRIMARY KEY` |
| Referential integrity | FK must match an existing PK or be NULL | `REFERENCES customers(customer_id)` |
| Key constraint | No duplicate keys | `UNIQUE`, `PRIMARY KEY` |
| NOT NULL | Value required | `name VARCHAR NOT NULL` |
| CHECK | Custom condition | `CHECK (status IN (...))` |
| DEFAULT | Value if none given | `DEFAULT CURRENT_DATE` |

**Referential actions** (what happens to children when parent is deleted/updated):

| Action | Meaning |
|---|---|
| `NO ACTION` / `RESTRICT` (default) | Block the delete if children exist |
| `CASCADE` | Delete/update children too (`order_items` when order deleted) |
| `SET NULL` | Set FK to NULL (`employees.dept_id` when dept deleted) |
| `SET DEFAULT` | Set FK to default |

```sql
-- Fails: customer 99 does not exist (referential integrity)
INSERT INTO orders VALUES (200, 99, '2026-04-01', 'PLACED', 100);
-- ERROR: insert or update on table "orders" violates foreign key constraint

-- Fails: CHECK
UPDATE products SET stock = -1 WHERE product_id = 1;
-- ERROR: new row for relation "products" violates check constraint
```

**Interview tip:** Put rules in the DB, not only in app code. Two app servers racing can both pass an app check; only a DB constraint is race-proof. That's why 1.45M replayed events gave **0 duplicate rewards** in LocalArena — deterministic IDs + dedupe + **unique constraints**.

---

## 5. Relational Algebra

**Simple explanation:** Relational algebra is the "math" behind SQL. Each operator takes table(s) and returns a table. The query optimizer converts SQL into a tree of these operators.

| Operator | Symbol | Meaning | SQL equivalent |
|---|---|---|---|
| Select | σ (sigma) | Filter **rows** | `WHERE` |
| Project | π (pi) | Choose **columns** (removes duplicates in theory) | `SELECT DISTINCT col` |
| Rename | ρ (rho) | Rename table/column | `AS` |
| Union | ∪ | Rows in A or B (same columns) | `UNION` |
| Set difference | − | Rows in A not in B | `EXCEPT` |
| Intersection | ∩ | Rows in both | `INTERSECT` |
| Cartesian product | × | Every row of A with every row of B | `CROSS JOIN` |
| Join | ⋈ | × followed by σ on a condition | `JOIN ... ON` |
| Natural join | ⋈ | Join on all same-named columns | `NATURAL JOIN` |
| Division | ÷ | "for all" queries | `NOT EXISTS ... NOT EXISTS` |

**Examples:**

```
σ salary > 100000 (employees)
  → SELECT * FROM employees WHERE salary > 100000;

π name, salary (σ dept_id = 1 (employees))
  → SELECT DISTINCT name, salary FROM employees WHERE dept_id = 1;

employees ⋈ employees.dept_id = departments.dept_id departments
  → SELECT * FROM employees e JOIN departments d ON e.dept_id = d.dept_id;

π customer_id (customers) − π customer_id (orders)      -- customers with no order
  → SELECT customer_id FROM customers EXCEPT SELECT customer_id FROM orders;   -- {4}
```

Union / difference / intersection need **union-compatible** relations: same number of columns and compatible types.

**Division (briefly):** "Find customers who bought **all** Electronics products." R(customer, product) ÷ S(product) = customers paired with every product in S. In SQL use **double NOT EXISTS** ("there is no electronics product that this customer has not bought"):

```sql
SELECT c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM products p
    WHERE p.category = 'Electronics'
      AND NOT EXISTS (
          SELECT 1 FROM orders o JOIN order_items oi ON oi.order_id = o.order_id
          WHERE o.customer_id = c.customer_id AND oi.product_id = p.product_id));
-- Electronics = Laptop, Phone, Headphones.
-- Rohan: Laptop, Headphones (no Phone) ✗ · Sara: Phone, Headphones ✗
-- Imran: Phone (order 104), Laptop, Headphones (order 106) ✓  → Imran
```

**Other operators to name:** Outer joins (⟕ left, ⟖ right, ⟗ full), aggregation (𝒢). **Relational calculus** (tuple/domain) is the declarative "what" version; algebra is the procedural "how" version. SQL is based on both.

**Interview trap:** σ works on rows, π on columns. People mix them up — "**S**elect = **S**ift rows, **P**roject = **P**ick columns".

---

## 6. SQL in Depth

> Some demo statements below change data (UPDATE/DELETE/CALL). Outputs elsewhere assume the **original** data — re-run the Section 1 script to reset.

### 6.1 SQL command categories ⭐

**Simple explanation:** SQL commands come in 4 families (some books add DQL = SELECT).

| Family | Purpose | Commands |
|---|---|---|
| **DDL** (Data Definition) | Define structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`, `COMMENT` |
| **DML** (Data Manipulation) | Change/read data | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` |
| **DCL** (Data Control) | Permissions | `GRANT`, `REVOKE` |
| **TCL** (Transaction Control) | Transactions | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `SET TRANSACTION` |

```sql
-- DDL
CREATE TABLE reviews (
    review_id   BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  -- auto-increment (SERIAL is older style)
    product_id  INT NOT NULL REFERENCES products(product_id),
    rating      INT CHECK (rating BETWEEN 1 AND 5),
    body        TEXT,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
ALTER TABLE reviews ADD COLUMN customer_id INT REFERENCES customers(customer_id);
ALTER TABLE reviews ALTER COLUMN body SET NOT NULL;
ALTER TABLE reviews ADD CONSTRAINT uq_one_review UNIQUE (product_id, customer_id);
ALTER TABLE reviews DROP COLUMN body;

-- DML
INSERT INTO reviews (product_id, customer_id, rating) VALUES (1, 1, 5) RETURNING review_id;
UPDATE products SET price = price * 0.9 WHERE category = 'Books';
DELETE FROM reviews WHERE rating < 2;

-- DCL
CREATE ROLE analyst LOGIN PASSWORD '...';
GRANT SELECT ON orders, customers TO analyst;
REVOKE SELECT ON customers FROM analyst;

-- TCL: see Section 9
```

**DELETE vs TRUNCATE vs DROP** ⭐

| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | Selected rows (`WHERE`) | All rows | Whole table (data + structure) |
| Speed | Slow (row by row, logs each row) | Fast (frees pages) | Fast |
| Triggers fire? | Yes (row triggers) | No row triggers | No |
| Rollback? | Yes | **Yes in PostgreSQL** (inside a transaction); not in MySQL/Oracle | Yes in PostgreSQL inside a txn |
| Identity reset | No | With `RESTART IDENTITY` | Gone |

**Trap:** "TRUNCATE can't be rolled back" is true in MySQL/Oracle, **false in PostgreSQL** (DDL is transactional in Postgres). Say this — it shows depth.

### 6.2 SELECT — order of execution ⭐

**Simple explanation:** You *write* `SELECT` first, but the DB *runs* it in a different order.

```
Written:   SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
Executed:  FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT (window fns here) → DISTINCT → ORDER BY → LIMIT/OFFSET
```

That's why you **cannot use a SELECT alias in WHERE** (WHERE runs before SELECT), but you **can** in ORDER BY.

```sql
SELECT name, salary * 12 AS annual
FROM employees
WHERE salary * 12 > 1000000     -- must repeat expression, "annual" not visible yet
ORDER BY annual DESC;           -- alias OK here
```

Useful operators: `BETWEEN a AND b` (inclusive), `IN (...)`, `LIKE 'A%'` / `ILIKE` (case-insensitive, Postgres), `IS NULL`, `COALESCE(x, default)`, `NULLIF(a,b)`, `CASE WHEN ... THEN ... ELSE ... END`, `DISTINCT`, `DISTINCT ON (col)` (Postgres), `LIMIT n OFFSET m`.

### 6.3 Joins ⭐

**Simple explanation:** A join glues rows from two tables when a condition matches — like matching students to their class teacher by class name.

Small sample (from Section 1):

`employees` (subset of columns) and `departments`:

| emp | dept_id |   | dept_id | dept |
|---|---|---|---|---|
| Asha | 1 |  | 1 | Engineering |
| Ravi | 1 |  | 2 | Sales |
| Neha | 1 |  | 3 | HR |
| Karan | 2 |  | 4 | Marketing |
| Priya | 2 | | | |
| Vikram | 2 | | | |
| Meera | 3 | | | |
| Arjun | NULL | | | |

```sql
SELECT e.name, d.name AS dept
FROM employees e
<JOIN TYPE> departments d ON e.dept_id = d.dept_id;
```

| Join type | Rule | Output for our data |
|---|---|---|
| `INNER JOIN` | Only matching rows on both sides | 7 rows: Asha/Ravi/Neha–Engineering, Karan/Priya/Vikram–Sales, Meera–HR (**Arjun and Marketing missing**) |
| `LEFT JOIN` | All left rows + matches (else NULL) | 7 rows + **Arjun–NULL** = 8 |
| `RIGHT JOIN` | All right rows + matches | 7 rows + **NULL–Marketing** = 8 |
| `FULL OUTER JOIN` | All rows from both | 7 + Arjun–NULL + NULL–Marketing = 9 |
| `CROSS JOIN` | Every pair (no ON) | 8 × 4 = **32** rows |
| `SELF JOIN` | Table joined with itself | employee–manager pairs (below) |
| `NATURAL JOIN` | Auto-join on same-named columns | Dangerous: here both have `name` and `dept_id` → joins on both → wrong (0 rows) |

Venn-style memory:
```
INNER:  (A∩B)        LEFT:  (A)          RIGHT: (B)          FULL: (A∪B)
LEFT anti-join (A only):  LEFT JOIN ... WHERE B.key IS NULL
```

**Self join — employee with manager name:**
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```
| employee | manager |
|---|---|
| Asha | NULL |
| Ravi | Asha |
| Neha | Asha |
| Karan | NULL |
| Priya | Karan |
| Vikram | Karan |
| Meera | Asha |
| Arjun | Asha |

**Three-table join — what each customer bought:**
```sql
SELECT c.name, o.order_id, p.name AS product, oi.quantity
FROM customers c
JOIN orders o       ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id   = o.order_id
JOIN products p     ON p.product_id  = oi.product_id
ORDER BY o.order_id;
```

**Interview traps:**
1. **WHERE on the right table kills a LEFT JOIN.** `LEFT JOIN orders o ON ... WHERE o.status = 'DELIVERED'` removes the NULL rows → behaves like INNER. Put the filter in `ON` instead: `LEFT JOIN orders o ON o.customer_id = c.customer_id AND o.status = 'DELIVERED'`.
2. **Join on a non-unique column multiplies rows** (fan-out). Summing after a join can double-count.
3. **Semi-join** (`EXISTS`/`IN`) = "rows of A that have a match in B" without duplicates. **Anti-join** (`NOT EXISTS`) = no match.

### 6.4 Subqueries — non-correlated vs correlated ⭐

**Simple explanation:** A subquery is a query inside a query.
- **Non-correlated:** inner query runs **once**, independent of outer. Like computing the class average once, then comparing everyone.
- **Correlated:** inner query **refers to the outer row**, so logically runs **once per outer row**. Like computing *each student's own section average*.

```sql
-- Non-correlated: earn more than company average (inner runs once)
SELECT name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);       -- avg = 102500
-- Asha, Ravi, Neha

-- Correlated: earn more than THEIR department average
SELECT e.name, e.dept_id, e.salary
FROM employees e
WHERE e.salary > (SELECT AVG(salary) FROM employees x WHERE x.dept_id = e.dept_id);
-- Asha (150000 > 143333), Neha, Priya (95000 > 93333), Vikram

-- EXISTS (correlated, stops at first match)
SELECT c.name FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
-- Rohan, Sara, Imran
```

Where subqueries can appear: `WHERE` (scalar / IN / EXISTS), `FROM` (derived table — needs an alias), `SELECT` (scalar subquery, must return ≤1 row).

| | IN | EXISTS |
|---|---|---|
| Works by | Builds list of values | Checks if any row exists |
| Best when | Small inner result | Large inner table with index on join column |
| NULL trap | `NOT IN` with a NULL in list → **returns nothing** | `NOT EXISTS` is NULL-safe |

**Trap ⭐:** `WHERE dept_id NOT IN (SELECT dept_id FROM employees)` returns **0 rows** because Arjun's dept_id is NULL (`x <> NULL` is UNKNOWN). Always prefer `NOT EXISTS` for anti-joins. Modern optimizers often rewrite correlated subqueries into joins, so "correlated = always slow" is not fully true — but say "logically per row".

### 6.5 GROUP BY, HAVING, aggregates ⭐

**Simple explanation:** `GROUP BY` puts rows into buckets; aggregate functions (`COUNT, SUM, AVG, MIN, MAX`) give one value per bucket. `WHERE` filters rows **before** bucketing, `HAVING` filters buckets **after**.

```sql
SELECT d.name AS dept, COUNT(*) AS emp_count, AVG(e.salary)::INT AS avg_sal
FROM employees e
JOIN departments d ON d.dept_id = e.dept_id
WHERE e.hire_date >= '2019-01-01'        -- row filter (removes Karan, hired 2018)
GROUP BY d.name
HAVING COUNT(*) >= 2                     -- group filter
ORDER BY avg_sal DESC;
```
| dept | emp_count | avg_sal |
|---|---|---|
| Engineering | 3 | 143333 |
| Sales | 2 | 95000 |

**Key points:**
- Every non-aggregated column in SELECT must be in GROUP BY (Postgres allows skipping if you group by the PK).
- `COUNT(*)` counts rows; `COUNT(col)` skips NULLs; `COUNT(DISTINCT col)` counts unique non-null.
- Aggregates ignore NULLs (`AVG` of 10, NULL, 20 = 15).
- Postgres extras: `FILTER (WHERE ...)`, `STRING_AGG(name, ', ')`, `ARRAY_AGG`, `GROUPING SETS / ROLLUP / CUBE` for subtotals.

**Trap:** "WHERE vs HAVING?" — WHERE can't use aggregates (`WHERE COUNT(*) > 1` is an error). HAVING can.

### 6.6 Set operations

```sql
SELECT city FROM customers UNION     SELECT location FROM departments;  -- unique values
SELECT city FROM customers UNION ALL SELECT location FROM departments;  -- keeps duplicates, faster
SELECT city FROM customers INTERSECT SELECT location FROM departments;  -- Delhi, Mumbai, Bengaluru, Pune
SELECT city FROM customers EXCEPT    SELECT location FROM departments;  -- empty
```
**Tip:** Use `UNION ALL` unless you need de-duplication — `UNION` must sort/hash to remove duplicates.

### 6.7 Window functions ⭐⭐ (very common in interviews)

**Simple explanation:** An aggregate with `GROUP BY` **collapses** rows into one per group. A **window function** computes over a group of rows but **keeps every row** — like writing each student's rank next to their name without merging the class.

Syntax: `func() OVER (PARTITION BY ... ORDER BY ... [frame])`
- `PARTITION BY` = the groups (like GROUP BY but no collapse).
- `ORDER BY` = order inside each group.
- Frame = which rows count (`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`).

```sql
SELECT name, dept_id, salary,
       ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS row_num,
       RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk,
       DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS dense_rnk
FROM employees
WHERE dept_id IN (1, 2);
```
| name | dept_id | salary | row_num | rnk | dense_rnk |
|---|---|---|---|---|---|
| Neha | 1 | 160000 | 1 | 1 | 1 |
| Asha | 1 | 150000 | 2 | 2 | 2 |
| Ravi | 1 | 120000 | 3 | 3 | 3 |
| Priya | 2 | 95000 | 1 | 1 | 1 |
| Vikram | 2 | 95000 | 2 | 1 | 1 |
| Karan | 2 | 90000 | 3 | **3** | **2** |

| Function | Ties | Sequence for 95k, 95k, 90k |
|---|---|---|
| `ROW_NUMBER()` | Always unique (ties broken arbitrarily) | 1, 2, 3 |
| `RANK()` | Same rank, then **gap** | 1, 1, 3 |
| `DENSE_RANK()` | Same rank, **no gap** | 1, 1, 2 |

**LAG / LEAD** — look at previous / next row:
```sql
SELECT order_id, order_date, total_amount,
       LAG(total_amount)  OVER (ORDER BY order_date) AS prev_amount,
       LEAD(order_date)   OVER (ORDER BY order_date) AS next_order_date,
       total_amount - LAG(total_amount, 1, 0) OVER (ORDER BY order_date) AS diff
FROM orders;
```

**Running total and moving average:**
```sql
SELECT order_id, order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date
                               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
       AVG(total_amount) OVER (ORDER BY order_date
                               ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)         AS moving_avg_3
FROM orders WHERE status <> 'CANCELLED';
```

**Aggregate as window — compare each row to its group:**
```sql
SELECT name, dept_id, salary,
       AVG(salary) OVER (PARTITION BY dept_id)           AS dept_avg,
       salary - MAX(salary) OVER (PARTITION BY dept_id)  AS gap_from_top
FROM employees;
```

Others: `NTILE(4)` (quartiles), `FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE`, `PERCENT_RANK`, `CUME_DIST`.

**Traps:**
1. You **can't** filter a window function in `WHERE` (it's computed after WHERE). Wrap it in a subquery/CTE and filter outside.
2. Default frame with ORDER BY is `RANGE ... CURRENT ROW` — rows with **same ORDER BY value** are included together. Use `ROWS` for a strict per-row running total.
3. `LAST_VALUE` with default frame returns the current row — set frame to `UNBOUNDED FOLLOWING`.

### 6.8 CTEs and Recursive CTE ⭐

**Simple explanation:** A **CTE** (`WITH name AS (...)`) is a named temporary result for one query — like a variable that holds a sub-result. Makes long queries readable. A **recursive CTE** calls itself — perfect for trees (org charts, categories, graph paths).

```sql
-- Normal CTE
WITH dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id
)
SELECT e.name, e.salary, ROUND(d.avg_sal) AS dept_avg
FROM employees e JOIN dept_avg d USING (dept_id)
WHERE e.salary > d.avg_sal;
```

**Recursive CTE — org chart (who reports to whom, at what level):**
```sql
WITH RECURSIVE org AS (
    -- 1) Anchor: top bosses (no manager)
    SELECT emp_id, name, manager_id, 0 AS level, name::TEXT AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- 2) Recursive step: people whose manager is already in "org"
    SELECT e.emp_id, e.name, e.manager_id, o.level + 1, o.path || ' > ' || e.name
    FROM employees e
    JOIN org o ON e.manager_id = o.emp_id
)
SELECT level, name, path FROM org ORDER BY path;
```
| level | name | path |
|---|---|---|
| 0 | Asha | Asha |
| 1 | Arjun | Asha > Arjun |
| 1 | Meera | Asha > Meera |
| 1 | Neha | Asha > Neha |
| 1 | Ravi | Asha > Ravi |
| 0 | Karan | Karan |
| 1 | Priya | Karan > Priya |
| 1 | Vikram | Karan > Vikram |

How it runs: anchor gives level 0 → step joins to get level 1 → repeat until the step returns no new rows. Use `UNION ALL` (faster); add a depth limit (`WHERE o.level < 10`) or cycle check (`CYCLE emp_id SET is_cycle USING path_arr` in PG 14+) for graphs that may have loops.

**Other recursive uses:** generate series of dates, all sub-categories of a category, "all reports under manager X" (anchor = `WHERE emp_id = X`).

**Trap:** In Postgres 12+, a non-recursive CTE used once is **inlined** (optimized like a subquery). Before PG 12 CTEs were an "optimization fence". `WITH x AS MATERIALIZED (...)` forces materialization.

### 6.9 Views vs Materialized Views

**Simple explanation:** A **view** is a saved query (a window into tables — no data stored). A **materialized view** stores the result physically (a photo — fast to read, but goes stale).

```sql
CREATE VIEW v_emp_public AS
SELECT emp_id, name, dept_id FROM employees;          -- hides salary (security)

CREATE MATERIALIZED VIEW mv_monthly_revenue AS
SELECT date_trunc('month', order_date) AS month, SUM(total_amount) AS revenue
FROM orders WHERE status <> 'CANCELLED'
GROUP BY 1;

REFRESH MATERIALIZED VIEW CONCURRENTLY mv_monthly_revenue;  -- needs a UNIQUE index on the MV
```

| | View | Materialized View |
|---|---|---|
| Stores data? | No (query runs each time) | Yes |
| Always fresh? | Yes | No — until `REFRESH` |
| Speed | Same as underlying query | Fast reads |
| Indexable? | No | Yes |
| Use for | Security, simplifying queries, logical data independence | Dashboards, heavy aggregates |

Simple views (one table, no aggregates) are **updatable** in Postgres.

### 6.10 Stored Procedures and Functions

**Simple explanation:** Code stored and run **inside** the DB. Fewer network round trips; logic close to data. Downside: harder to version, test and scale than app code.

```sql
-- Function (returns a value, usable in SELECT): Nth highest salary
CREATE OR REPLACE FUNCTION nth_highest_salary(n INT)
RETURNS NUMERIC LANGUAGE sql AS $$
    SELECT DISTINCT salary FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET n - 1;
$$;
SELECT nth_highest_salary(3);   -- 120000

-- Procedure (CALL, can manage transactions): money transfer
CREATE OR REPLACE PROCEDURE transfer(from_id INT, to_id INT, amt NUMERIC)
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE accounts SET balance = balance - amt WHERE account_id = from_id;
    UPDATE accounts SET balance = balance + amt WHERE account_id = to_id;
    -- CHECK (balance >= 0) raises an error if from_id lacks money -> whole call rolls back
END;
$$;
CALL transfer(1, 2, 1000);
```

| | Function | Procedure |
|---|---|---|
| Called by | `SELECT f()` | `CALL p()` |
| Returns | Value / table | Nothing (OUT params allowed) |
| COMMIT inside? | No | Yes (PG 11+) |

### 6.11 Triggers

**Simple explanation:** A trigger is code the DB runs **automatically** on INSERT/UPDATE/DELETE — like a CCTV camera that logs every time a door opens.

```sql
CREATE TABLE salary_audit (
    emp_id INT, old_salary NUMERIC, new_salary NUMERIC, changed_at TIMESTAMPTZ DEFAULT now()
);

CREATE OR REPLACE FUNCTION log_salary_change() RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    IF NEW.salary <> OLD.salary THEN
        INSERT INTO salary_audit(emp_id, old_salary, new_salary)
        VALUES (OLD.emp_id, OLD.salary, NEW.salary);
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_salary_audit
AFTER UPDATE OF salary ON employees
FOR EACH ROW EXECUTE FUNCTION log_salary_change();

UPDATE employees SET salary = 130000 WHERE emp_id = 2;   -- audit row created automatically
```

Key points: `BEFORE` (can modify `NEW` / reject) vs `AFTER` (row is final); `FOR EACH ROW` vs `FOR EACH STATEMENT`; `OLD` and `NEW` records.

**Trap:** Triggers are "hidden logic" — slow bulk writes, surprise side effects, hard to debug. Use for audit/`updated_at`, not business workflows.

### 6.12 Indexes in SQL

```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);                         -- B-tree (default)
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date DESC);       -- composite
CREATE UNIQUE INDEX uq_customers_email ON customers(lower(email));               -- expression + unique
CREATE INDEX idx_orders_open ON orders(order_date) WHERE status = 'PLACED';      -- partial index
CREATE INDEX idx_orders_cover ON orders(customer_id) INCLUDE (total_amount);     -- covering
CREATE INDEX CONCURRENTLY idx_emp_dept ON employees(dept_id);                    -- no long write lock
DROP INDEX idx_emp_dept;
```
Postgres index types: **B-tree** (default; =, <, >, BETWEEN, ORDER BY, prefix LIKE), **Hash** (= only), **GIN** (arrays, JSONB, full-text), **GiST** (geo, ranges), **BRIN** (huge naturally ordered tables like time-series logs).

**Tip:** On a live production table use `CREATE INDEX CONCURRENTLY` — normal `CREATE INDEX` blocks writes until done. Theory in Section 12.

### 6.13 EXPLAIN

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 1;            -- plan + estimates only
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 1;    -- actually runs it, real timings
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;                         -- + pages read from cache/disk
```
**Trap:** `EXPLAIN ANALYZE` **executes** the query. For `UPDATE/DELETE`, wrap in `BEGIN; ... ROLLBACK;`. Full reading guide with before/after output in Section 13.3.

---

## 7. SQL Practice: 25 Most-Asked Interview Queries

All use the Section 1 schema. Write each by hand at least once. Expected outputs are for the original sample data.

Schemas used: `employees(emp_id, name, email, dept_id, manager_id, salary, hire_date)`, `departments(dept_id, name, location)`, `customers(customer_id, name, city, email)`, `products(product_id, name, category, price, stock)`, `orders(order_id, customer_id, order_date, status, total_amount)`, `order_items(order_id, product_id, quantity, unit_price)`, `logs(id, num)`, `contacts(id, email)`.

**Q1. ⭐ Second highest salary**
```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);     -- 150000
```
Max below the max. Returns NULL (not "no row") if there is no second salary — interviewers like that.

**Q2. ⭐ Nth highest salary (N = 3)**
```sql
-- Way 1: DENSE_RANK (handles ties correctly)
SELECT DISTINCT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk FROM employees
) t WHERE rnk = 3;                                       -- 120000

-- Way 2: LIMIT/OFFSET on distinct values
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 2;

-- Way 3: correlated (no window functions): N-1 distinct salaries are greater
SELECT DISTINCT salary FROM employees e1
WHERE 2 = (SELECT COUNT(DISTINCT salary) FROM employees e2 WHERE e2.salary > e1.salary);
```
Use DENSE_RANK so duplicates (95000, 95000) don't shift the answer; know Way 3 for "without LIMIT/window".

**Q3. ⭐ Find duplicate emails**
```sql
SELECT email, COUNT(*) AS cnt
FROM contacts
GROUP BY email
HAVING COUNT(*) > 1;                                     -- a@x.com 3, b@x.com 2
```
Group by the column that should be unique; HAVING keeps groups with more than one row.

**Q4. ⭐ Delete duplicates, keep the lowest id**
```sql
-- Way 1: self-join DELETE ... USING (Postgres)
DELETE FROM contacts a
USING contacts b
WHERE a.email = b.email AND a.id > b.id;                 -- deletes ids 3, 5, 6

-- Way 2: ROW_NUMBER
DELETE FROM contacts
WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn FROM contacts
    ) t WHERE rn > 1
);
```
Delete every row that has a "twin" with a smaller id. (No PK at all? Use Postgres hidden `ctid` instead of `id`.) Then add `UNIQUE(email)` so it can't happen again.

**Q5. ⭐ Employees earning more than their manager**
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;                               -- Neha, Priya, Vikram
```
Self join: one copy of the table plays "employee", the other plays "manager".

**Q6. ⭐ Highest-paid employee(s) in each department**
```sql
SELECT d.name AS dept, e.name, e.salary
FROM employees e JOIN departments d ON d.dept_id = e.dept_id
WHERE (e.dept_id, e.salary) IN (SELECT dept_id, MAX(salary) FROM employees GROUP BY dept_id);
-- Engineering Neha 160000 · Sales Priya 95000 · Sales Vikram 95000 · HR Meera 60000
```
Tuple IN with a grouped subquery returns all ties (ROW_NUMBER would wrongly keep only one).

**Q7. ⭐ Top 2 salaries in each department**
```sql
SELECT dept_id, name, salary FROM (
    SELECT dept_id, name, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees WHERE dept_id IS NOT NULL
) t
WHERE rnk <= 2
ORDER BY dept_id, salary DESC;
-- 1: Neha, Asha · 2: Priya, Vikram, Karan · 3: Meera
```
PARTITION BY = per department; filter the rank outside because WHERE can't see window results. Use ROW_NUMBER if they want exactly 2 rows per dept.

**Q8. Second highest salary in each department**
```sql
SELECT dept_id, name, salary FROM (
    SELECT dept_id, name, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees
) t WHERE rnk = 2;                                       -- Asha (1), Karan (2); HR has none
```
Same pattern as Q7 with `= 2`; DENSE_RANK so the tie in Sales gives Karan, not nobody.

**Q9. ⭐ Customers who never ordered**
```sql
-- Way 1: NOT EXISTS (NULL-safe, usually best)
SELECT c.name FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);   -- Divya

-- Way 2: LEFT JOIN anti-join
SELECT c.name FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL;
```
Anti-join. Avoid `NOT IN` when the subquery column can be NULL — it returns zero rows.

**Q10. Products never ordered**
```sql
SELECT p.name FROM products p
WHERE NOT EXISTS (SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id);  -- Pen
```
Same anti-join pattern on a different table.

**Q11. ⭐ Count employees per department, including empty ones**
```sql
SELECT d.name, COUNT(e.emp_id) AS emp_count
FROM departments d
LEFT JOIN employees e ON e.dept_id = d.dept_id
GROUP BY d.name
ORDER BY emp_count DESC;                    -- Engineering 3, Sales 3, HR 1, Marketing 0
```
LEFT JOIN from departments and `COUNT(e.emp_id)` (not `COUNT(*)`, which would give Marketing 1).

**Q12. ⭐ Numbers that appear at least 3 times consecutively**
```sql
SELECT DISTINCT num FROM (
    SELECT num,
           LAG(num)  OVER (ORDER BY id) AS prev_num,
           LEAD(num) OVER (ORDER BY id) AS next_num
    FROM logs
) t
WHERE num = prev_num AND num = next_num;                 -- 1 (ids 1,2,3)
```
Compare each row with its neighbours using LAG/LEAD. (Old way: 3-way self join on id, id+1, id+2 — breaks if ids have gaps.)

**Q13. ⭐ Running total of revenue by date**
```sql
SELECT order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date, order_id
                               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders WHERE status <> 'CANCELLED';
-- 62000 → 82000 → 83000 → 85500 → 149500
```
SUM as a window with an ordered frame; `ROWS` avoids grouping same-date rows together.

**Q14. Monthly revenue**
```sql
SELECT date_trunc('month', order_date)::DATE AS month,
       SUM(total_amount) AS revenue, COUNT(*) AS orders
FROM orders
WHERE status <> 'CANCELLED'
GROUP BY 1 ORDER BY 1;
-- 2026-01-01 82000 2 · 2026-02-01 1000 1 · 2026-03-01 66500 2
```
`date_trunc` buckets dates into months (`EXTRACT(MONTH FROM d)` alone mixes different years).

**Q15. Month-over-month growth %**
```sql
WITH m AS (
    SELECT date_trunc('month', order_date) AS month, SUM(total_amount) AS revenue
    FROM orders WHERE status <> 'CANCELLED' GROUP BY 1
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev,
       ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
             / NULLIF(LAG(revenue) OVER (ORDER BY month), 0), 2) AS growth_pct
FROM m;
```
Aggregate first in a CTE, then LAG over months; `NULLIF(...,0)` prevents divide-by-zero.

**Q16. Date-based: orders in the last 30 days / in a given month / weekday**
```sql
SELECT * FROM orders WHERE order_date >= CURRENT_DATE - INTERVAL '30 days';
SELECT * FROM orders WHERE order_date >= '2026-02-01' AND order_date < '2026-03-01';  -- index-friendly
SELECT order_id, to_char(order_date, 'Day') AS weekday FROM orders;
SELECT name, AGE(CURRENT_DATE, hire_date) AS tenure FROM employees;
```
Use a half-open range (`>= start AND < next`) instead of `EXTRACT(MONTH FROM order_date) = 2` so an index on `order_date` can be used.

**Q17. ⭐ Pivot-style count: orders per status for each customer**
```sql
SELECT c.name,
       COUNT(o.order_id) FILTER (WHERE o.status = 'DELIVERED') AS delivered,
       COUNT(o.order_id) FILTER (WHERE o.status = 'SHIPPED')   AS shipped,
       COUNT(o.order_id) FILTER (WHERE o.status = 'PLACED')    AS placed,
       SUM(CASE WHEN o.status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled  -- portable way
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.name ORDER BY c.name;
```
| name | delivered | shipped | placed | cancelled |
|---|---|---|---|---|
| Divya | 0 | 0 | 0 | 0 |
| Imran | 1 | 0 | 0 | 1 |
| Rohan | 1 | 1 | 0 | 0 |
| Sara | 1 | 0 | 1 | 0 |

Conditional aggregation turns row values into columns. `FILTER` is Postgres; `SUM(CASE ...)` works everywhere.

**Q18. ⭐ Top 3 customers by total spend**
```sql
SELECT c.name, SUM(o.total_amount) AS spend
FROM customers c JOIN orders o ON o.customer_id = c.customer_id
WHERE o.status <> 'CANCELLED'
GROUP BY c.customer_id, c.name
ORDER BY spend DESC
LIMIT 3;                                    -- Imran 64000, Rohan 63000, Sara 22500
```
Join, group, order, limit. Group by the id too — two customers can share a name.

**Q19. Employees earning above their department average**
```sql
SELECT name, dept_id, salary FROM (
    SELECT name, dept_id, salary, AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg
    FROM employees
) t WHERE salary > dept_avg;                -- Asha, Neha, Priya, Vikram
```
Window AVG avoids re-scanning per row (the correlated-subquery version is in 6.4).

**Q20. Percentage of revenue by category**
```sql
SELECT p.category,
       SUM(oi.quantity * oi.unit_price) AS revenue,
       ROUND(100.0 * SUM(oi.quantity * oi.unit_price)
             / SUM(SUM(oi.quantity * oi.unit_price)) OVER (), 2) AS pct
FROM order_items oi JOIN products p ON p.product_id = oi.product_id
GROUP BY p.category;
```
`SUM(SUM(x)) OVER ()` = grand total across all groups — a window over the aggregated result.

**Q21. First order of each customer**
```sql
SELECT DISTINCT ON (customer_id) customer_id, order_id, order_date
FROM orders
ORDER BY customer_id, order_date;           -- 1→101, 2→102, 3→104
```
Postgres `DISTINCT ON` keeps the first row per group by the ORDER BY; portable version is `ROW_NUMBER() ... WHERE rn = 1`.

**Q22. Customers who bought both Laptop and Headphones**
```sql
SELECT o.customer_id
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
JOIN products p     ON p.product_id = oi.product_id
WHERE p.name IN ('Laptop', 'Headphones')
GROUP BY o.customer_id
HAVING COUNT(DISTINCT p.name) = 2;          -- 1 (Rohan), 3 (Imran)
```
Filter to the wanted items, then require the group to contain all of them (`HAVING COUNT(DISTINCT) = k`).

**Q23. Managers with at least 2 direct reports**
```sql
SELECT m.name, COUNT(*) AS reports
FROM employees e JOIN employees m ON e.manager_id = m.emp_id
GROUP BY m.emp_id, m.name
HAVING COUNT(*) >= 2;                       -- Asha 4, Karan 2
```
Self join + GROUP BY on the manager side.

**Q24. Median salary**
```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median FROM employees;   -- 95000
```
Ordered-set aggregate; `PERCENTILE_DISC` returns an actual value from the column instead of interpolating.

**Q25. Fetch odd rows, and find missing IDs (gaps)**
```sql
-- Odd rows by position
SELECT * FROM (SELECT *, ROW_NUMBER() OVER (ORDER BY emp_id) AS rn FROM employees) t
WHERE rn % 2 = 1;

-- Missing order ids between min and max
SELECT g.id AS missing_id
FROM generate_series((SELECT MIN(order_id) FROM orders), (SELECT MAX(order_id) FROM orders)) AS g(id)
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.order_id = g.id);   -- none here
```
Tables have no inherent row order — number rows with ROW_NUMBER first; `generate_series` builds the "full list" to anti-join against.

**Bonus patterns to recognise quickly**

| Ask | Pattern |
|---|---|
| "Top N per group" | `ROW_NUMBER/DENSE_RANK OVER (PARTITION BY g ORDER BY x DESC)` in subquery, filter outside |
| "X that never Y" | `NOT EXISTS` / `LEFT JOIN ... IS NULL` |
| "X that did all of Y" | `GROUP BY ... HAVING COUNT(DISTINCT) = k` or double `NOT EXISTS` |
| "Compare with previous row" | `LAG/LEAD` |
| "Cumulative" | `SUM() OVER (ORDER BY ...)` |
| "Duplicates" | `GROUP BY ... HAVING COUNT(*) > 1` |
| "Hierarchy" | `WITH RECURSIVE` |
| "Rows to columns" | `COUNT/SUM ... FILTER` or `SUM(CASE ...)` |
| "Swap values" | `UPDATE t SET sex = CASE sex WHEN 'm' THEN 'f' ELSE 'm' END;` |

---

## 8. Functional Dependencies and Normalization

### 8.1 Functional Dependency (FD)

**Simple explanation:** `X → Y` means "if two rows have the same X, they **must** have the same Y". X *determines* Y. Example: `roll_no → name` (one roll number, one name). But `name → roll_no` is false (two students can be called Rahul).

**Key terms:**
- **Trivial FD:** Y ⊆ X (`{roll, name} → name`). Always true, useless.
- **Non-trivial:** Y not inside X.
- **Determinant:** left side X.
- **Partial dependency:** non-prime attribute depends on **part** of a composite candidate key.
- **Transitive dependency:** `A → B` and `B → C` where B is not a key, so `A → C` "through" B.

**Armstrong's axioms** (sound and complete — every FD can be derived with these):

| Rule | Statement |
|---|---|
| Reflexivity | If Y ⊆ X then X → Y |
| Augmentation | If X → Y then XZ → YZ |
| Transitivity | If X → Y and Y → Z then X → Z |
| *Derived:* Union | X → Y, X → Z ⇒ X → YZ |
| *Derived:* Decomposition | X → YZ ⇒ X → Y and X → Z |
| *Derived:* Pseudo-transitivity | X → Y, WY → Z ⇒ WX → Z |

### 8.2 Attribute closure X⁺ and finding candidate keys ⭐

**Simple explanation:** X⁺ = every attribute you can "reach" from X using the FDs. If X⁺ = all attributes, X is a super key. If no smaller subset also reaches everything, X is a **candidate key**.

**Algorithm:** start `result = X`; repeat: for each FD `A → B`, if A ⊆ result, add B; stop when nothing changes.

**Example 1:** R(A, B, C, D, E), F = { A → B, B → C, CD → E }
- A⁺: {A} → A→B → {A,B} → B→C → {A,B,C}. CD→E needs D — not there. **A⁺ = {A,B,C}** (not a key).
- Trick: attributes **never on any right side** must be in every key. Here A and D never appear on the right.
- (AD)⁺ = {A,D} → B → C → (CD→E) E = **{A,B,C,D,E}** ✓. Subsets A⁺ and D⁺ don't cover all → **AD is the only candidate key**.
- Prime attributes: A, D. Non-prime: B, C, E.

**Example 2:** R(A, B, C, D), F = { A → B, B → C, C → A }
- D never appears on the right → D is in every key. D⁺ = {D} — not enough.
- (AD)⁺ = {A,D,B,C} ✓, (BD)⁺ = {B,D,C,A} ✓, (CD)⁺ = {C,D,A,B} ✓.
- **Candidate keys: AD, BD, CD.** Prime attributes: A, B, C, D (all).

**Interview tip:** Always start with "attributes not on any RHS must be in the key", then grow from there. Also asked: **canonical / minimal cover** — split RHS to single attributes, remove extraneous LHS attributes, remove redundant FDs.

### 8.3 Why normalize? Anomalies

**Simple explanation:** Redundancy (same fact stored many times) causes 3 problems:

| Anomaly | Example in an unnormalised `orders_flat` table |
|---|---|
| **Insert** | Can't add a new product until someone orders it (no order_id for the key) |
| **Update** | Customer moves city → must update 50 rows; miss one = inconsistent data |
| **Delete** | Delete the only order of a product → you lose the product's price too |

Normalization = splitting tables so **each fact is stored once**.

### 8.4 Normal forms step by step (one table) ⭐⭐

**Start:** an order sheet someone made in Excel.

`OrderSheet`

| order_id | order_date | customer_id | customer_name | customer_city | products |
|---|---|---|---|---|---|
| 101 | 2026-01-05 | 1 | Rohan | Delhi | Laptop(60000)x1, Headphones(2000)x1 |
| 103 | 2026-02-10 | 1 | Rohan | Delhi | Book(500)x2 |

**1NF — atomic values, no repeating groups.** `products` holds a list → not atomic. Fix: one row per (order, product).

`Order1NF` — key = **(order_id, product_id)**

| order_id | order_date | customer_id | customer_name | customer_city | product_id | product_name | unit_price | qty |
|---|---|---|---|---|---|---|---|---|
| 101 | 2026-01-05 | 1 | Rohan | Delhi | 1 | Laptop | 60000 | 1 |
| 101 | 2026-01-05 | 1 | Rohan | Delhi | 3 | Headphones | 2000 | 1 |
| 103 | 2026-02-10 | 1 | Rohan | Delhi | 4 | Book | 500 | 2 |

FDs:
- `order_id → order_date, customer_id`
- `customer_id → customer_name, customer_city`
- `product_id → product_name, unit_price`
- `(order_id, product_id) → qty`

**2NF — 1NF + no partial dependency** (no non-prime column depends on only *part* of the composite key). Here `order_date` depends only on `order_id`, `product_name` only on `product_id` → partial dependencies. Split by "what does each column really depend on":

- `Orders2(order_id PK, order_date, customer_id, customer_name, customer_city)`
- `Products(product_id PK, product_name, unit_price)`
- `OrderItems(order_id, product_id, qty)` — PK (order_id, product_id)

(A table with a **single-column** key is automatically in 2NF.)

**3NF — 2NF + no transitive dependency** (non-key → non-key). In `Orders2`: `order_id → customer_id → customer_name, customer_city`. City is a fact about the customer, not the order. Split:

- `Orders(order_id PK, order_date, customer_id FK)`
- `Customers(customer_id PK, customer_name, customer_city)`
- `Products(...)`, `OrderItems(...)` unchanged

Formal 3NF test: for every non-trivial FD `X → A`, **X is a super key OR A is a prime attribute**.

Memory line: *"Every non-key column depends on the key (1NF), the whole key (2NF), and nothing but the key (3NF)."*

**BCNF (Boyce-Codd) — for every non-trivial FD `X → Y`, X must be a super key.** Stricter than 3NF (removes the "A is prime" escape). 3NF-but-not-BCNF only happens with **overlapping candidate keys**.

Classic example: `Teaching(student, subject, professor)`
- Rule: each professor teaches only one subject → `professor → subject`.
- A student takes a subject from one professor → `(student, subject) → professor`.
- Candidate keys: (student, subject) and (student, professor).
- `professor → subject`: professor is **not** a super key, but subject **is prime** → 3NF OK, **BCNF violated**.

| student | subject | professor |
|---|---|---|
| Rohan | DBMS | Dr. Rao |
| Sara | DBMS | Dr. Rao |
| Rohan | OS | Dr. Iyer |

Anomaly: if Dr. Rao switches to teaching CN, you must update many rows. Decompose:
- `ProfSubject(professor PK, subject)`
- `StudentProf(student, professor)` — PK (student, professor)

This is **lossless** (common column `professor` is a key of `ProfSubject`), **but not dependency preserving**: `(student, subject) → professor` can no longer be checked inside one table. That's the BCNF trade-off.

**4NF — BCNF + no multi-valued dependency (MVD).** `X →→ Y` means X determines a *set* of Y values independent of other columns.

| emp | skill | language |
|---|---|---|
| Asha | Go | Hindi |
| Asha | Go | English |
| Asha | SQL | Hindi |
| Asha | SQL | English |

Skills and languages are independent → every combination must be stored (2 × 2 rows). Split into `EmpSkill(emp, skill)` and `EmpLanguage(emp, language)`.

**5NF (PJNF)** — no join dependency that isn't implied by keys. Rarely asked; just name it.

| NF | Rule (one line) |
|---|---|
| 1NF | Atomic values, no repeating groups |
| 2NF | 1NF + no partial dependency on a composite key |
| 3NF | 2NF + no transitive dependency (X super key or A prime) |
| BCNF | Every determinant is a super key |
| 4NF | BCNF + no non-trivial multi-valued dependency |
| 5NF | No non-trivial join dependency |

### 8.5 Decomposition properties: lossless join and dependency preservation ⭐

**Lossless join:** after splitting R into R1 and R2, joining them gives back **exactly** R (no fake extra rows). Test for binary split:
> R1 ∩ R2 → R1 **or** R1 ∩ R2 → R2 (the common columns are a key of at least one side).

Example: `Orders(order_id, date, customer_id)` and `Customers(customer_id, name, city)` share `customer_id`, which is the key of Customers → lossless ✓.
Lossy example: splitting `(student, subject, professor)` into `(student, subject)` and `(subject, professor)` — common `subject` is a key of neither side → joining creates fake rows (Sara paired with Dr. Iyer if Iyer also taught DBMS).

**Dependency preservation:** every original FD can be checked inside some single decomposed table (without joins). (F1 ∪ F2 ∪ ...)⁺ = F⁺.

| | 3NF | BCNF |
|---|---|---|
| Lossless | Always possible | Always possible |
| Dependency preserving | **Always possible** | **Not always** |
| Redundancy | A little possible | Less |

**Interview tip:** "Why not always BCNF?" → It may lose dependency preservation; 3NF is the practical target for most OLTP schemas.

### 8.6 Denormalization

**Simple explanation:** Deliberately adding redundancy back to make **reads faster** (fewer joins), accepting extra write work and risk of inconsistency.

Examples:
- Store `orders.total_amount` instead of summing `order_items` every time (our schema does this).
- Store `customer_name` on an order for invoices (also preserves history).
- Counters (`post.like_count`), materialized views, read models in a cache, analytics star schemas.

How to keep it correct: update in the same transaction, triggers, or async via events (outbox/CDC), or a periodic rebuild.

**Interview tip:** "Normalize for writes and correctness, denormalize for read-heavy hot paths — and measure first." OLTP = normalized; OLAP/data warehouse = denormalized (star/snowflake schemas).

---

## 9. Transactions

### 9.1 What is a transaction?

**Simple explanation:** A transaction is a group of operations that must succeed **together or not at all**. Bank transfer of ₹1000 from Rohan to Sara = (1) subtract from Rohan, (2) add to Sara. If the server crashes between 1 and 2, money vanishes — unless both are in one transaction.

### 9.2 ACID ⭐⭐

| Property | Meaning (bank transfer) | How the DB provides it |
|---|---|---|
| **Atomicity** | Both updates happen or neither | Undo info / WAL; rollback on failure |
| **Consistency** | Rules hold before and after (balance ≥ 0, total money unchanged) | Constraints, triggers + correct app logic |
| **Isolation** | Another transaction never sees "money left Rohan but not reached Sara" | Locks / MVCC, isolation levels |
| **Durability** | Once COMMIT returns, the transfer survives a crash | WAL flushed (fsync) to disk before commit returns; replication |

> **Project:** "The **transactional outbox** in LocalArena relies on atomicity: the purchase row and its event row commit together or not at all. That's how I got **0 lost events during a 60 s Kafka outage**."

### 9.3 Transaction code: BEGIN / COMMIT / ROLLBACK / SAVEPOINT

```sql
-- Happy path
BEGIN;
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;   -- Rohan 5000 -> 4000
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;   -- Sara  3000 -> 4000
COMMIT;

-- Failure: CHECK (balance >= 0) fails -> transaction is aborted
BEGIN;
UPDATE accounts SET balance = balance - 9000 WHERE account_id = 1;   -- ERROR: violates check constraint
UPDATE accounts SET balance = balance + 9000 WHERE account_id = 2;   -- ERROR: current transaction is aborted
ROLLBACK;                                                             -- nothing changed

-- SAVEPOINT: partial rollback inside a transaction
BEGIN;
INSERT INTO orders VALUES (107, 4, CURRENT_DATE, 'PLACED', 500);
SAVEPOINT before_items;
INSERT INTO order_items VALUES (107, 99, 1, 500);   -- ERROR: product 99 doesn't exist (FK)
ROLLBACK TO SAVEPOINT before_items;                 -- undo only the failed part
INSERT INTO order_items VALUES (107, 4, 1, 500);    -- correct product
RELEASE SAVEPOINT before_items;
COMMIT;                                              -- order 107 + valid item saved
```

**Trap:** In PostgreSQL, after any error inside a transaction, every later statement fails with "current transaction is aborted" until you `ROLLBACK` (or roll back to a savepoint). Also, `psql`/most drivers are in **autocommit** mode by default — each statement is its own transaction unless you `BEGIN`.

### 9.4 Transaction states

```
             +-----------> Failed -----------> Aborted (rolled back)
             |                                   |-> restart or kill
 Active --> Partially committed --> Committed
 (running)  (last stmt done,        (WAL durable)
            not yet durable)
                     |
                     +---> Failed (e.g., crash before flush)
 Committed and Aborted are both "Terminated".
```

| State | Meaning |
|---|---|
| Active | Executing statements |
| Partially committed | Last statement executed, commit not yet durable |
| Committed | Changes permanent |
| Failed | Error found, can't continue |
| Aborted | Rolled back; DB restored to before txn |
| Terminated | Finished (committed or aborted) |

### 9.5 Schedules and serializability ⭐

**Simple explanation:** A **schedule** is the order in which operations of several concurrent transactions are interleaved. **Serial schedule** = one transaction fully, then the next (always correct, but no concurrency). A schedule is **serializable** if its result equals *some* serial order — we get concurrency **and** correctness.

Notation: `R1(A)` = T1 reads A, `W2(B)` = T2 writes B.

**Conflict:** two operations conflict if they are from **different transactions**, on the **same item**, and **at least one is a write**: R-W, W-R, W-W. (R-R never conflicts.)

**Conflict serializable** = can be turned into a serial schedule by swapping adjacent **non-conflicting** operations. Test using a **precedence graph**:
1. One node per transaction.
2. Edge Ti → Tj if an operation of Ti conflicts with a **later** operation of Tj.
3. **Cycle → not conflict serializable. No cycle → serializable**; a topological sort gives the equivalent serial order.

**Example A (serializable):**
```
S1: R1(A) W1(A) R2(A) W2(A) R1(B) W1(B) R2(B) W2(B)
Conflicts: W1(A) before R2(A)/W2(A)  -> T1 → T2
           W1(B) before R2(B)/W2(B)  -> T1 → T2
Graph: T1 → T2   (no cycle)  => equivalent to serial T1, T2 ✓
```

**Example B (not serializable):**
```
S2: R1(A) R2(A) W1(A) W2(A)
Conflicts: R1(A) before W2(A) -> T1 → T2
           R2(A) before W1(A) -> T2 → T1
Graph: T1 ⇄ T2  (cycle)  => NOT conflict serializable ✗  (this is the lost update!)
```

**View serializability** (weaker, more schedules allowed): schedules S and S' are view-equivalent if (1) same transaction does the **initial read** of each item, (2) each read reads the value written by the **same** writer, (3) same transaction does the **final write**.
- Every conflict-serializable schedule is view-serializable, **not** vice versa.
- Non-conflict but view serializable schedules always contain **blind writes** (write without read).
- Example: `R1(A) W2(A) W1(A) W3(A)` — graph has cycle T1⇄T2, but it's view-equivalent to T1, T2, T3 (T1 reads initial A, T3 does final write) → view serializable.
- Testing view serializability is **NP-complete**, so real DBs use conflict-based methods (locking, MVCC + SSI).

### 9.6 Recoverable, cascadeless and strict schedules

| Type | Rule | Why |
|---|---|---|
| **Recoverable** | If T2 reads data written by T1, then T1 **commits before** T2 commits | Otherwise T2 committed using data that may be rolled back → irrecoverable |
| **Cascadeless** (ACA) | Transactions read only **committed** data | Prevents cascading rollback (T1 aborts → T2, T3 that read from it must abort too) |
| **Strict** | No read **or write** of an item until the last writer of it commits/aborts | Makes undo simple (restore before-image) |

Hierarchy: **Strict ⊂ Cascadeless ⊂ Recoverable**. (Serial ⊂ Conflict-serializable ⊂ View-serializable is a separate axis.)

Irrecoverable example: `W1(A) R2(A) C2 ... A1` — T2 committed after reading T1's dirty A; T1 aborts later → can't undo T2.

**Interview tip:** Strict 2PL (next section) produces strict schedules — that's why databases use it.

---

## 10. Concurrency Control

### 10.1 Concurrency problems ⭐⭐

**Simple explanation:** When transactions run at the same time without enough isolation, they can see or overwrite each other's half-done work.

| Problem | What happens | Example |
|---|---|---|
| **Dirty read** | T2 reads T1's **uncommitted** change; T1 rolls back | T2 sees Rohan's balance 4000, T1 aborts, real balance is 5000 |
| **Lost update** | Two read-modify-write cycles; second write overwrites the first | Both read stock 10, both write 9 → one sale lost |
| **Non-repeatable read** | Same row read twice in one txn gives different values (someone committed an UPDATE in between) | Price 500, then 450 |
| **Phantom read** | Same **query** run twice returns **new/missing rows** (someone INSERTed/DELETEd) | `COUNT(*) WHERE status='PLACED'` = 1, then 2 |
| **Write skew** | Two txns read the same data, each updates a **different** row based on it, together they break a rule | Rule: ≥1 doctor on call. Both doctors check "2 on call", both go off call → 0 |

### 10.2 Isolation levels ⭐⭐

| Level | Dirty read | Non-repeatable read | Phantom | Lost update / write skew | PostgreSQL notes |
|---|---|---|---|---|---|
| Read Uncommitted | Possible (SQL std) | Possible | Possible | Possible | **Behaves as Read Committed** in PG (no dirty reads ever) |
| **Read Committed** | ✗ | Possible | Possible | Possible | **PG default.** Each *statement* sees a fresh snapshot |
| Repeatable Read | ✗ | ✗ | Possible (std) — **✗ in PG** | Lost update ✗ (error), write skew possible | Snapshot Isolation: one snapshot per *transaction*; conflicting update → error `40001` |
| Serializable | ✗ | ✗ | ✗ | ✗ | **SSI** (Serializable Snapshot Isolation): detects dangerous patterns, aborts one txn with `40001` — app must retry |

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
-- ...
COMMIT;
SHOW transaction_isolation;     -- read committed
```

MySQL InnoDB default = **Repeatable Read**. PostgreSQL / Oracle / SQL Server default = **Read Committed**.

**Interview tip:** Higher isolation → fewer anomalies but more blocking/aborts. At Repeatable Read and Serializable in Postgres, **your app must retry on serialization failure (SQLSTATE 40001)**.

### 10.3 Locking: shared and exclusive

**Simple explanation:** A lock is a "reserved" sign on data.
- **Shared (S) lock** — for reading. Many readers can hold it together.
- **Exclusive (X) lock** — for writing. Only one holder; blocks others.

Compatibility matrix:

| Held → / Requested ↓ | S | X |
|---|---|---|
| **S** | ✓ | ✗ |
| **X** | ✗ | ✗ |

**Granularity:** database → table → page → row. Finer = more concurrency, more lock overhead. **Intention locks** (IS, IX) on the table say "someone holds row locks inside" so a table lock can be checked quickly. **Lock escalation** (SQL Server) turns many row locks into one table lock.

PostgreSQL row-level lock modes (strongest → weakest):

| Lock | Taken by | Blocks |
|---|---|---|
| `FOR UPDATE` | `SELECT ... FOR UPDATE`, `DELETE`, UPDATE of key columns | all other row locks |
| `FOR NO KEY UPDATE` | normal `UPDATE` (non-key columns) | FOR UPDATE, FOR NO KEY UPDATE, FOR SHARE — but **not** FOR KEY SHARE |
| `FOR SHARE` | `SELECT ... FOR SHARE` | writers |
| `FOR KEY SHARE` | **Foreign key checks** (inserting a child row locks the parent row) | only FOR UPDATE (key changes / deletes) |

Readers in Postgres **don't take row locks** at all (MVCC) — plain `SELECT` never blocks `UPDATE` and vice versa.

### 10.4 Two-Phase Locking (2PL)

**Simple explanation:** Each transaction has two phases: a **growing phase** (only acquire locks) and a **shrinking phase** (only release). Once you release any lock, you can't take new ones. This guarantees **conflict serializability**.

```
 #locks
   ^        lock point
   |        /\
   |       /  \
   |      /    \
   +-----/------\------> time
     growing  shrinking
```

| Variant | Rule | Gives |
|---|---|---|
| Basic 2PL | grow then shrink | Serializable, but cascading rollbacks possible, deadlocks possible |
| **Strict 2PL** | Hold **exclusive** locks until commit/abort | Serializable + **strict/cascadeless** schedules. Used by most DBs |
| Rigorous 2PL | Hold **all** locks until commit | Serial order = commit order |
| Conservative 2PL | Take all locks **before** starting | **No deadlocks**, but must know data in advance; low concurrency |

**Trap:** 2PL (except conservative) does **not** prevent deadlocks.

### 10.5 Deadlocks in databases ⭐

**Simple explanation:** T1 holds row A and wants B; T2 holds B and wants A. Both wait forever — like two cars on a one-lane bridge.

```sql
-- Session 1                                   -- Session 2
BEGIN;                                         BEGIN;
UPDATE accounts SET balance=balance-100
 WHERE account_id=1;      -- locks row 1
                                               UPDATE accounts SET balance=balance-100
                                                WHERE account_id=2;    -- locks row 2
UPDATE accounts SET balance=balance+100
 WHERE account_id=2;      -- waits for S2
                                               UPDATE accounts SET balance=balance+100
                                                WHERE account_id=1;    -- waits for S1 -> cycle
                                               -- ERROR: deadlock detected (SQLSTATE 40P01)
                                               -- PG aborts one victim after deadlock_timeout (1 s)
```

**Handling strategies:**
- **Detection:** DB builds a **wait-for graph**; a cycle = deadlock; abort a victim (PostgreSQL does this).
- **Prevention:** lock resources in a **fixed global order** (always lower account_id first); conservative 2PL; timestamp schemes — **wait-die** (older waits, younger dies) and **wound-wait** (older wounds/aborts younger, younger waits).
- **Timeouts:** `lock_timeout`, `statement_timeout`.
- **Application:** keep transactions short; **retry** the victim with jittered backoff.

> **Project (your best story — S1):** "`SELECT ... FOR UPDATE` on the item row, while the purchase INSERT's foreign key took **`FOR KEY SHARE`** on the same item row. Every buyer waited for the others — **0 of 200 succeeded**. Fix: **`FOR NO KEY UPDATE`** (doesn't conflict with KEY SHARE) or a conditional update, plus **fixed lock ordering** and **jittered retry on `40P01`**. With the conditional update, 200 buyers / 20 items finished in **129 ms** (vs 832–984 ms for the other approaches)."

### 10.6 Timestamp ordering

**Simple explanation:** No locks. Each transaction gets a timestamp TS when it starts (older = smaller). The DB forces conflicts to happen **in timestamp order**. Each item keeps `read_TS` (largest TS that read it) and `write_TS` (largest TS that wrote it).

| Operation by Ti | Rule |
|---|---|
| Read(X) | If TS(Ti) < write_TS(X) → a younger txn already wrote X → **abort Ti** (restart with new TS). Else read, update read_TS |
| Write(X) | If TS(Ti) < read_TS(X) or TS(Ti) < write_TS(X) → **abort Ti**. Else write, set write_TS |

- Conflict serializable, **deadlock-free** (no waiting), but can cause **starvation** and many restarts; schedules may be non-recoverable unless extended.
- **Thomas write rule:** if TS(Ti) < write_TS(X) on a write, just **ignore** the obsolete write instead of aborting (allows some view-serializable schedules).

### 10.7 MVCC in PostgreSQL (simply) ⭐

**Simple explanation:** Instead of overwriting a row, Postgres writes a **new version** of it. Each transaction reads from a **snapshot** — a photo of committed data at a point in time. So **readers never block writers and writers never block readers.** Like Google Docs version history: you keep reading the version you opened.

How it works:
- Every row version (tuple) has hidden columns **`xmin`** (txn id that created it) and **`xmax`** (txn id that deleted/updated it, else 0).
- `UPDATE` = mark old version's `xmax` + insert a new version. `DELETE` = set `xmax`.
- A snapshot says which txn ids were committed when it was taken. A tuple is visible if `xmin` is committed-and-visible and `xmax` is not.
- Old, invisible versions = **dead tuples** → cleaned by **VACUUM** (autovacuum). Without it: **table bloat** and eventually **transaction ID wraparound** risk.
- Read Committed: new snapshot per statement. Repeatable Read/Serializable: one snapshot per transaction.

```sql
SELECT xmin, xmax, product_id, stock FROM products WHERE product_id = 1;
```

| | Locking (2PL) | MVCC |
|---|---|---|
| Readers block writers? | Yes (S vs X) | **No** |
| Storage | One version | Many versions → needs VACUUM |
| Writers vs writers | Block | Still block (row lock on update) |

**Trap:** MVCC does **not** remove the need for locks between **writers**. Two UPDATEs on the same row still serialize — and the lost update is still possible with read-modify-write in app code at Read Committed.

### 10.8 Lost update demo and the two fixes ⭐⭐

**Problem (Read Committed, read-modify-write in app code):**
```sql
-- Session A                                   -- Session B
BEGIN;                                         BEGIN;
SELECT stock FROM products WHERE product_id=1;
-- 10
                                               SELECT stock FROM products WHERE product_id=1;
                                               -- 10
-- app computes 10 - 1 = 9
UPDATE products SET stock = 9 WHERE product_id=1;
COMMIT;
                                               -- app computes 10 - 1 = 9
                                               UPDATE products SET stock = 9 WHERE product_id=1;
                                               COMMIT;
-- Two laptops sold, stock shows 9 instead of 8.  ONE UPDATE IS LOST.
```

**Fix 1 — Pessimistic locking: `SELECT ... FOR UPDATE`** (lock first, then think)
```sql
-- Session A                                   -- Session B
BEGIN;                                         BEGIN;
SELECT stock FROM products
 WHERE product_id=1 FOR UPDATE;   -- 10, row locked
                                               SELECT stock FROM products
                                                WHERE product_id=1 FOR UPDATE;
                                               -- BLOCKS, waiting for A
UPDATE products SET stock = 9 WHERE product_id=1;
COMMIT;                           -- lock released
                                               -- unblocks, now reads 9 (latest committed)
                                               UPDATE products SET stock = 8 WHERE product_id=1;
                                               COMMIT;                -- correct: 8
```
Variants: `FOR UPDATE NOWAIT` (error immediately if locked), `FOR UPDATE SKIP LOCKED` (skip locked rows — great for **job queues**: many workers each grab a different job). Prefer `FOR NO KEY UPDATE` if you won't change key columns (avoids blocking FK checks — your deadlock story).

**Fix 2 — Optimistic locking: version column** (don't lock, check at write time)
```sql
-- Both sessions read: stock = 10, version = 1
-- Session A
UPDATE products SET stock = 9, version = version + 1
WHERE product_id = 1 AND version = 1;          -- UPDATE 1  (success, version now 2)

-- Session B (still thinks version = 1)
UPDATE products SET stock = 9, version = version + 1
WHERE product_id = 1 AND version = 1;          -- UPDATE 0  -> conflict detected!
-- App: re-read (stock 9, version 2), recompute, retry.
```

**Fix 3 — Atomic conditional update** (no read-modify-write at all — best for counters/stock)
```sql
UPDATE products SET stock = stock - 1
WHERE product_id = 1 AND stock > 0
RETURNING stock;            -- 0 rows returned = sold out
```
The row lock taken by UPDATE + re-check of `stock > 0` on the latest version makes it safe even at Read Committed.

| | Pessimistic (`FOR UPDATE`) | Optimistic (version) | Conditional update |
|---|---|---|---|
| Idea | Lock before reading | Check version on write | Let DB do math + check atomically |
| Best when | High contention, short txns | **Low** contention, long "think time" (user editing a form) | Counters, stock, balances |
| Cost | Waiting, deadlock risk | Retries under contention | Very cheap; one statement |
| Failure | Blocks / deadlock `40P01` | 0 rows updated → retry | 0 rows → business rule says no |

> **Project:** "I used **optimistic locking for lobbies** (rare conflicts). For **hot purchases**, optimistic needed **206 retries** under contention, so I switched to the atomic **conditional update** `UPDATE ... SET stock = stock-1 WHERE stock > 0` — **129 ms for 200 buyers**."

**Interview trap:** "Is optimistic always faster?" No — under heavy contention retries explode (206 retries above). Pick by contention level.

### 10.9 Idempotency key pattern (INSERT ... ON CONFLICT)

**Simple explanation:** Networks retry. A client may send "buy item" twice. An **idempotency key** (a unique id per logical request) lets the DB itself reject the duplicate — a unique constraint is race-proof, an app-side "check then insert" is not.

```sql
CREATE TABLE idempotency_keys (
    player_id        BIGINT      NOT NULL,
    idempotency_key  UUID        NOT NULL,
    response         JSONB,
    created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (player_id, idempotency_key)        -- composite unique key
);

BEGIN;
INSERT INTO idempotency_keys (player_id, idempotency_key)
VALUES ($1, $2)
ON CONFLICT (player_id, idempotency_key) DO NOTHING
RETURNING player_id;
-- 1 row returned  -> first time: do the real work (grant reward, create order) in THIS txn,
--                    then UPDATE idempotency_keys SET response = ... ; COMMIT;
-- 0 rows returned -> duplicate: ROLLBACK and return the stored response
--                    (SELECT response FROM idempotency_keys WHERE player_id=$1 AND idempotency_key=$2)
COMMIT;
```
`ON CONFLICT ... DO UPDATE SET ...` = **upsert** (insert or update). Clean up old keys with a TTL job (`DELETE ... WHERE created_at < now() - interval '7 days'`).

> **Project:** "Composite key `(player_id, idempotency_key)`. Replaying **1.45M events gave 0 duplicate rewards** — deterministic IDs + dedupe + unique constraints."

### 10.10 Transactional outbox pattern

**Simple explanation:** Problem — "save order in DB **and** publish event to Kafka". If you do both separately, one can fail (dual-write problem). Fix — write the event into an **outbox table in the same DB transaction**. A separate relay (poller, or **Debezium CDC reading the WAL**) publishes it to Kafka later.

```sql
CREATE TABLE outbox (
    id             BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    aggregate_type TEXT        NOT NULL,      -- 'order'
    aggregate_id   TEXT        NOT NULL,      -- '108'
    event_type     TEXT        NOT NULL,      -- 'OrderPlaced'
    payload        JSONB       NOT NULL,
    created_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    published_at   TIMESTAMPTZ                -- set by poller (not needed with CDC)
);

BEGIN;
UPDATE products SET stock = stock - 1 WHERE product_id = 2 AND stock > 0;   -- must affect 1 row
INSERT INTO orders VALUES (108, 2, CURRENT_DATE, 'PLACED', 20000);
INSERT INTO order_items VALUES (108, 2, 1, 20000);
INSERT INTO outbox (aggregate_type, aggregate_id, event_type, payload)
VALUES ('order', '108', 'OrderPlaced',
        '{"order_id":108,"customer_id":2,"amount":20000}');
COMMIT;      -- order and event are saved together, or neither is
```
Relay delivers **at least once** → consumers must be idempotent (dedupe by event id).

> **Project:** "Transactional outbox + **Debezium CDC reading the WAL** → Kafka. During a **60 s Kafka outage: 0 lost events**, and game ticks were unaffected."

---

## 11. Recovery

### 11.1 Failure types

| Failure | Example | Handled by |
|---|---|---|
| Transaction failure | Constraint error, deadlock victim | Rollback (undo) |
| System crash | Power cut, OS crash — RAM lost, disk OK | WAL replay (redo + undo) |
| Media/disk failure | Disk dies | Backups + WAL archive (PITR), replicas |

### 11.2 Write-Ahead Logging (WAL) ⭐

**Simple explanation:** Before changing data pages on disk, first write **what you are going to change** into a sequential **log** and flush it. Like a shopkeeper writing every sale in the ledger before touching the cash box — after a blackout, the ledger tells exactly what happened.

**WAL rules:**
1. A log record must reach disk **before** the changed data page is written (write-**ahead**).
2. All log records of a transaction (incl. COMMIT record) must be on disk **before** commit returns success → durability.

Why it's fast: log writes are **sequential appends**; data pages can be written lazily later.

Each log record: `<LSN, txn_id, page, before-image (undo), after-image (redo)>`. **LSN** = log sequence number (position in the log); each page stores the LSN of the last change applied to it.

Postgres uses WAL for: crash recovery, **streaming replication** (replicas replay WAL), **point-in-time recovery**, and **logical decoding / CDC** (Debezium).

### 11.3 Undo and Redo; buffer policies

- **Undo:** reverse changes of **uncommitted** txns (use before-image). Needed if dirty pages of uncommitted txns may hit disk (**STEAL** policy).
- **Redo:** re-apply changes of **committed** txns (use after-image). Needed if committed data may not be on disk yet (**NO-FORCE** policy).

| Policy | Meaning | Need |
|---|---|---|
| Steal / No-steal | Can a dirty page of an uncommitted txn be flushed? | Steal → need UNDO |
| Force / No-force | Must all pages be flushed at commit? | No-force → need REDO |

Real DBs use **Steal + No-force** (best performance) → need both UNDO and REDO. (Postgres is a special case: thanks to MVCC, old versions stay in the table, so it mostly needs only REDO; uncommitted versions are simply invisible.)

Older techniques to name: **deferred update** (no undo, only redo), **immediate update** (undo + redo), **shadow paging** (copy-on-write page table, no log; used in some embedded DBs; poor for concurrency).

### 11.4 Checkpoints

**Simple explanation:** Without checkpoints, recovery would replay the log from the beginning of time. A **checkpoint** flushes all dirty pages to disk and writes a checkpoint record. Recovery starts from the last checkpoint.

- Postgres: `checkpoint_timeout` (default 5 min), `max_wal_size`. Too frequent → I/O spikes; too rare → long recovery.
- **Fuzzy checkpoint:** doesn't stop the world; records which txns/dirty pages were active.

### 11.5 ARIES (high level)

**ARIES** = Algorithm for Recovery and Isolation Exploiting Semantics (IBM). Used (in spirit) by most DBs. Three principles: **WAL**, **repeat history during redo**, **log changes during undo** (Compensation Log Records, CLRs, so undo is never undone twice).

Three passes after a crash:
1. **Analysis:** scan forward from the last checkpoint → find which txns were active (loser txns) and which pages were dirty (Dirty Page Table).
2. **Redo:** replay **all** logged changes (even of losers) from the oldest needed LSN — "repeat history" to restore the exact state at crash. Skip if page LSN ≥ record LSN.
3. **Undo:** roll back loser txns in reverse order, writing CLRs.

**Interview tip:** One-liner: "ARIES = analysis, redo everything to repeat history, then undo losers, with WAL and CLRs, supporting steal/no-force."

---

## 12. Storage and Indexing

### 12.1 Pages, heap files, and why disk access matters

**Simple explanation:** The DB doesn't read single rows from disk; it reads fixed-size **pages** (blocks). PostgreSQL page = **8 KB**. A table file is a heap of pages; each page holds many rows. Cost of a query ≈ **number of pages read**. RAM is ~1000× faster than disk, so the DB keeps hot pages in a **buffer pool** (`shared_buffers` in Postgres).

```
Page (8 KB): [header][item pointers → → →] ...free space... [← ← ← tuples]
Row address in PG = ctid = (page_number, item_number), e.g. (0,3)
```

- **Heap file:** rows in no particular order (Postgres tables).
- **Sorted file / clustered:** rows ordered by a key (MySQL InnoDB stores rows **inside** the primary-key B+ tree).
- Large values (> ~2 KB) are moved to a side table (**TOAST** in Postgres).
- Without an index, finding a row = **full table scan** (read every page): O(n).

### 12.2 B-Tree vs B+ Tree ⭐⭐

**Simple explanation:** An index is like the index at the back of a book: sorted keys with page numbers, so you jump straight to the right page instead of reading the whole book. Databases use a **B+ tree**: a short, very wide, always-balanced tree. With ~hundreds of keys per node, a tree of height 3–4 covers **millions to billions** of rows → 3–4 page reads per lookup.

**B-Tree:** keys **and data pointers in every node** (internal + leaf).
**B+ Tree:** internal nodes hold **only keys** (signposts); **all data pointers are in leaves**; **leaves are linked** in a list.

```
B+ Tree (order small for drawing), keys = emp_id

                    [ 30 | 60 ]                   <- root: only keys (signposts)
                  /      |      \
        [10 | 20]    [40 | 50]    [70 | 80]       <- internal: only keys
        /  |  \       /  |  \       /  |  \
  leaves:
  [5,8]->[10,15]->[20,25]->[30,35]->[40,45]->[50,55]->[60,65]->[70,75]->[80,90]
     ^ each leaf entry = key + pointer to row (ctid); leaves linked for range scans
```

Range query `WHERE emp_id BETWEEN 15 AND 45`: go down once to the leaf containing 15, then **walk the linked leaves** until 45. In a plain B-tree you'd have to go up and down the tree.

| | B-Tree | B+ Tree |
|---|---|---|
| Data pointers | In all nodes | **Only in leaves** |
| Internal nodes | Keys + data | Keys only → **more keys per node → shorter tree** |
| Leaves linked? | No | **Yes** → fast range scans and ORDER BY |
| Search cost | Can stop early at internal node | Always goes to leaf (predictable) |
| Duplicated keys | No | Internal keys repeat in leaves |
| Used in DBs | Rare | **Almost all** (Postgres "btree" is actually a B+ tree variant, MySQL InnoDB) |

Properties: balanced (all leaves same depth), O(log n) search/insert/delete, nodes ≥ half full, splits/merges keep balance.

**Why not a binary search tree / AVL?** Height log₂(n) ≈ 30 for 1 billion rows → 30 disk reads. B+ tree with fanout ~500: log₅₀₀(10⁹) ≈ 3–4. **Disk-friendly = fewer, wider nodes = one node per page.**

**Why not a hash table?** No range queries, no ordering.

### 12.3 Hash index

**Simple explanation:** `hash(key) → bucket → row pointer`. O(1) for **equality only** (`=`). Useless for `<`, `>`, `BETWEEN`, `ORDER BY`, prefix `LIKE`.
```sql
CREATE INDEX idx_customers_email_hash ON customers USING HASH (email);
```
In practice B-tree is almost always chosen in Postgres; hash indexes matter more in in-memory stores (Redis is a giant hash table).

### 12.4 Clustered vs non-clustered (primary vs secondary) ⭐

| | Clustered | Non-clustered (secondary) |
|---|---|---|
| What | Table rows **physically stored in key order** (the index *is* the table) | Separate structure with key → pointer to row |
| How many | **One** per table | Many |
| Range scan | Very fast (contiguous rows) | Each match may be a random page read |
| Example | MySQL InnoDB primary key; SQL Server clustered index | Every Postgres index; InnoDB secondary indexes (store PK, then look up PK tree) |

- **PostgreSQL tables are heaps → all indexes are secondary.** `CLUSTER table USING idx` reorders once, but it's not maintained.
- Other terms: **Primary index** (on ordering key field), **dense index** (entry for every row) vs **sparse index** (entry per page/block — only possible on sorted files), **multilevel index** (index on index = B+ tree idea).

**Trap:** In InnoDB, a random UUID primary key causes page splits all over the clustered index → slow inserts. Use auto-increment or time-ordered IDs (UUIDv7/ULID).

### 12.5 Composite index and the leftmost-prefix rule ⭐

**Simple explanation:** An index on `(a, b, c)` is sorted like a phone book by (last name, first name, city). You can quickly find by last name, or last + first — but **not** by first name alone.

Index `idx (customer_id, order_date)`:

| Query | Uses index? |
|---|---|
| `WHERE customer_id = 1` | ✓ |
| `WHERE customer_id = 1 AND order_date > '2026-01-01'` | ✓ (best) |
| `WHERE customer_id = 1 ORDER BY order_date DESC LIMIT 20` | ✓ no sort needed |
| `WHERE order_date > '2026-01-01'` | ✗ (usually; leftmost column missing) |
| `WHERE customer_id > 1 AND order_date = '...'` | Partly — range on first column stops use of second for seeking |

**Column order rule:** equality columns first, then range/sort column. More selective first is a secondary consideration.

### 12.6 Covering index and index-only scan

**Simple explanation:** If the index has **all columns the query needs**, the DB answers from the index alone and never visits the table (an **index-only scan**).
```sql
CREATE INDEX idx_orders_cust_cover ON orders (customer_id) INCLUDE (total_amount);
SELECT customer_id, SUM(total_amount) FROM orders WHERE customer_id = 1 GROUP BY 1;
-- Index Only Scan using idx_orders_cust_cover
```
Postgres needs the **visibility map** to be fresh (VACUUM) for true index-only scans (`Heap Fetches: 0`).

### 12.7 Index selectivity and when indexes hurt ⭐

**Selectivity** = distinct values / total rows (closer to 1 = more selective = better for an index). `email` ≈ 1.0 (great). `status` with 4 values in 10M rows ≈ 0.0000004 (bad — unless querying a rare value; then use a **partial index**).

**When an index is NOT used / hurts:**
1. **Low selectivity** — query returns a big fraction (e.g., >5–10%) of the table → seq scan is cheaper than many random reads.
2. **Function on the column:** `WHERE lower(email) = 'x'` → need an **expression index** on `lower(email)`.
3. **Type mismatch / implicit cast:** comparing `varchar` column to a number.
4. **Leading wildcard:** `LIKE '%arena'` (B-tree can't; use `pg_trgm` GIN index). `LIKE 'arena%'` is fine.
5. **Wrong column order** in a composite index (leftmost-prefix).
6. **`OR` across different columns** (may need two indexes + BitmapOr).
7. **Tiny tables** — a seq scan of 1 page beats any index.
8. **Stale statistics** → bad estimates; run `ANALYZE`.

**Costs of indexes:**
- Every INSERT/UPDATE/DELETE must update **every** index (write amplification). In Postgres, updating an indexed column prevents **HOT updates**.
- Extra disk and RAM (indexes compete for buffer cache).
- Unused indexes slow writes for nothing → find with `pg_stat_user_indexes` (`idx_scan = 0`).

**Interview tip:** "Index columns used in WHERE, JOIN and ORDER BY of frequent queries; check with EXPLAIN ANALYZE; don't index everything."

### 12.8 Other index types (one line each)

| Type | Use |
|---|---|
| GIN | JSONB `@>`, arrays, full-text search (`tsvector`) |
| GiST / SP-GiST | Geometric/geo (PostGIS), ranges, nearest-neighbour |
| BRIN | Huge append-only tables ordered by time (tiny index, stores min/max per block range) |
| Bitmap (Oracle) / bitmap scan (PG) | Combining several indexes with AND/OR |
| LSM tree (not PG) | Write-optimised storage in Cassandra, RocksDB: writes go to memtable → sorted SSTables, merged by compaction |

**B+ tree vs LSM tree:** B+ tree = fast reads, in-place updates (Postgres, MySQL). LSM = fast writes (sequential), reads may check several files (Cassandra, RocksDB, LevelDB).

---

## 13. Query Processing and Optimization

### 13.1 Steps of query processing

```
SQL text → Parser (syntax check → parse tree)
        → Rewriter / Analyzer (resolve names, expand views)
        → Optimizer / Planner (choose cheapest plan using statistics)
        → Executor (run the plan, return rows)
```

**Optimizer** = cost-based: estimates rows (cardinality) using table statistics (`pg_stats`: histograms, most common values, n_distinct), then picks scan types, join order and join algorithms with the lowest estimated cost (I/O + CPU).

**Heuristic rules it applies:** push selections (`WHERE`) down as early as possible, push projections down, choose join order to keep intermediate results small, replace subqueries with joins when possible.

### 13.2 Scan types and join algorithms ⭐

**Scans:**

| Scan | When |
|---|---|
| Seq Scan | Read whole table; good for large fractions or small tables |
| Index Scan | Follow index to each matching row; few rows |
| Index Only Scan | All needed columns are in the index |
| Bitmap Heap Scan | Medium number of rows: build a bitmap of pages from index, read pages in order |

**Join algorithms** (N = outer rows, M = inner rows):

| Algorithm | How | Cost | Best when |
|---|---|---|---|
| **Nested Loop** | For each outer row, scan inner for matches | O(N × M); O(N log M) with **index on inner** | Outer is small and inner has an index; non-equi joins |
| **Hash Join** | Build hash table on the **smaller** input, probe with the bigger | O(N + M), needs memory (`work_mem`) | Large unsorted inputs, **equality** join |
| **Merge Join** (sort-merge) | Sort both on join key, walk them together like merging in merge sort | O(N log N + M log M), O(N + M) if already sorted (index) | Both inputs sorted / big, equality or range |

```
Nested loop (C++ thinking):     Hash join:                       Merge join:
for (r : R)                     unordered_map h; for (s:S) h[s.k]   sort R, sort S; two pointers
  for (s : S)                   for (r : R) probe h[r.k]           i over R, j over S
    if (r.k == s.k) emit;
```

### 13.3 EXPLAIN ANALYZE — before and after an index ⭐

Scenario: `orders` has ~1,000,000 rows. Query = "latest 20 orders of a customer" (the match-history shape). Output below is **representative** (numbers depend on hardware).

**Before (no index on customer_id):**
```sql
EXPLAIN ANALYZE
SELECT order_id, order_date, total_amount
FROM orders WHERE customer_id = 42
ORDER BY order_date DESC LIMIT 20;
```
```
Limit  (cost=21433.12..21433.17 rows=20 width=18) (actual time=118.402..118.406 rows=20 loops=1)
  ->  Sort  (cost=21433.12..21433.37 rows=100 width=18) (actual time=118.400..118.402 rows=20 loops=1)
        Sort Key: order_date DESC
        Sort Method: top-N heapsort  Memory: 26kB
        ->  Seq Scan on orders  (cost=0.00..21430.00 rows=100 width=18) (actual time=0.031..118.300 rows=98 loops=1)
              Filter: (customer_id = 42)
              Rows Removed by Filter: 999902
Planning Time: 0.110 ms
Execution Time: 118.440 ms
```

**Add the right composite index:**
```sql
CREATE INDEX CONCURRENTLY idx_orders_cust_date ON orders (customer_id, order_date DESC);
ANALYZE orders;
```

**After:**
```
Limit  (cost=0.42..80.10 rows=20 width=18) (actual time=0.025..0.060 rows=20 loops=1)
  ->  Index Scan using idx_orders_cust_date on orders  (cost=0.42..398.80 rows=100 width=18) (actual time=0.024..0.056 rows=20 loops=1)
        Index Cond: (customer_id = 42)
Planning Time: 0.150 ms
Execution Time: 0.080 ms
```

**How to read it:**
1. Read **bottom-up / innermost first**: the most indented node runs first and feeds its parent.
2. `cost=startup..total` — planner's **estimate** in arbitrary units (not ms). `rows=` estimated rows, `width=` bytes per row.
3. `actual time=first_row..last_row ms`, `rows=` real rows, `loops=` how many times the node ran (multiply time × loops for nested loops).
4. **Compare estimated vs actual rows.** Big mismatch (est 10, actual 100,000) → stale stats → `ANALYZE`, or correlated columns → extended statistics.
5. Red flags: `Seq Scan` with huge `Rows Removed by Filter`; `Sort Method: external merge Disk` (increase `work_mem` or add an index); Nested Loop with large `loops`; `Heap Fetches` high on index-only scans.
6. Here: seq scan read 1M rows to keep 98 → index lets the DB jump to customer 42 and read **already sorted** entries, so the `Sort` node disappeared and `LIMIT` stops after 20 rows. ~118 ms → ~0.08 ms.

> **Project:** "LocalArena's match-history query: indexed and **limit-first** — **0.8 ms**."

### 13.4 Practical optimization checklist

- Select only needed columns (no `SELECT *` in hot paths) → enables index-only scans, less I/O.
- Index WHERE/JOIN/ORDER BY columns; composite with equality first.
- Avoid functions on indexed columns in WHERE; use half-open date ranges.
- **Keyset pagination** instead of large OFFSET: `WHERE (order_date, order_id) < ($last_date, $last_id) ORDER BY order_date DESC, order_id DESC LIMIT 20`. `OFFSET 100000` still reads and throws away 100,000 rows.
- Use `EXISTS` instead of `COUNT(*) > 0`; `UNION ALL` instead of `UNION`.
- Batch inserts (`COPY`, multi-row `INSERT`), avoid N+1 (Section 16).
- Keep statistics fresh (`ANALYZE`), autovacuum healthy.
- Watch slow queries with `pg_stat_statements` and `log_min_duration_statement`.

---

## 14. Scaling Databases

### 14.1 Vertical vs horizontal scaling

- **Vertical (scale up):** bigger machine (more CPU/RAM/faster disk). Simple, no code change, has a ceiling and a single point of failure.
- **Horizontal (scale out):** more machines — **replication** (copies, for reads + availability) and **sharding** (split data, for writes + size).

Order to try in interviews: **index & query fix → cache → read replicas → vertical scale → partitioning → sharding** (sharding is the last resort because it's the most complex).

### 14.2 Replication: sync vs async ⭐

**Simple explanation:** The **primary** (leader) takes writes and streams its **WAL** to **replicas** (followers), which replay it. Like a teacher writing on the board and students copying.

| | Synchronous | Asynchronous |
|---|---|---|
| Commit returns when | Replica has also received/flushed the WAL | Primary alone has flushed |
| Data loss on primary failure | **None (RPO = 0)** | Recent writes may be lost |
| Write latency | Higher (network round trip) | Low |
| Availability risk | If replica is down, writes can stall | Primary keeps going |

Middle ground: **semi-sync / quorum** (`synchronous_standby_names = 'ANY 1 (r1, r2)'`).

Other forms: **multi-leader** (writes in many regions, needs conflict resolution), **leaderless** (Dynamo/Cassandra; quorum reads/writes: `R + W > N` for strong-ish consistency).

**RPO** (Recovery Point Objective) = how much data you can lose. **RTO** (Recovery Time Objective) = how long until service is back.

> **Project:** "Async replica failover: **RTO 0.7 s**; **RPO 0 when healthy**, but **2.8 s of writes lost after a 5 s partition** — exactly the async trade-off. If losing writes is unacceptable (payments), use sync or quorum replication."

### 14.3 Read replicas and replication lag; read-your-writes ⭐

- Send **reads to replicas**, **writes to primary** → scales read-heavy apps.
- **Replication lag**: replica may be behind by ms–seconds → **stale reads**.
- **Read-your-writes consistency:** a user who just updated their profile must see the update. Techniques:
  - Read from the **primary for a short window** after a user's write (or for that user's own data).
  - Track the write's **LSN** and only read from a replica that has replayed past it (`pg_last_wal_replay_lsn()`).
  - Sticky sessions to primary for writers.
- Other guarantees: **monotonic reads** (don't go back in time — pin a user to one replica).

> **Project:** "Reads right after a write must go to the primary — otherwise a player can buy something and not see it."

### 14.4 Partitioning vs sharding ⭐

**Simple explanation:** **Partitioning** splits one big table into smaller pieces **inside one DB server**. **Sharding** puts the pieces on **different servers**.

PostgreSQL declarative partitioning:
```sql
CREATE TABLE events (
    id BIGINT, player_id BIGINT, created_at TIMESTAMPTZ NOT NULL, payload JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2026_01 PARTITION OF events
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE events_2026_02 PARTITION OF events
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
-- Query with created_at filter touches only matching partitions (partition pruning).
-- Drop old data instantly: DROP TABLE events_2026_01;  (vs a slow huge DELETE)
```
Types: `RANGE` (dates), `LIST` (region IN ('IN','US')), `HASH` (spread evenly).

**Horizontal vs vertical partitioning:** horizontal = split **rows** (by date/user); vertical = split **columns** (move big rarely used `bio`, `avatar` to another table).

**Sharding strategies:**

| Strategy | How | Pros | Cons |
|---|---|---|---|
| Range | user_id 1–1M → shard 1, ... | Range queries easy | **Hot spots** (newest users all on last shard) |
| Hash (modulo) | `hash(key) % N` | Even spread | Changing N moves **almost all** keys |
| **Consistent hashing** | Ring / jump hash | Adding a shard moves only ~1/N keys | Slightly complex |
| Directory / lookup | Table maps key → shard | Flexible, can move tenants | Lookup service is a dependency |
| Geo | By region/country | Data locality, compliance | Uneven load |

**Shard key choice:** high cardinality, even distribution, and matches the main access pattern (most queries include it). For a chat/game app: `player_id` or `match_id`.

**Sharding problems:** cross-shard joins (avoid or denormalize), cross-shard transactions (2PC or sagas), **hot keys** (celebrity user), **resharding** (moving data online), global unique IDs (Snowflake IDs, UUIDv7), fan-out queries.

### 14.5 Consistent hashing ⭐

**Simple explanation:** With `hash % N`, going from 3 to 4 servers changes the result for ~75% of keys → cache stampede / mass data movement. **Consistent hashing** places servers and keys on a **ring**; a key belongs to the next server clockwise. Adding a server only takes keys from its neighbour → only ~**1/N** of keys move.

```
          S1
       /      \
   k3            k1  → S2 (next clockwise)
   |              |
  S3              S2
       \      /
          k2  → S3
```
- **Virtual nodes:** each server appears at many points on the ring → even load, smooth rebalancing.
- **Jump Consistent Hash** (Google): no ring, tiny loop, perfectly even, minimal movement; best when shards are numbered 0..N-1 and only added at the end.
- Used in: DynamoDB, Cassandra, Memcached clients, load balancers, Redis Cluster uses 16,384 **hash slots** (similar idea).

> **Project:** "I used **Jump Consistent Hash** — going from **2 → 3 shards moved only ~33% of keys** (the theoretical minimum is 1/3)."

### 14.6 Connection pooling ⭐ (+ parameterized queries)

**Simple explanation:** Opening a DB connection is expensive (TCP + TLS + auth; in **Postgres each connection is a separate OS process** using several MB). A pool keeps a few connections open and lends them out — like a taxi stand instead of buying a car per trip.

- **In-app pool:** HikariCP (Java), `pgxpool` (Go), `psycopg_pool` / SQLAlchemy (Python), Prisma pool (TS).
- **External pooler:** **PgBouncer** — many app instances share a small number of server connections.

PgBouncer modes:

| Mode | Server connection is returned to pool when | Note |
|---|---|---|
| Session | Client disconnects | Safest, least sharing |
| **Transaction** | Transaction ends | Most common; session state (SET, temp tables, advisory locks) breaks |
| Statement | Each statement ends | No multi-statement transactions |

Pool size rule of thumb: small! Roughly `cores × 2 (+ disks)` per DB server, not 1000. Too many connections → context switching and lock contention → **slower**.

> **Project (story H5):** "PgBouncer **pool was paused** → about half of requests failed. I diagnosed it with **`SHOW POOLS`** on the PgBouncer admin console." Also: "In a DB brownout test I saw a **retry storm of 7,482 retries** before the fix (my timeout-ordering story) — pools + timeouts + bounded retries must be designed together."

**Go: pool + parameterized queries (SQL injection prevention)**
```go
package main

import (
    "context"
    "log"
    "os"
    "time"

    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    ctx := context.Background()
    cfg, err := pgxpool.ParseConfig(os.Getenv("DATABASE_URL")) // never hard-code credentials
    if err != nil { log.Fatal(err) }
    cfg.MaxConns = 10                       // small, bounded pool
    cfg.MaxConnIdleTime = 5 * time.Minute

    pool, err := pgxpool.NewWithConfig(ctx, cfg)
    if err != nil { log.Fatal(err) }
    defer pool.Close()

    email := os.Args[1] // untrusted input, e.g. "x' OR '1'='1"

    // SAFE: $1 placeholder. The value is sent separately from the SQL text,
    // so it can never change the query's structure.
    var id int
    var name string
    qctx, cancel := context.WithTimeout(ctx, 2*time.Second) // always bound DB calls
    defer cancel()
    err = pool.QueryRow(qctx,
        "SELECT customer_id, name FROM customers WHERE email = $1", email).Scan(&id, &name)
    if err != nil { log.Println("not found or error:", err); return }
    log.Println(id, name)

    // UNSAFE (never do this):
    // q := "SELECT ... WHERE email = '" + email + "'"   -> becomes WHERE email = 'x' OR '1'='1'
}
```

**Python equivalent (psycopg 3):**
```python
from psycopg_pool import ConnectionPool
import os

pool = ConnectionPool(os.environ["DATABASE_URL"], min_size=1, max_size=10)

def find_customer(email: str):
    with pool.connection() as conn:                          # borrow + auto-return
        return conn.execute(
            "SELECT customer_id, name FROM customers WHERE email = %s", (email,)  # parameter, not f-string
        ).fetchone()
```

**SQL injection key points:** use parameterized queries / prepared statements everywhere; placeholders can't be used for identifiers (table/column names) → use an **allowlist**; least-privilege DB user; ORMs are safe only when you don't build raw strings.

### 14.7 Caching patterns ⭐

**Simple explanation:** Keep hot data in RAM (Redis) so most reads don't hit the DB.

| Pattern | Read path | Write path | Pros | Cons |
|---|---|---|---|---|
| **Cache-aside (lazy loading)** | App checks cache → miss → read DB → put in cache with TTL | App writes DB, then **deletes** cache key | Simple, cache only holds hot data | First read is slow; stale window |
| **Read-through** | Cache library loads from DB on miss | — | App code simpler | Needs cache provider support |
| **Write-through** | — | Write cache **and** DB synchronously | Cache always fresh | Slower writes; caches cold data |
| **Write-back (write-behind)** | — | Write cache, flush to DB **later** async | Very fast writes, batching | **Data loss** if cache dies before flush |
| Write-around | — | Write DB only; cache fills on read | No cache churn for write-once data | Read after write is a miss |

**Problems to name:**
- **Stale data / invalidation** — prefer **delete on write** over update-on-write (avoids race ordering bugs); always set a TTL as a safety net.
- **Cache stampede / thundering herd** — many misses at once for a hot key: use a lock/single-flight, early refresh, TTL jitter.
- **Cache penetration** — queries for keys that don't exist: cache the "null" result briefly or use a Bloom filter.
- **Cache avalanche** — many keys expire together: random TTL jitter.

### 14.8 CAP and PACELC ⭐

**CAP:** In a distributed system, when a **network Partition** happens, you must choose **Consistency** (every read sees the latest write, or errors) or **Availability** (every request gets a response, maybe stale). P is not optional in real networks, so the real choice is **C vs A during a partition**.

| Choice | Behaviour during partition | Examples |
|---|---|---|
| CP | Refuse/timeout some requests to stay correct | PostgreSQL primary + sync replica, HBase, ZooKeeper, etcd, MongoDB (default majority) |
| AP | Keep answering, reconcile later | Cassandra, DynamoDB (eventual reads), CouchDB, DNS |

**PACELC:** **if P**artition → choose **A or C**; **E**lse (normal operation) → choose **L**atency or **C**onsistency. It adds the everyday trade-off: even without failures, strong consistency costs latency (waiting for replicas / other regions).
- Cassandra/Dynamo: **PA/EL**. Traditional RDBMS with sync replication: **PC/EC**. MongoDB: PC/EC-ish by default.

> **Project:** "Multi-region week: a write to a remote region took **805 ms vs 35 ms** locally — that's the **L vs C** part of PACELC in numbers. Async replication: low latency but lost **2.8 s** of writes after a partition — the C side of the trade."

**Trap:** CAP "consistency" ≠ ACID "consistency". CAP-C = linearizability (latest value); ACID-C = constraints/invariants hold.

### 14.9 ACID vs BASE

| ACID (SQL) | BASE (many NoSQL) |
|---|---|
| Atomic, Consistent, Isolated, Durable | **B**asically **A**vailable, **S**oft state, **E**ventually consistent |
| Strong consistency, transactions | Availability and scale first; replicas converge "eventually" |
| Banking, orders, inventory | Feeds, likes, analytics, carts, sessions |

Eventual consistency variants: read-your-writes, monotonic reads, causal consistency. Many NoSQL DBs now offer ACID per document/partition (MongoDB multi-document transactions, DynamoDB transactions).

---

## 15. NoSQL and Redis

### 15.1 Why NoSQL?

**Simple explanation:** "Not only SQL". Gives up some relational features (joins, fixed schema, multi-row transactions) to get **flexible schema, horizontal scale, very high throughput, or a special data model**.

| Type | Data shape | Examples | Good for | Not good for |
|---|---|---|---|---|
| **Key-value** | `key → value` (blob/structure) | **Redis**, DynamoDB, Memcached, etcd | Cache, sessions, counters, rate limits, leaderboards | Complex queries |
| **Document** | JSON/BSON documents, nested | **MongoDB**, Couchbase, Firestore | Product catalogs, user profiles, CMS — data read as one object | Many-to-many joins, strict multi-entity transactions |
| **Wide-column** | Rows with flexible columns, grouped by **partition key**, sorted by **clustering key** | **Cassandra**, HBase, ScyllaDB, Bigtable | Huge **write-heavy** data: time-series, IoT, messages, activity logs | Ad-hoc queries, joins; you design tables **per query** |
| **Graph** | Nodes + edges with properties | **Neo4j**, Amazon Neptune | Social networks, recommendations, fraud rings, "friends of friends" | Bulk aggregation over everything |
| Search | Inverted index | Elasticsearch/OpenSearch | Full-text, fuzzy search, logs | Primary source of truth |
| Time-series | Time-indexed points | InfluxDB, TimescaleDB, Prometheus | Metrics | General OLTP |

Tiny examples:
```js
// MongoDB document
db.orders.insertOne({ _id: 101, customer: { id: 1, name: "Rohan" },
                      items: [{ product: "Laptop", qty: 1, price: 60000 }], status: "DELIVERED" })
db.orders.find({ "customer.id": 1, status: "DELIVERED" })
```
```sql
-- Cassandra (CQL): table designed for "latest messages of a chat room"
CREATE TABLE messages_by_room (
  room_id uuid, sent_at timeuuid, sender text, body text,
  PRIMARY KEY ((room_id), sent_at)            -- partition key, clustering key
) WITH CLUSTERING ORDER BY (sent_at DESC);
```
```
// Neo4j (Cypher): friends of friends of Rohan
MATCH (:User {name:'Rohan'})-[:FRIEND]->()-[:FRIEND]->(fof) RETURN DISTINCT fof.name
```

**SQL vs NoSQL — how to choose ⭐**

| Need | Pick |
|---|---|
| Transactions, joins, money, inventory, strong integrity | PostgreSQL / MySQL |
| Cache, TTL, counters, leaderboards, presence, rate limits | Redis |
| Flexible nested documents, evolving schema | MongoDB |
| Massive write-heavy, wide rows, time-series, multi-DC | Cassandra |
| Relationship-heavy traversals | Neo4j |
| Full-text search | Elasticsearch / OpenSearch |
| Event log / replay | Kafka (a durable log, not a DB) |

**Interview tip:** Don't say "NoSQL scales, SQL doesn't." Say: "Start with Postgres for correctness; add Redis for hot reads/ephemeral data; pick a specialised store when a specific access pattern needs it."

### 15.2 Redis essentials ⭐

**Simple explanation:** Redis = in-memory key-value store with rich data structures. Commands execute on a **single thread** (one at a time) → each command is **atomic**, sub-millisecond latency.

| Type | Like (C++) | Commands | Use case |
|---|---|---|---|
| String | `std::string` / int | `SET, GET, INCR, SETEX, SET NX` | Cache, counters, locks |
| Hash | `unordered_map<string,string>` | `HSET, HGET, HGETALL, HINCRBY` | Object fields (user profile, room metadata) |
| List | `deque` | `LPUSH, RPUSH, LPOP, LRANGE, LTRIM` | Recent items, simple queues |
| Set | `unordered_set` | `SADD, SREM, SMEMBERS, SISMEMBER, SINTER` | Unique members, tags, online users |
| Sorted Set (ZSET) | `map<score, member>` (skip list + hash) | `ZADD, ZINCRBY, ZREVRANGE, ZRANK` | **Leaderboards**, priority, time-ordered sets |
| Others | — | Bitmaps, HyperLogLog (`PFADD`/`PFCOUNT`), Geo, **Streams** (`XADD`), Pub/Sub | DAU flags, unique counts, nearby, event logs |

> **Project (TTL Chat):** "Rooms use Redis **HASH** (room metadata), **LIST** (messages) and **SET** (members) — all **O(1)** operations — with **EXPIRE** so a room disappears automatically when its TTL ends."

**Persistence:**

| | RDB (snapshot) | AOF (append-only file) |
|---|---|---|
| How | Fork + dump whole dataset periodically | Log every write command |
| Data loss | Since last snapshot (minutes) | ~1 s with `appendfsync everysec` (or none with `always`, slow) |
| File / restart | Compact, fast restart | Bigger, slower replay (rewritten/compacted in background) |
| Use | Backups | Durability |

Common: **both** (RDB for backups + AOF for durability). Redis is still usually a **cache, not the source of truth**.

**Eviction** (when `maxmemory` is reached):

| Policy | Evicts |
|---|---|
| `noeviction` (default) | Nothing — writes return errors |
| `allkeys-lru` | Least recently used, any key (**typical for cache**) |
| `volatile-lru` | LRU among keys **with a TTL** |
| `allkeys-lfu` / `volatile-lfu` | Least frequently used |
| `volatile-ttl` | Keys closest to expiry |
| `allkeys-random` / `volatile-random` | Random |

Expiry: lazy (checked on access) + active (background sampling).

**HA/Scale:** replication (primary → replicas, async), **Sentinel** (automatic failover), **Redis Cluster** (16,384 hash slots across shards).

### 15.3 Redis code: cache-aside, leaderboard, rate limiter

**Cache-aside with TTL**
```
# Read path
GET product:1                                 # (nil)  -> cache miss
# app runs: SELECT * FROM products WHERE product_id = 1;
SET product:1 '{"id":1,"name":"Laptop","price":60000}' EX 300    # cache 5 min
GET product:1                                 # hit next time
TTL product:1                                 # seconds left, e.g. 297

# Write path (update DB first, then invalidate)
# app runs: UPDATE products SET price = 58000 WHERE product_id = 1;
DEL product:1                                 # next read reloads fresh data

# Stampede guard: only one worker rebuilds
SET lock:product:1 worker-7 NX EX 10          # OK = you rebuild; (nil) = someone else is
```

**Sorted-set leaderboard**
```
ZADD leaderboard 1500 "player:42" 1320 "player:7" 1710 "player:99"
ZINCRBY leaderboard 25 "player:7"             # player 7 wins a match -> 1345
ZREVRANGE leaderboard 0 9 WITHSCORES          # top 10, highest first
ZREVRANK leaderboard "player:42"              # 0-based rank -> 1 (2nd place)
ZSCORE leaderboard "player:42"                # 1500
ZCOUNT leaderboard 1400 +inf                  # players with score >= 1400
```
All O(log N) — far better than `ORDER BY score DESC LIMIT 10` on a busy SQL table at every request.

**Fixed-window rate limiter (INCR + EXPIRE): 100 requests / minute per user**
```
# key = rate:{user}:{current minute}
INCR rate:user42:202611251030                 # -> 1 (first request this minute)
EXPIRE rate:user42:202611251030 60 NX         # set TTL only if none (Redis 7+)
# if value > 100 -> reject with HTTP 429
```
Make INCR+EXPIRE atomic with `MULTI ... EXEC` or a small **Lua script** (on older Redis, `EXPIRE` without `NX` only when INCR returned 1). Fixed windows allow bursts at the window edge — a **sliding window** uses a ZSET of timestamps (`ZADD`, `ZREMRANGEBYSCORE`, `ZCARD`) or a **token bucket**.

**Trap:** Rate limiting in each app server's memory doesn't work with many servers — that's why a shared Redis counter is used.

---

## 16. ORMs and the N+1 Problem

**Simple explanation:** An **ORM** (Object-Relational Mapper) maps tables to classes: Hibernate/JPA (Java), GORM (Go), SQLAlchemy/Django ORM (Python), Prisma/TypeORM (TS). You write `customer.orders` instead of SQL.

| Pros | Cons |
|---|---|
| Less boilerplate, type safety, migrations | Hides SQL → easy to write slow queries |
| Parameterized by default (injection-safe) | **N+1 queries**, over-fetching |
| DB portability | Complex queries (window functions, CTEs) are awkward → drop to raw SQL |

**N+1 problem ⭐:** load a list (1 query), then for each item lazily load its relation (N queries).

```python
# 1 query for orders + 1 query PER order for its customer = N+1
orders = session.query(Order).all()          # SELECT * FROM orders;           (1)
for o in orders:
    print(o.customer.name)                   # SELECT * FROM customers WHERE id = ? (N times)
```
With 1,000 orders → 1,001 round trips; at ~1 ms each that's a full second.

**Fixes:**
```sql
-- 1) JOIN (eager loading)
SELECT o.order_id, c.name FROM orders o JOIN customers c ON c.customer_id = o.customer_id;

-- 2) Batch loading with IN / ANY (2 queries total)
SELECT * FROM orders;
SELECT * FROM customers WHERE customer_id = ANY($1);   -- $1 = array of ids from step 1
```
ORM switches: `joinedload/selectinload` (SQLAlchemy), `select_related/prefetch_related` (Django), `JOIN FETCH` / `@EntityGraph` (JPA), `include` (Prisma), `Preload` (GORM). In GraphQL: **DataLoader** batching.

**How to detect:** enable SQL logging, count queries per request, look for the same query shape repeated N times (`pg_stat_statements` shows huge `calls`).

**Interview tip:** "I use an ORM for simple CRUD and raw parameterized SQL for hot or complex queries, and I always check generated SQL with EXPLAIN."

---

## 17. Most Asked DBMS Interview Questions (Top 40)

⭐ = top 15 most frequently asked in Indian product-company rounds. Answer in 2–5 lines, then give an example or a project link.

**1. ⭐ What is ACID?**
Atomicity: all or nothing (undo/WAL). Consistency: constraints and invariants hold before and after. Isolation: concurrent transactions behave as if run one by one (locks/MVCC). Durability: after COMMIT, data survives a crash (WAL fsync). Example: bank transfer; my outbox commits order + event atomically.

**2. ⭐ What is normalization? Explain 1NF, 2NF, 3NF, BCNF.**
Splitting tables to remove redundancy and insert/update/delete anomalies. 1NF: atomic values. 2NF: no partial dependency on part of a composite key. 3NF: no transitive dependency (non-key → non-key). BCNF: every determinant is a super key. Walk through the order-sheet example (Section 8.4).

**3. ⭐ Types of keys? Primary vs unique key?**
Super, candidate (minimal super), primary (chosen candidate, NOT NULL, one per table), alternate, composite, foreign, surrogate. Unique key allows NULLs and you can have many; PK can't be NULL and there's one. Mine: composite `(player_id, idempotency_key)`.

**4. ⭐ Explain all types of joins.**
INNER (matches only), LEFT/RIGHT (all of one side + matches, NULLs otherwise), FULL (all of both), CROSS (cartesian), SELF (table with itself, e.g., employee–manager). Mention the trap: a WHERE filter on the right table turns a LEFT JOIN into INNER.

**5. ⭐ Find the 2nd / Nth highest salary.**
`SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);` For Nth: `DENSE_RANK() OVER (ORDER BY salary DESC)` and filter `= N`, or `ORDER BY salary DESC LIMIT 1 OFFSET N-1` on DISTINCT salaries. DENSE_RANK handles ties.

**6. ⭐ What are isolation levels? What does each prevent?**
Read Uncommitted (dirty reads possible), Read Committed (no dirty reads; PG default), Repeatable Read (no non-repeatable reads; in PG also no phantoms), Serializable (no anomalies; PG uses SSI and aborts with 40001). Higher level = more safety, more blocking/retries.

**7. ⭐ Dirty read vs non-repeatable read vs phantom read?**
Dirty: reading uncommitted data that may be rolled back. Non-repeatable: same row read twice gives different values because another txn updated + committed. Phantom: same query returns new/missing rows because another txn inserted/deleted. Add lost update and write skew.

**8. ⭐ What is an index? How does a B+ tree work?**
A sorted data structure that finds rows without scanning the whole table. B+ tree: balanced, wide nodes (one page each), keys only in internal nodes, all row pointers in linked leaves → O(log n) lookups in 3–4 page reads, and fast range scans by walking leaves.

**9. ⭐ Clustered vs non-clustered index?**
Clustered: rows physically stored in index order; only one per table (InnoDB PK). Non-clustered: separate structure pointing to rows; many allowed. PostgreSQL tables are heaps, so all indexes are non-clustered.

**10. ⭐ WHERE vs HAVING? GROUP BY order of execution?**
WHERE filters rows before grouping and can't use aggregates; HAVING filters groups after aggregation. Execution: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT. That's why SELECT aliases aren't visible in WHERE.

**11. ⭐ DELETE vs TRUNCATE vs DROP?**
DELETE: DML, row by row, WHERE allowed, fires triggers, slow. TRUNCATE: DDL, removes all rows fast, no row triggers. DROP: removes table structure too. In PostgreSQL even TRUNCATE/DROP can be rolled back inside a transaction (not in MySQL).

**12. ⭐ What is a deadlock? How do you handle it?**
Two transactions each hold a lock the other needs → wait forever. DB detects cycles in a wait-for graph and aborts one (PG: error 40P01). Prevent with fixed lock order, short transactions, timeouts; retry victims with jitter. My story: FOR UPDATE vs FK's FOR KEY SHARE → fixed with FOR NO KEY UPDATE + lock order + retry.

**13. ⭐ Optimistic vs pessimistic locking?**
Pessimistic: lock first (`SELECT ... FOR UPDATE`) — good for high contention. Optimistic: no lock; `UPDATE ... WHERE version = $v`, retry if 0 rows — good for low contention. My numbers: optimistic needed 206 retries on hot stock, so I used conditional update `stock = stock-1 WHERE stock > 0` → 129 ms for 200 buyers.

**14. ⭐ SQL vs NoSQL — when to use which?**
SQL: structured data, relationships, joins, transactions, strong consistency (orders, payments). NoSQL: flexible schema, huge scale, specific access patterns (Redis cache/leaderboards, MongoDB documents, Cassandra write-heavy time-series, Neo4j graphs). Usually both: Postgres as source of truth + Redis for hot data.

**15. ⭐ What is the CAP theorem?**
During a network partition a distributed system must pick Consistency (latest data or error) or Availability (always answer, maybe stale). PACELC adds: else, Latency vs Consistency. Postgres primary is CP-leaning; Cassandra/Dynamo AP. My data: async failover lost 2.8 s of writes after a partition; remote write 805 ms vs local 35 ms.

**16. What is a view? View vs materialized view?**
View = saved query, no data stored, always fresh; used for security and simplicity. Materialized view stores the result, can be indexed, fast to read, but stale until `REFRESH MATERIALIZED VIEW`.

**17. What is a transaction? Explain its states.**
A logical unit of work that's all-or-nothing. States: Active → Partially committed → Committed, or Active → Failed → Aborted; both end in Terminated.

**18. What is a schedule? What is conflict serializability?**
A schedule is the interleaved order of operations of concurrent transactions. Conflict serializable = can be reordered (swapping non-conflicting ops) into a serial schedule. Test: precedence graph; no cycle → serializable.

**19. Conflict vs view serializability?**
Conflict uses only R-W/W-R/W-W ordering; view requires same initial reads, same reads-from, same final writes. Every conflict-serializable schedule is view-serializable, not vice versa (extras involve blind writes). View test is NP-complete, so DBs use conflict-based methods.

**20. What is two-phase locking? Strict 2PL?**
Growing phase (acquire only), then shrinking phase (release only) → guarantees conflict serializability. Strict 2PL holds exclusive locks until commit → no cascading rollbacks. 2PL doesn't prevent deadlocks (except conservative 2PL).

**21. What is MVCC?**
Multi-Version Concurrency Control: writers create new row versions; readers see a consistent snapshot, so reads don't block writes. PG stores xmin/xmax on each tuple; VACUUM removes dead versions. Writers on the same row still lock each other.

**22. What is a WAL? Why is it needed?**
Write-Ahead Log: changes are appended to a sequential log and flushed before data pages are written and before COMMIT returns. Gives durability and crash recovery (redo/undo), and is also used for replication, PITR and CDC (Debezium reads the WAL for my outbox).

**23. Explain checkpoints and ARIES.**
Checkpoint flushes dirty pages and records a point so recovery starts there instead of from the beginning. ARIES: Analysis (find losers and dirty pages) → Redo (repeat history) → Undo (roll back losers with CLRs).

**24. What is a foreign key and referential integrity? ON DELETE options?**
FK column must match a PK/unique value in the parent or be NULL. Options: RESTRICT/NO ACTION (block), CASCADE (delete children), SET NULL, SET DEFAULT.

**25. Functional dependency, closure, and how to find candidate keys?**
X → Y: same X implies same Y. X⁺ = all attributes derivable from X via FDs. To find keys: attributes never on any RHS must be in every key; compute closures of combinations grown from them; minimal sets reaching all attributes are candidate keys.

**26. What is a lossless decomposition? Dependency preservation?**
Lossless: joining the parts returns exactly the original (common attributes are a key of one part). Dependency preserving: every FD can be checked in a single part. 3NF always achieves both; BCNF may lose dependency preservation.

**27. What is denormalization? When do you use it?**
Adding controlled redundancy for faster reads (fewer joins): stored totals, counters, materialized views, read models. Used in read-heavy systems and analytics; costs extra writes and consistency work.

**28. Correlated vs non-correlated subquery? IN vs EXISTS?**
Non-correlated runs once independently; correlated references the outer row and logically runs per row. EXISTS stops at first match and is NULL-safe; NOT IN breaks if the subquery returns any NULL — use NOT EXISTS.

**29. RANK vs DENSE_RANK vs ROW_NUMBER?**
For salaries 95, 95, 90: ROW_NUMBER 1,2,3 (unique); RANK 1,1,3 (gap after tie); DENSE_RANK 1,1,2 (no gap). Use DENSE_RANK for "Nth highest distinct".

**30. UNION vs UNION ALL?**
UNION removes duplicates (sort/hash cost); UNION ALL keeps all rows and is faster. Both need the same number and compatible types of columns.

**31. When will an index not be used? Downsides of indexes?**
Low selectivity, functions on the column, leading `%` LIKE, implicit casts, wrong composite order (leftmost prefix), tiny tables, stale stats. Downsides: slower writes (every index updated), storage, memory.

**32. What is a composite / covering index? Leftmost-prefix rule?**
Composite: index on multiple columns, e.g., (customer_id, order_date) — serves filters on customer_id or customer_id+order_date, not order_date alone. Covering: contains all columns the query needs → index-only scan.

**33. How do you optimize a slow query?**
Run EXPLAIN ANALYZE; look for seq scans with many rows removed, estimate vs actual mismatch, sorts spilling to disk, large nested loops. Add the right index, rewrite (avoid functions on columns, SELECT only needed columns), keyset pagination, ANALYZE. My match-history query: indexed + limit-first = 0.8 ms.

**34. Explain join algorithms.**
Nested loop: for each outer row scan inner (great with inner index, small outer). Hash join: build hash on smaller input, probe with larger (big equality joins). Merge join: sort both and merge (inputs already sorted).

**35. Replication: sync vs async? What is replication lag?**
Sync: commit waits for replica → no data loss, higher latency. Async: faster, can lose recent writes on failover. Lag = replica behind primary → stale reads; fix read-your-writes by reading from primary after a write. My failover: RTO 0.7 s, RPO 0 healthy, 2.8 s lost after a 5 s partition.

**36. Sharding vs partitioning? How do you choose a shard key?**
Partitioning splits a table inside one server; sharding spreads data across servers. Shard key: high cardinality, even distribution, present in most queries. Problems: cross-shard joins/transactions, hot keys, resharding. Consistent hashing minimizes movement — my Jump Hash 2→3 shards moved ~33% of keys.

**37. Why connection pooling?**
Connections are expensive (Postgres: one process per connection); a pool reuses a small bounded set, reducing latency and protecting the DB. PgBouncer transaction mode is common. My incident: PgBouncer pool paused → half the requests failed; diagnosed with `SHOW POOLS`.

**38. Explain caching strategies. How do you keep cache consistent?**
Cache-aside (app reads cache, falls back to DB, writes DB then deletes key), write-through (write both synchronously), write-back (write cache, flush later — risk of loss). Use TTLs, delete-on-write, jitter, and single-flight to avoid stampedes.

**39. What is the N+1 query problem?**
Fetching N parent rows with one query, then one extra query per row for a relation → N+1 round trips (common with lazy-loading ORMs). Fix with JOIN/eager loading or batching with `IN`/`ANY` (2 queries total).

**40. How do you make an API operation idempotent at the DB level? What is the outbox pattern?**
Store an idempotency key with a unique constraint and `INSERT ... ON CONFLICT DO NOTHING`; if no row inserted, it's a duplicate → return saved response. Outbox: write the event row in the same transaction as the business change; a relay/CDC publishes it later. LocalArena: 1.45M replayed events → 0 duplicate rewards; 60 s Kafka outage → 0 lost events.

**Rapid-fire extras (one line each):**
- *DBMS vs file system?* DBMS adds integrity, concurrency, transactions, security, recovery, query language.
- *Data independence?* Change physical (easy) or logical (hard) schema without breaking the level above.
- *Weak entity?* No own key; identified by owner's key + partial key; total participation.
- *M:N in tables?* Junction table with two FKs as composite PK.
- *Trigger?* Code auto-run on INSERT/UPDATE/DELETE; good for audits, risky for business logic.
- *Stored procedure vs function?* Procedure via CALL, can COMMIT; function returns a value, used in SELECT.
- *CHAR vs VARCHAR?* CHAR fixed length padded; VARCHAR variable. In PG use TEXT/VARCHAR.
- *What is a cursor?* A pointer to iterate a result set row by row (server-side cursors for large results).
- *OLTP vs OLAP?* OLTP: many short transactions, normalized (Postgres). OLAP: few big analytical scans, denormalized, columnar (Redshift, BigQuery).
- *Redis persistence?* RDB snapshots + AOF log; eviction like allkeys-lru.

---

## 18. Last-minute Revision

**Basics:** DBMS > files (integrity, concurrency, ACID, security, recovery). 3 levels: external / conceptual / internal. Physical independence easy, logical hard.

**ER → tables:** entity → table; multi-valued attr → separate table; 1:N → FK on N side; M:N → junction table; weak entity → PK = owner PK + partial key, ON DELETE CASCADE.

**Keys:** super ⊇ candidate (minimal) ⊇ primary. Alternate = unused candidates. FK can be NULL; PK cannot; UNIQUE allows NULLs. Composite key `(player_id, idempotency_key)`.

**Relational algebra:** σ rows (WHERE), π columns (SELECT), ⋈ join, ∪ ∩ − (UNION/INTERSECT/EXCEPT), ÷ "for all" (double NOT EXISTS).

**SQL order:** FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT.

**SQL patterns:**
| Ask | Use |
|---|---|
| 2nd highest | `MAX(x) WHERE x < (SELECT MAX(x))` |
| Nth / top N per group | `DENSE_RANK() OVER (PARTITION BY g ORDER BY x DESC)` → filter outside |
| Never did X | `NOT EXISTS` (not `NOT IN` — NULL trap) |
| Duplicates / delete dups | `GROUP BY HAVING COUNT(*)>1` / `DELETE ... USING ... a.id > b.id` |
| Previous row / consecutive | `LAG/LEAD` |
| Running total | `SUM() OVER (ORDER BY d ROWS UNBOUNDED PRECEDING)` |
| Pivot counts | `COUNT(*) FILTER (WHERE ...)` / `SUM(CASE ...)` |
| Hierarchy | `WITH RECURSIVE` (anchor UNION ALL step) |
| ROW_NUMBER / RANK / DENSE_RANK | 1,2,3 / 1,1,3 / 1,1,2 |

**Normalization:** 1NF atomic · 2NF no partial dep · 3NF no transitive (X super key or A prime) · BCNF every determinant super key · 4NF no MVD. Candidate keys: start from attributes never on RHS, compute closures. Lossless if common attrs are a key of one side. 3NF always dependency-preserving; BCNF not always.

**Transactions:** ACID (undo/WAL, constraints, locks/MVCC, fsync). States: active → partially committed → committed / failed → aborted. Precedence graph cycle = not conflict serializable. Strict ⊂ cascadeless ⊂ recoverable.

**Concurrency:**
| Level | Prevents |
|---|---|
| Read Committed (PG default) | dirty read |
| Repeatable Read | + non-repeatable (PG: + phantom); write skew still possible |
| Serializable (SSI) | everything; retry on 40001 |
- S/X locks; strict 2PL holds X locks till commit; deadlock = wait-for cycle → victim aborted (40P01) → fixed order + jittered retry.
- MVCC: xmin/xmax versions, snapshots, readers never block writers, VACUUM cleans dead tuples.
- Lost update fixes: `FOR UPDATE` (pessimistic) · version column (optimistic) · `SET stock = stock-1 WHERE stock > 0` (atomic). **206 retries → 129 ms for 200 buyers.**
- Deadlock story: FOR UPDATE vs FK FOR KEY SHARE → 0/200 → `FOR NO KEY UPDATE` + lock order + retry on 40P01.
- Idempotency: unique key + `ON CONFLICT DO NOTHING` (1.45M replays, 0 duplicate rewards). Outbox: event row in same txn + Debezium reads WAL (0 lost events in 60 s Kafka outage).

**Recovery:** WAL rule = log before data, log flushed before commit. Steal → undo, no-force → redo. Checkpoint limits replay. ARIES = analysis → redo (repeat history) → undo (CLRs).

**Indexing:** 8 KB pages; B+ tree = keys in internals, data in linked leaves, height 3–4, range scans. Hash = equality only. Clustered = one, rows in order (InnoDB); PG = heap, all secondary. Composite = leftmost prefix, equality columns first. Covering → index-only scan. Index hurts: low selectivity, function on column, `%x` LIKE, many writes.

**Query processing:** parse → rewrite → plan (cost-based, uses stats) → execute. Joins: nested loop (small outer + index), hash (big equality), merge (sorted). EXPLAIN ANALYZE: bottom-up, estimated vs actual rows, Rows Removed by Filter, sort to disk, loops. Seq scan 118 ms → index scan 0.08 ms; my match history 0.8 ms. Keyset pagination over OFFSET.

**Scaling:** index → cache → replicas → scale up → partition → shard. Sync = RPO 0, slower; async = fast, may lose data (**RTO 0.7 s, 2.8 s lost after 5 s partition**). Read-your-writes → read from primary after write. Shard key: high cardinality, even, in most queries. Consistent hashing moves ~1/N keys (**Jump Hash 2→3 moved ~33%**). Pooling: PG = process per connection; PgBouncer transaction mode; **pool paused → `SHOW POOLS`**. Parameterized queries (`$1`) stop SQL injection.

**Caching:** cache-aside (read: cache → DB → set EX; write: DB → DEL), write-through, write-back (loss risk). Stampede → lock/jitter; penetration → cache nulls/Bloom.

**CAP/PACELC:** partition → C or A; else → L or C (805 ms remote vs 35 ms local). CAP-C ≠ ACID-C. BASE = basically available, soft state, eventual consistency.

**NoSQL:** KV (Redis, DynamoDB) · document (MongoDB) · wide-column (Cassandra: partition + clustering key, write-heavy) · graph (Neo4j). Redis: single-threaded atomic commands; String/Hash/List/Set/ZSET; RDB + AOF; `allkeys-lru`; leaderboard = ZADD/ZINCRBY/ZREVRANGE; rate limit = INCR + EXPIRE; TTL Chat = HASH/LIST/SET + EXPIRE.

**N+1:** 1 list query + N child queries → fix with JOIN / `IN`/`ANY` batch / eager loading.

**Your 5 best DB stories (say them with numbers):**
1. Deadlock: FOR UPDATE vs FOR KEY SHARE → FOR NO KEY UPDATE, lock order, jittered retry on 40P01.
2. Contention: optimistic 206 retries → conditional update, 129 ms for 200 buyers (vs 832–984 ms).
3. Outbox + Debezium CDC on WAL → 0 lost events in a 60 s Kafka outage.
4. Async failover → RTO 0.7 s; RPO 0 healthy, 2.8 s lost after 5 s partition.
5. PgBouncer pool paused → half the requests failed → found with `SHOW POOLS`.
