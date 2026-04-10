  

Indexing

Introduction

References

Script

Create, alter & delete

Explain

# Indexing

## Introduction

There are several types of indexes used in database management systems to optimize data retrieval and improve query performance. The most common index types include:

1. Single-column index: This is the simplest form of index and involves creating an index on a single column. It speeds up queries that involve conditions on that specific column.
2. Composite index: Also known as a multi-column index, it involves creating an index on a combination of two or more columns. It is useful when queries involve conditions or join operations that span multiple columns.
3. Unique index: This index enforces the uniqueness of values in the indexed column(s). It prevents duplicate entries and is typically used for primary keys or columns that should have unique values.
4. Clustered index: In databases that support clustered indexes, the data in the table is physically reordered based on the clustered index. Each table can have only one clustered index, and it determines the physical order of data on disk. In some databases, the primary key is automatically a clustered index.
5. Non-clustered index: Unlike clustered indexes, non-clustered indexes do not change the physical order of data. Instead, they create a separate index structure that contains the indexed column(s) and pointers to the actual data rows.
6. Bitmap index: This index type is specific to certain database systems and is used for columns with a low cardinality (few distinct values). It represents multiple values as bitmaps and allows for fast indexing and retrieval of data with certain combinations of values.
7. Function-based index: This type of index allows you to create an index on the result of a function or expression involving one or more columns. It can be useful for indexing computed or derived values.
8. Covering index: A covering index includes all the columns required to satisfy a query. When a query can be entirely resolved using the index without the need to access the actual table, it is said to be a covering index.

The choice of index type depends on the specific use case and the nature of the data and queries. Different types of indexes have strengths and weaknesses, and the selection of the appropriate index type is crucial for optimizing query performance and overall database efficiency. In practice, it's common to use a combination of different index types to address various query patterns and optimize data access.

  

## References

1. "Use The Index, Luke!": This website ([**https://use-the-index-luke.com/**](https://use-the-index-luke.com/)) is an excellent resource that provides in-depth explanations and practical examples about how indexes work, how to use them effectively, and how to optimize SQL queries with indexes. It covers index types, index design, and query optimization strategies.
2. "High-Performance MySQL: Optimization, Backups, and Replication": This book by Baron Schwartz, Peter Zaitsev, and Vadim Tkachenko is a comprehensive guide to optimizing MySQL databases, including a section on index design and best practices for improving database performance.
3. "SQL Performance Explained": This book by Markus Winand is a practical guide that explains SQL performance concepts, including indexes, in an easy-to-understand manner. It covers both theory and practice and provides insights into how indexes impact query performance.
4. "Database Systems: The Complete Book": This book by Hector Garcia-Molina, Jeffrey D. Ullman, and Jennifer Widom is a comprehensive textbook on database systems. It covers indexing in detail, along with various other database-related topics.
5. "PostgreSQL Documentation - Indexes": The official documentation for the PostgreSQL database system provides detailed information about indexing in PostgreSQL. It covers various index types and best practices for using indexes effectively.
6. "MySQL Documentation - How MySQL Uses Indexes": The official MySQL documentation provides insights into how MySQL uses indexes and tips for optimizing queries with indexes.
7. "SQL Server Indexing": This Microsoft documentation section provides information on index design, index types, and performance considerations for SQL Server databases.
8. [https://www.freecodecamp.org/news/database-indexing-at-a-glance-bb50809d48bd/](https://www.freecodecamp.org/news/database-indexing-at-a-glance-bb50809d48bd/)

  

# Script

## Create, alter & delete

Create

```SQL
CREATE INDEX idx_smart_id_partner ON virtual_accounts(smart_id, partner);
```

Alter

can not

delete

```SQL
drop index appID on OrderLog202310
```

  

## Explain

![[/Untitled 7.png|Untitled 7.png]]

To understand the flow:

- The system first scans a range of the index and applies a selection filter (`**Selection_16**`), which narrows down the row handles.
- It then limits the number of results (`**Limit_17**`), potentially further reducing the set of row handles.
- Finally, it fetches the actual rows from the table storage (`**TableRowIDScan_15**`) corresponding to the row handles obtained in the previous steps.

  

Let's break down the given output:

1. **IndexLookUp_18**: This is the top-level task. An `**IndexLookUp**` operation in TiDB is usually split into two parts: first, an index scan (`**IndexRangeScan**`) to fetch the matching row handles (essentially pointers to rows in the table), and then a table scan (`**TableRowIDScan**`) to fetch the corresponding rows using these row handles.
2. **Limit_17(Build)**: This operation applies a limit to the number of results, which means only a certain number of rows will be returned. The exact number isn't provided in this simplified output. "Build" here indicates that this is the first phase of the operation that gathers the required row handles.
3. **Selection_16**: This task filters data based on a specific condition, typically using a WHERE clause. It processes data coming from the next task (`**IndexRangeScan_14**`).
4. **IndexRangeScan_14**: This task reads data from an index. It scans a range of the index to fetch the row handles, which are pointers to the actual rows in the main table.
5. **TableRowIDScan_15(Probe)**: This is the second main part of the `**IndexLookUp**` operation. After fetching the row handles from the `**IndexRangeScan**`, this task uses these handles to fetch the actual rows from the table storage. "Probe" indicates this is the phase where the system looks up the actual row data using the row handles gathered in the "Build" phase.