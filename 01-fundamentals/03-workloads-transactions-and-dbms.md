# Workloads, Transactions, ACID, and DBMS vs. RDBMS

## 1. What is a database workload?

A **workload** describes how a database is used: the kinds of operations it handles, their frequency, the amount of data involved, the concurrency level, and the response-time requirements.

Two widely used workload categories are **OLTP** and **OLAP**. They describe typical patterns, not absolute rules; real systems can contain both kinds of work.

## 2. OLTP — Online Transaction Processing

OLTP systems support the routine operations that keep an application running. Examples include recording a payment, placing an order, marking attendance, or booking a seat.

Typical characteristics:

- Many concurrent users or application requests.
- Frequent, relatively small operations.
- Short response-time requirements.
- Many reads and writes involving individual records or small sets of records.
- Strong correctness requirements, commonly supported by transactions and constraints.

Examples:
- A student pays course fees online.
- An attendance application records a student's presence.
- A cab-booking service confirms one trip.
- An online store records an order.

OLTP is not defined solely by “simple SELECT statements.” A transaction may contain several reads and writes and must preserve the application's invariants.

## 3. OLAP — Online Analytical Processing

OLAP systems support analysis, reporting, and decision-making. Instead of focusing on one customer's immediate action, they often aggregate or compare data across many records and time periods.

Typical characteristics:

- Queries that scan or aggregate large datasets.
- Grouping, filtering, joins, and statistical calculations.
- Analysis of historical as well as recent data.
- Workloads commonly run by analysts, data scientists, product teams, or leadership.

Examples:
- Comparing student enrollment by year across the last five years.
- Finding which city has the most premium users.
- Analyzing monthly revenue and identifying when sales declined.

OLAP queries may take longer than OLTP operations, but duration depends on data size, system design, hardware, indexes, caching, and query complexity. OLAP is not necessarily limited to a small number of users.

## 4. OLTP vs. OLAP at a glance

| Feature | OLTP | OLAP |
|---|---|---|
| Main purpose | Run day-to-day operations | Analyze data and support decisions |
| Typical operations | Small reads and writes, transactions | Large scans, joins, and aggregations |
| Data focus | Often current operational state | Often historical and combined data |
| Response-time target | Usually short and predictable | May tolerate longer analytical queries |
| Common users | Customers, employees, application services | Analysts, data scientists, managers |
| Example | Record one fee payment | Compare revenue across five years |

These are typical tendencies, not strict boundaries. An OLTP system can produce reports, and an OLAP platform can ingest fresh data.

## 5. Transactional databases and transactions

A **transaction** is a logical unit of work made up of one or more operations. Atomicity means that the transaction's effects are treated as a whole: it should not leave only part of the transaction committed.

Imagine a student paying fees. The application may need to:
1. Debit a wallet or initiate a payment.
2. Record the payment.
3. Update the student's outstanding balance or fee status.

The database can make database-side changes atomic when they are included in one transaction. But a real payment may also involve an external bank or payment gateway. A local database transaction cannot automatically roll back a charge already performed by an independent external system. Distributed workflows often need idempotency, reconciliation, and patterns such as a saga or compensating action.

### A simplified SQL transaction

Syntax varies somewhat by database system, but a transaction often looks like this:

```sql
BEGIN;

INSERT INTO Payments (payment_id, student_id, amount)
VALUES (9001, 101, 5000.00);

UPDATE Students
SET fee_status = 'Paid'
WHERE student_id = 101;

COMMIT;
```

If an error occurs before the transaction commits, the application can roll back the database transaction:

```sql
ROLLBACK;
```

This is a simplified illustration; the table names and columns must exist, and production payment workflows need more careful handling of retries, duplicate requests, and external payment confirmation.

## 6. How databases recover from crashes

### 6.1 Transaction logs and write-ahead logging

Many database systems use a transaction log to record information needed for recovery. With **write-ahead logging (WAL)**, relevant log records are written to durable storage before corresponding changed data pages are written out. After a crash, the database uses its recovery procedure to restore a consistent state.

Recovery may **redo** operations from committed transactions and **undo** incomplete work, depending on the DBMS's architecture. It is misleading to say that every system simply reads a log and finalizes every completed transaction: recovery algorithms and log formats vary.

Durability also depends on configuration and the failures covered by the system's guarantees. Backups, replication, storage reliability, and recovery testing remain important.

## 7. Isolation: locks and MVCC

When many transactions run at the same time, the DBMS must coordinate them to prevent unacceptable interference.

- **Locks:** control access to data or resources. Depending on the database and operation, locks may be held on rows, pages, tables, or other resources.
- **MVCC (Multi-Version Concurrency Control):** maintains multiple versions of data so readers can often see a consistent snapshot while writers make changes.

MVCC does not mean that no one can ever observe an intermediate state under every possible isolation level. Exact behavior depends on the DBMS and transaction isolation level. Isolation is about the consistency guarantees offered to concurrent transactions, not a universal rule that every transaction sees the entire database frozen in time.

### Example: the last seat

Suppose one seat remains and two customers attempt to reserve it simultaneously. The system needs a database-enforced rule—such as a unique constraint on the seat booking or a suitable locking/conditional-update strategy—so that both transactions cannot successfully reserve the same seat.

Transactions and isolation help, but application and schema design must also express the invariant correctly.

## 8. ACID properties

ACID describes four important transaction properties.

### A — Atomicity

**All or nothing.** If a transaction fails, its partial database effects are rolled back rather than left as a partially committed result.

### C — Consistency

A successful transaction preserves the database's declared constraints and invariants, assuming the transaction and schema correctly encode the required rules.

Consistency does not mean that the DBMS automatically understands every business rule. Developers must model rules such as valid fee statuses, unique bookings, and permitted balance changes.

### I — Isolation

Concurrent transactions are coordinated according to the database's isolation guarantees. The exact behavior depends on the isolation level; some levels permit phenomena that stricter levels prevent.

### D — Durability

After a transaction commits successfully, the database is designed to preserve its effects across the failures covered by its durability configuration. This is not an unconditional guarantee against every conceivable event, such as total loss of all storage without a usable backup.

## 9. DBMS vs. RDBMS

A **DBMS (Database Management System)** is software that manages data. Depending on the product, responsibilities include storing and querying data, controlling access, coordinating concurrent work, enforcing rules, and recovering after failures.

An **RDBMS (Relational Database Management System)** is a DBMS that implements the relational data model. Relational systems typically organize data into tables and support keys, constraints, and relational queries, commonly using SQL.

Examples of relational database systems include PostgreSQL, MySQL, Oracle Database, Microsoft SQL Server, and SQLite. Examples of non-relational DBMS products include MongoDB (document-oriented), Redis (commonly used as a key-value store), and Apache Cassandra (wide-column).

A useful relationship is:

- **DBMS:** the broad category.
- **RDBMS:** the relational subset of that category.

The phrase “NoSQL database” usually refers to a non-relational database system, but capabilities vary by product. Some non-relational systems offer transactions and schemas of their own; “NoSQL” does not mean “no rules” or “no reliability.”

## Quick revision

- **Workload:** the pattern of operations a database is expected to handle.
- **OLTP:** frequent operational transactions, often involving small sets of rows.
- **OLAP:** analytical queries over large or combined datasets.
- **Transaction:** a logical unit of work.
- **COMMIT:** successfully commits a transaction.
- **ROLLBACK:** cancels uncommitted transactional changes.
- **WAL:** a logging strategy that writes required log records ahead of corresponding data-page writes.
- **Lock:** a mechanism for coordinating concurrent access.
- **MVCC:** concurrency control based on multiple data versions.
- **ACID:** Atomicity, Consistency, Isolation, Durability.
- **DBMS:** software for managing data.
- **RDBMS:** a DBMS based on the relational model.

### One-sentence takeaway

**OLTP keeps operational systems correct while work is happening; OLAP helps people learn from data; transactions and DBMS mechanisms provide the tools to coordinate, constrain, and recover database changes.**
