# Assignment 2: Database Selection Framework

## Introduction

Choosing the right database depends on the type of data, workload, scale, consistency needs, query patterns, cost model, and operational requirements. No single database is best for every situation. A good database selection framework compares available options and chooses the system that fits the problem.

This assignment compares five major database and storage categories:

1. Relational databases
2. NoSQL databases
3. Cloud data warehouses
4. In-memory databases
5. Object storage

---

## 1. Relational Databases

Relational databases store data in structured tables with rows and columns. They support SQL, transactions, constraints, joins, and strong consistency. They are commonly used for business applications where data accuracy is critical.

### Common Options

| Database | Strengths | Best Use Case |
|---|---|---|
| PostgreSQL | Open-source, advanced SQL, strong consistency, JSON support, extensibility | Complex applications, analytics-friendly OLTP systems |
| MySQL | Simple, fast, widely used, large hosting ecosystem | Web applications, read-heavy workloads |
| Microsoft SQL Server | Strong enterprise tooling, BI integration, Windows ecosystem support | Enterprise applications, finance, reporting |
| Oracle Database | Very mature, highly scalable, advanced enterprise features | Large banks, telecom systems, mission-critical enterprise workloads |
| MariaDB | MySQL-compatible, open-source, good performance | MySQL replacement where open-source licensing matters |

### PostgreSQL vs MySQL

PostgreSQL is usually preferred when the application needs complex queries, strict correctness, advanced indexing, JSON support, and extensibility. MySQL is often preferred when the workload is simpler, read-heavy, and needs broad hosting support.

### SQL Server vs Oracle

SQL Server is a strong choice for organizations already using Microsoft tools such as .NET, Power BI, and Azure. Oracle is often selected for very large enterprise systems that require advanced performance, partitioning, replication, and reliability features.

### When to Pick One Over Another

- Pick **PostgreSQL** when you need strong SQL support, complex joins, transactions, and flexibility.
- Pick **MySQL** when building a simple web application with high read traffic and common hosting requirements.
- Pick **SQL Server** when the business already depends on Microsoft enterprise tools.
- Pick **Oracle** when the workload is extremely large, mission-critical, and enterprise licensing cost is acceptable.
- Pick **MariaDB** when you want MySQL compatibility with a stronger open-source direction.

### Real-World Scenario

An e-commerce platform needs to store customers, products, orders, payments, refunds, and inventory. These entities have strong relationships and require transaction safety. For example, when a customer places an order, payment, inventory update, and order creation must succeed together.

**Best choice:** PostgreSQL.

**Justification:** PostgreSQL supports ACID transactions, foreign keys, joins, indexes, and strong consistency. It is reliable for transactional systems and flexible enough to support reporting and JSON-based metadata.

---

## 2. NoSQL Databases

NoSQL databases are designed for flexible schemas, distributed scale, and specialized data access patterns. Unlike relational databases, they do not all follow the same data model. The fundamental difference between NoSQL systems is how they organize and access data.

### Common NoSQL Categories

| Type | Examples | Data Pattern | Best Use Case |
|---|---|---|---|
| Document Database | MongoDB, Couchbase | JSON-like documents | Flexible application data |
| Key-Value Store | DynamoDB, Redis, Riak | Key mapped to value | Fast lookup by key |
| Wide-Column Store | Cassandra, HBase, Bigtable | Rows with flexible columns, distributed by partition key | High-scale writes and time-series data |
| Graph Database | Neo4j, Amazon Neptune | Nodes and relationships | Relationship-heavy queries |
| Search Database | Elasticsearch, OpenSearch | Indexed documents for search | Full-text search and log analytics |

### Fundamental Difference in Data Patterns

The main difference between NoSQL databases is not just that they are "non-relational." The difference is the access pattern they optimize for.

- **Document databases** store complete business objects together, such as a user profile with preferences and addresses.
- **Key-value databases** retrieve data using a single key, such as session ID to session data.
- **Wide-column databases** handle massive distributed writes using partition keys, such as IoT readings by device ID and timestamp.
- **Graph databases** focus on relationships, such as friends, recommendations, fraud networks, or routes.
- **Search databases** optimize text search, filtering, ranking, and log exploration.

### When to Pick One Over Another

- Pick **MongoDB** when records are naturally document-shaped and the schema changes often.
- Pick **DynamoDB** when you need serverless scale, predictable access patterns, and very high availability.
- Pick **Cassandra** when you need extremely high write throughput across multiple regions.
- Pick **Neo4j** when relationships are the main part of the problem.
- Pick **Elasticsearch/OpenSearch** when users need fast search, filtering, and text matching.

### Real-World Scenario

A social media platform wants to suggest friends based on mutual connections, interests, and relationship depth. The main query is not simply "find user by ID"; it is "find people connected through multiple relationship paths."

**Best choice:** Neo4j or another graph database.

**Justification:** A graph database stores users as nodes and relationships as edges. This makes relationship traversal faster and more natural than trying to calculate complex connection paths through many relational joins.

---

## 3. Cloud Data Warehouses

Cloud data warehouses are designed for analytics, reporting, dashboards, and large-scale SQL queries. They are usually not used for day-to-day transactional processing. Instead, they store historical and aggregated data from many systems.

### Common Options

| Warehouse | Architecture | Pricing Model | Best Use Case |
|---|---|---|---|
| Snowflake | Separates storage and compute, multi-cloud | Pay for storage and virtual warehouse compute | Multi-cloud analytics and easy scaling |
| BigQuery | Serverless, fully managed | Pay per data scanned or reserved slots | Large-scale analytics with minimal operations |
| Amazon Redshift | Cluster-based or serverless options | Pay for nodes, storage, or serverless usage | AWS-centered analytics workloads |
| Azure Synapse | Integrated with Azure ecosystem | Pay for dedicated pools, serverless queries, storage | Microsoft/Azure enterprise analytics |
| Databricks SQL | Lakehouse architecture | Pay for compute clusters/serverless SQL | Data engineering, ML, and analytics together |

### Pricing Differences

Cloud data warehouse pricing is mainly driven by storage, compute, and query usage.

- **Snowflake** charges separately for storage and compute warehouses.
- **BigQuery** commonly charges based on the amount of data scanned per query, or through reserved capacity.
- **Redshift** traditionally charges for provisioned clusters, though serverless pricing is also available.
- **Azure Synapse** charges based on dedicated SQL pools, serverless query usage, and storage.
- **Databricks** charges for compute used by clusters or SQL warehouses.

### Architectural Differences

- **Snowflake** separates storage and compute clearly, making it easy to scale different workloads independently.
- **BigQuery** is serverless, so users do not manage clusters.
- **Redshift** is deeply integrated with AWS and can be cluster-based for predictable workloads.
- **Synapse** fits well when the company already uses Azure Data Lake, Power BI, and Microsoft tools.
- **Databricks** is best when analytics, machine learning, and data lake processing need to work together.

### When to Pick One Over Another

- Pick **Snowflake** when you need multi-cloud support, easy scaling, and separate compute workloads.
- Pick **BigQuery** when you want serverless analytics and are already using Google Cloud.
- Pick **Redshift** when your data and applications are mostly on AWS.
- Pick **Synapse** when your company is Microsoft/Azure-focused.
- Pick **Databricks** when your team needs a lakehouse for data engineering, streaming, analytics, and machine learning.

### Real-World Scenario

A retail company collects sales data from stores, website activity, inventory systems, and marketing campaigns. Business teams need dashboards showing daily sales, customer trends, and regional performance.

**Best choice:** Snowflake.

**Justification:** Snowflake allows the company to separate workloads. One virtual warehouse can serve dashboards while another handles heavy data transformation. This prevents reporting users and data engineers from slowing each other down.

---

## 4. In-Memory Databases

In-memory databases store data primarily in RAM instead of on disk. This makes them extremely fast, but they are usually used for specific performance-sensitive problems rather than as the main permanent database.

### Common Options

| Database | Strengths | Problem Solved |
|---|---|---|
| Redis | Very fast key-value operations, caching, queues, pub/sub, streams | Low-latency caching and real-time operations |
| Memcached | Simple distributed cache | Basic caching for web applications |
| SAP HANA | In-memory relational and analytical processing | Enterprise analytics and real-time business processing |
| VoltDB | In-memory transactional database | High-speed transactional workloads |
| SingleStore | Distributed SQL with memory-optimized processing | Real-time analytics and operational analytics |

### Completely Different Problems They Solve

In-memory databases are not all used for the same purpose.

- **Redis** is often used as a cache, session store, rate limiter, leaderboard, queue, or real-time messaging system.
- **Memcached** is mainly used as a simple cache for frequently accessed data.
- **SAP HANA** is used for enterprise-scale real-time analytics and business applications.
- **VoltDB** focuses on very fast transaction processing.
- **SingleStore** supports fast SQL analytics on fresh operational data.

### When to Pick One Over Another

- Pick **Redis** when you need rich data structures, caching, counters, queues, and real-time features.
- Pick **Memcached** when you only need simple caching with minimal complexity.
- Pick **SAP HANA** when an enterprise needs real-time analytics inside a business application ecosystem.
- Pick **VoltDB** when the workload needs extremely fast transactions with strict performance needs.
- Pick **SingleStore** when the business needs fast SQL analytics on rapidly changing data.

### Real-World Scenario

A food delivery app needs to show driver locations, manage user sessions, apply rate limits to APIs, and cache restaurant menus.

**Best choice:** Redis.

**Justification:** Redis provides low-latency reads and writes, supports expiring keys, counters, geospatial queries, pub/sub, and queues. These features solve real-time application problems that a traditional disk-based database would handle more slowly.

---

## 5. Object Storage

Object storage stores files as objects inside buckets or containers. It is commonly used for images, videos, backups, logs, data lake files, documents, and archives. It is not designed for relational queries or frequent small row updates.

### Common Options

| Storage Service | Cloud Provider | Strengths | Best Use Case |
|---|---|---|---|
| Amazon S3 | AWS | Mature ecosystem, high durability, many storage classes | General-purpose object storage and data lakes |
| Google Cloud Storage | Google Cloud | Strong integration with BigQuery and GCP analytics | GCP-based analytics and application storage |
| Azure Blob Storage | Microsoft Azure | Strong Microsoft and Azure integration | Enterprise storage in Azure environments |
| Cloudflare R2 | Cloudflare | S3-compatible API, no egress fees in many cases | Public asset delivery and cost-sensitive workloads |
| MinIO | Self-hosted or private cloud | S3-compatible, runs on private infrastructure | On-premises or private cloud object storage |

### When to Pick One Over Another

- Pick **Amazon S3** when the application is on AWS or needs the broadest object storage ecosystem.
- Pick **Google Cloud Storage** when data is mainly used with Google Cloud tools such as BigQuery.
- Pick **Azure Blob Storage** when the organization uses Azure, Microsoft 365, or enterprise Microsoft tooling.
- Pick **Cloudflare R2** when egress cost is a major concern and objects are served publicly.
- Pick **MinIO** when data must stay on-premises or in a private cloud.

### When Vendor Lock-In Becomes a Real Concern

Vendor lock-in becomes a real concern when the storage system is deeply connected to cloud-specific services, APIs, security policies, lifecycle rules, event systems, and data processing tools.

Lock-in is less risky when the application only stores and retrieves files using standard S3-compatible APIs. It becomes more serious when the architecture depends on provider-specific features such as:

- AWS Lambda triggers from S3 events
- S3-specific lifecycle policies and Glacier archive tiers
- Azure Blob integration with Microsoft enterprise identity
- BigQuery external tables over Google Cloud Storage
- Provider-specific encryption, access control, and replication features
- Large data volumes that are expensive to move because of egress fees

### Real-World Scenario

A video streaming platform stores millions of video files, thumbnails, subtitles, and logs. The application runs mostly on AWS and uses CloudFront for delivery, Lambda for processing uploads, and S3 lifecycle rules for archiving older content.

**Best choice:** Amazon S3.

**Justification:** S3 integrates naturally with CloudFront, Lambda, IAM, and archival storage classes. However, vendor lock-in becomes a concern because moving petabytes of video to another cloud would be expensive and the processing pipeline depends on AWS-specific services.

---

## Final Database Selection Framework

| Requirement | Best Category | Example Choice |
|---|---|---|
| Strong transactions and relationships | Relational database | PostgreSQL |
| Flexible documents | NoSQL document database | MongoDB |
| Massive key-based scale | NoSQL key-value database | DynamoDB |
| Relationship traversal | Graph database | Neo4j |
| Full-text search | Search database | Elasticsearch |
| Business intelligence and analytics | Cloud data warehouse | Snowflake or BigQuery |
| Low-latency caching | In-memory database | Redis |
| Files, backups, logs, and media | Object storage | Amazon S3 |

## Conclusion

The best database depends on the workload. Relational databases are ideal for structured transactional systems. NoSQL databases are useful when data patterns require flexibility, scale, graph traversal, or search. Cloud data warehouses are designed for analytics. In-memory databases solve low-latency performance problems. Object storage is best for large files, backups, logs, and data lake storage.

A strong database selection decision should always start from the application's data model, query pattern, consistency requirement, scaling need, and cost constraints.
