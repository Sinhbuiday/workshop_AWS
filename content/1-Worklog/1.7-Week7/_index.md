---
title: "Week 7 Worklog"
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

- Database services on AWS.
- Amazon RDS & Amazon Aurora.
- Redshift - ElastiCache.

### Tasks to be carried out this week:

| Day | Task                                   | Start Date | Completion Date | Reference Source                                                                              |
| --- | -------------------------------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Database Concepts                    | 15/06/2026 | 16/06/2026    | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 2   | - Amazon RDS & Amazon Aurora           | 18/06/2026 | 18/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Amazon ElastiCache & Amazon RedShift | 20/06/2026 | 21/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Week 7 Achievements:

- **Database Concepts:**

  - **Database** is a storage system used to manage, process, and analyze data from multiple sources, allowing users to retrieve information quickly and efficiently when needed, especially in complex systems or when serving multiple user requests.
  - **Session** is the time period between when a user connects to the database system (start time) and when the connection ends (end time).
  - **Primary Key** is a column (or a combination of columns) designated to uniquely identify each record in a table.
  - **Foreign Key** is a column (or group of columns) in one table that establishes a relationship with a column in another table, helping create references and links between tables.
  - **Index** is a data structure built to improve the speed of data retrieval operations (read) on database tables, though it increases write costs and storage space to maintain the index structure. Indexes are used to quickly locate data within a table and can be created in various ways.
  - **Partitions** refer to the process of splitting large tables into smaller parts stored across different partitions. This allows queries to access only specific portions of data instead of the entire dataset, improving query speed and reducing processing costs.
  - **Execution Plan (Query Plan)** is a program or set of steps used by the SQL database management system to execute a query. When a query is submitted, the optimizer selects the most efficient method to execute it, improving query performance.
  - **Database Log** is a vital component in database design that ensures data integrity and enables recovery in case of failure. All data changes are recorded in log files associated with the database.
  - **Buffers** are temporary storage areas in memory that hold data while transferring it between locations. Database buffers store copies of data blocks, improving data retrieval speed and reducing write latency before synchronizing changes with the database.
  - **NoSQL Database** (“Not Only SQL”) does not use traditional tabular structures. NoSQL databases come in various types based on their data models.
  - **Main types include document**, key-value, wide-column, and graph databases. These models offer flexible schemas and scalability suited for large data volumes and high user loads.
  - **OLTP (Online Transaction Processing)** systems ensure transaction reliability and data integrity. Each transaction consists of multiple database operations. OLTP is optimized for fast, real-time operations and is typically used in transactional systems.
  - **OLAP (Online Analytical Processing)** systems are designed for historical data analysis, often aggregating OLTP data for reporting, analytics, and forecasting. OLAP databases (data warehouses) support advanced analytics and reporting tools to generate insights and predictions.

- **Amazon RDS & Amazon Aurora:**

  - **Amazon RDS**: A managed database service on AWS that provides low-cost, scalable RDBMS options without requiring access to underlying infrastructure. It supports **Aurora, MySQL, PostgreSQL, MSSQL, Oracle, and MariaDB**.
  - **Key features of Amazon RDS:**
    1. Automated backups (database logs retained for up to 35 days).
    2. Creation of Read Replicas for read-heavy workloads (e.g., reporting).
    3. Read Replicas can be detached from the primary node.
    4. Automatic failover mechanism (Primary/Standby) via Multi-AZ deployment.
    5. Commonly used for OLTP applications.
    6. Encryption support for data at rest and in transit.
    7. Network-level protection using Security Groups and NACLs.
    8. Instance resizing capability (scaling up/down).
    9. Automatic storage scaling.
  - **Amazon Aurora**: A cloud-optimized relational database engine compatible with MySQL and PostgreSQL. It is part of Amazon RDS but provides enhanced performance, fault tolerance, and scalability.
  - **Key features of Amazon Aurora:**
    1. **Backtrack** – Restore the database to a previous point in time.
    2. **Clone** – Create database copies quickly.
    3. **Global Database** – One master with multiple read replicas across different AWS Regions.
    4. **Multi-Master** – Multiple master databases for high availability.

- **Amazon ElastiCache & Amazon Redshift:**

  - **Amazon Redshift**: A fully managed data warehouse service optimized for OLAP workloads, based on PostgreSQL.
  - Redshift uses a **Massively Parallel Processing (MPP)** architecture where data is partitioned and stored across compute nodes (which include both compute and storage). A leader node coordinates and aggregates queries.
  - Data is stored in **columnar format**, ideal for analytical queries.
  - Redshift supports **SQL** and standard **JDBC/ODBC** drivers.
  - Cost optimization features include **Transient Clusters** and **Redshift Spectrum** for querying S3 data.
  - **Amazon ElastiCache**: A managed in-memory caching service that supports **Redis** and **Memcached** engines.
  - ElastiCache automatically detects and replaces failed nodes.
  - Commonly deployed alongside databases to cache frequently accessed data and reduce database load.
  - For new workloads, **Redis** is recommended due to its richer features and better performance.
  - Using ElastiCache typically requires modifying the application’s caching logic.
