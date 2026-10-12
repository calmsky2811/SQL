# What Is a Database?

## 1. The basic idea

A **database** is an organized collection of data. A **database management system (DBMS)** is the software used to define, store, query, update, protect, and recover that data. In everyday conversation, people sometimes use “database” to mean both the stored data and the system managing it, but the distinction is useful.

A simple mental model for a relational database is:

1. A database contains related objects, including tables.
2. A table organizes data into rows and columns.
3. Rows represent individual entries; columns represent attributes of those entries.

The folder-and-files analogy can help at first, but it is only an analogy: a database is not literally just a folder full of spreadsheet-like files. A DBMS also manages rules, transactions, concurrent access, indexes, and recovery.

## 2. What happens if the computer shuts down?

Databases are designed to persist data beyond a program's lifetime. They commonly use durable storage and transaction logs so that committed changes can survive a crash.

Consider a fee payment. The application might need to record a payment and update a student's outstanding balance. If the system crashes partway through, it should not leave the database in an inconsistent intermediate state.

Database systems use **transactions** and **recovery mechanisms** to handle this. On restart, the DBMS may use logs to redo changes from committed transactions and undo or roll back incomplete transactions, depending on its recovery design.

This is not an unconditional guarantee that every completed operation is safe under every configuration. Durability depends on the database's guarantees, storage reliability, and settings—for example, whether commit acknowledgment waits for the required data or log records to reach durable storage. Backups and tested recovery procedures are still essential.

### ACID: four transaction properties

- **Atomicity:** a transaction happens as a whole or does not take effect as a whole.
- **Consistency:** a transaction preserves the database's declared rules and constraints when used correctly.
- **Isolation:** concurrent transactions are coordinated according to the database's isolation guarantees.
- **Durability:** once a transaction is committed under the configured durability guarantees, its changes survive the failures covered by those guarantees.

These properties are a foundation for understanding why databases are used in systems such as payments, inventory, and accounting.

## 3. How databases store and find data internally

The exact implementation varies by DBMS, but several concepts are common.

### 3.1 Pages and blocks

Databases typically read and write data in units called **pages** (or similarly named units). A page can contain multiple rows or index entries. The operating system and storage device also use their own I/O units, sometimes called blocks; these terms are related but are not universally interchangeable.

The DBMS transfers pages between persistent storage and memory as needed.

### 3.2 Buffer cache

A **buffer pool** or **buffer cache** keeps database pages in RAM. If a needed page is already in memory, the DBMS may avoid reading it from storage again. This reduces I/O and can make repeated queries much faster.

A cache does not mean every frequently used value remains in RAM forever. Memory is limited, so the DBMS decides which pages to keep and which to evict.

### 3.3 Indexes: the textbook-index analogy

An index is an additional data structure that can help the DBMS locate rows without scanning an entire table.

- **B-tree-family indexes:** widely used for ordered lookups and range queries. Many systems use B-tree variants, including B+ tree designs.
- **Hash indexes or hash-based structures:** can support efficient equality lookups in systems that implement them, but are generally not suited to ordered range searches.

Indexes use extra storage and make some writes more expensive because the index may also need updating. The query optimizer chooses whether an index is useful for a particular query; creating an index does not guarantee it will be used.

**Mental model:** an index is like the index at the back of a textbook. It helps you find a topic, but it takes space and must be maintained as the book changes.

## 4. Tables, rows, and columns

A **relational database** organizes data using relations, commonly represented to learners as tables.

### 4.1 Table

A table stores data about a particular kind of entity or relationship. For example, a `customers` table might look like this:

| customer_id | name | email | signup_date | is_active |
|---:|---|---|---|---|
| 1 | Aisha Khan | aisha@example.com | 2024-01-10 | TRUE |
| 2 | Raj Patel | raj.patel@sample.org | 2024-02-15 | FALSE |
| 3 | Meera Joshi | meera.j@example.com | 2024-03-01 | TRUE |

The values shown are illustrative. Some databases use Boolean types, while others may represent true/false values differently. The original example's `1` and `0` are also common representations, but their meaning depends on the schema.

### 4.2 Row / record

A **row** (often called a record) represents one entry in the table. For example:

| customer_id | name | email | city |
|---:|---|---|---|
| 1 | Rahul Mehta | rahul.mehta@mail.com | Mumbai |

If each row represents one customer, then 1,000 customer rows represent 1,000 customer entries. The table's key and constraints determine how entries are identified and which values are permitted.

### 4.3 Column / attribute

A **column** describes an attribute shared by rows in the table, such as `email` or `signup_date`. A column has a defined type and may have constraints.

Rows do not all have to contain a concrete value for every column: a value can be `NULL` if the schema and operation allow it. `NULL` means a value is missing or unknown (or otherwise not applicable under the model); it is not the same as zero, an empty string, or the text `'NULL'`.

In relational theory, **attribute** is often the more precise term for a column's role, while “field” can mean different things in different tools.

## 5. Relational databases

Relational databases organize data into tables and model relationships between them. Tables are connected through keys and constraints—not merely because two tables happen to contain the same email or ID.

For a coaching institute, a simplified design might include:

- `Students(student_id, name, email)`
- `Attendance(attendance_id, student_id, class_date, status)`
- `Payments(payment_id, student_id, amount, paid_at)`

Here, `student_id` in `Students` can be a primary key. The matching `student_id` columns in `Attendance` and `Payments` can be foreign keys referencing it. This allows the database to enforce that attendance and payment rows refer to an existing student.

Common relational database systems include PostgreSQL, MySQL, Oracle Database, Microsoft SQL Server, and SQLite.

**Important:** a relationship is not automatically enforced just because matching IDs exist in two tables. A suitable key or foreign-key constraint must be defined, and the DBMS must enforce it.

## 6. What does NoSQL mean?

**NoSQL** is a broad category of database systems that do not rely exclusively on the traditional relational-table model. It is not one single data model, and many NoSQL systems support schemas or structured rules of their own.

Common categories include:

- **Document databases:** store records as documents, often using JSON-like structures. Example: MongoDB.
- **Key-value stores:** map keys to values. Example: Redis.
- **Wide-column stores:** organize data around partition keys and column families or related structures. Example: Apache Cassandra.
- **Graph databases:** represent entities as nodes and their relationships as edges. Example: Neo4j.

Some NoSQL systems store related data together in a document or other aggregate, which can make certain access patterns convenient. Others distribute data across many machines or optimize for specific workloads. NoSQL systems can also have relationships, indexes, transactions, and consistency guarantees; their capabilities differ by product.

## 7. Relational vs. NoSQL: which is better?

Neither category is universally better. The right choice depends on the data model, access patterns, consistency requirements, scale, operations, and team expertise.

| Consideration | Relational systems often fit well when… | NoSQL systems may fit well when… |
|---|---|---|
| Data model | Data is naturally represented as related tables | Documents, key-value access, graph relationships, or wide-column patterns fit the workload |
| Constraints | Strong schema constraints and relational integrity are central | Flexible or product-specific data modeling is useful |
| Queries | Queries combine data across tables and need rich SQL capabilities | Access patterns are well served by the chosen NoSQL model and query API |
| Scaling and distribution | The selected relational product meets the workload and availability needs | A distributed architecture or specific horizontal-scaling model is a key requirement |
| Transactions | Multi-row or multi-table transactions are important | The product's transaction and consistency model fits the application |

Avoid the oversimplification that relational databases are always accurate while NoSQL databases are always faster or more flexible. Relational databases can scale substantially, and NoSQL systems differ widely in performance, schema flexibility, transaction support, and consistency.

For a payment or accounting application, relational modeling and transactions are often a natural starting point because relationships and correctness constraints matter. For a cache, a document-oriented application, or a graph-heavy workload, a suitable NoSQL system may be a better fit. Real architectures sometimes use both.

## 8. A small SQL example

The following example illustrates how a relational table can be defined. Exact data types and Boolean syntax can vary across database systems.

```sql
CREATE TABLE Customers (
    customer_id INTEGER PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) UNIQUE,
    is_active   BOOLEAN NOT NULL
);
```

- `CREATE TABLE` defines a table.
- `PRIMARY KEY` uniquely identifies each row.
- `NOT NULL` requires a value.
- `UNIQUE` prevents duplicate values under the DBMS's rules; how multiple `NULL` values behave varies by system.
- `BOOLEAN` is supported by some DBMSs but not represented identically by every product.

## Quick revision

- **Database:** organized data; in everyday speech, sometimes also used to refer to the system managing it.
- **DBMS:** software that manages database definition, queries, updates, access, and recovery.
- **Table:** rows and columns representing structured data.
- **Row / record:** one entry.
- **Column / attribute:** one property represented across entries.
- **Page:** a common unit of database storage and I/O.
- **Buffer pool/cache:** memory holding database pages to reduce storage reads.
- **Index:** a data structure that can speed up lookups, at the cost of storage and maintenance.
- **Transaction:** a logical unit of work governed by transaction guarantees.
- **ACID:** Atomicity, Consistency, Isolation, Durability.
- **Relational database:** organizes data into related tables with keys and constraints.
- **NoSQL:** a broad family of non-relational data models, including document, key-value, wide-column, and graph systems.

### One-sentence takeaway

**A database is not just a place to keep data: a DBMS provides the structures and mechanisms to query, coordinate, constrain, and recover that data reliably.**
