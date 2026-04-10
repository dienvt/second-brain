---
title: "Understand EXPLAIN"
date: 2026-01-14
tags:
  - engineering
  - database
  - storage
  - percona
---

  

# Understand EXPLAIN

Understanding the result of the `**EXPLAIN**` command in Percona Server for MySQL (or any MySQL-based system) can be quite complex, as it provides a detailed breakdown of how the MySQL optimizer plans to execute a query. Here are the key components you'll typically encounter in an `**EXPLAIN**` output and what they mean:

1. **id**: This is a select identifier. It represents the order in which the sets of rows are processed. Lower numbers are processed first. A query might be broken down into multiple parts (subqueries, joins, etc.), each with a different `**id**`.
2. **select_type**: This field describes the type of select query. It can be values like `**SIMPLE**` (simple select without unions or subqueries), `**PRIMARY**` (the first or primary select statement), `**SUBQUERY**`, `**DERIVED**` (derived table select), and others.
3. **table**: Shows the table to which the row of output refers. In complex queries involving joins or subqueries, each row can refer to a different table.
4. **partitions**: Indicates which partitions of a partitioned table will be queried. This is relevant only for tables that have been partitioned.
5. **type**: One of the most important fields, it indicates the join type. Common values include `**ALL**` (full table scan), `**index**` (index scan), `**range**` (index range scan), `**ref**` (non-unique key lookup), `**eq_ref**` (unique key lookup), and others. Generally, `**ALL**` and `**index**` indicate less efficient queries, especially for large tables.
6. **possible_keys**: Shows which indices MySQL can choose from for this query.
7. **key**: Indicates the index MySQL actually decided to use. If this is `**NULL**`, no index is used.
8. **key_len**: Shows the length of the key that MySQL decided to use. This can give an indication of how many bytes of the index will be used to search for the desired results.
9. **ref**: Indicates which columns or constants are compared to the index named in the `**key**` field to select rows from the table.
10. **rows**: An estimate of the number of rows MySQL believes it must examine to execute the query.
11. **filtered**: Percentage of rows that will be filtered by the table condition.
12. **Extra**: This field contains additional information about how MySQL will resolve the query. Common values include:
    - `**Using index**`: Indicates that the information is retrieved from the index without accessing the table.
    - `**Using where**`: Indicates that a WHERE clause is used.
    - `**Using temporary**`: Indicates that MySQL will use a temporary table to resolve the query.
    - `**Using filesort**`: Shows that MySQL will perform an extra pass to sort the results. This is often a performance red flag, especially for large datasets.

To effectively use `**EXPLAIN**`, you'll often need to understand the specifics of your query and the underlying table structure (such as indexes, data types, etc.). It’s a powerful tool to optimize queries, as it reveals how MySQL plans to execute them and can point out potential performance bottlenecks. It's also worth noting that `**EXPLAIN**` provides estimated plans and costs, not the actual costs post-execution.

For complex queries, especially those involving joins or subqueries, the `**EXPLAIN**` output can be quite extensive, and you might need to analyze each part of the query separately to fully understand and optimize it.

  

  

# Type

### **Index Scan (Full Index Scan)**

An **Index Scan**, sometimes referred to as a **Full Index Scan**, involves scanning the entire index to find the matching rows. This happens when the query cannot pinpoint a specific range of values in the index but can still benefit from the index because the columns in the query are all part of the index.

- **When It Occurs**: An index scan typically occurs when the query's WHERE clause involves a non-unique index or when the optimizer determines that a large portion of the index needs to be scanned to retrieve the relevant data.
- **Performance**: While faster than a full table scan (as indexes are generally smaller and more efficient to traverse than the entire table), an index scan can still be relatively slow, especially for large indexes.
- **Example**: A query filtering on a non-unique column (like `**SELECT * FROM table WHERE non_unique_column = 'value'**`) might result in an index scan if the value isn't highly selective.

### **Range Scan (Index Range Scan)**

An **Index Range Scan**, or simply **Range Scan**, is a more optimized use of an index compared to a full index scan. In a range scan, the database engine reads a range of rows from the index—those that fall within a certain range of values specified in the query.

- **When It Occurs**: This occurs when the query's WHERE clause allows for a range of values to be extracted from an index. It's particularly common with queries using operators like `**>**`, `**<**`, `**BETWEEN**`, and `**LIKE**`.
- **Performance**: Range scans are typically more efficient than full index scans since they target a specific subset of the index.
- **Example**: A query like `**SELECT * FROM table WHERE indexed_column BETWEEN 10 AND 20**` would likely use a range scan if `**indexed_column**` is part of an index.

### **Key Differences**

1. **Scope of Scanning**:
    - **Index Scan**: Reads the entire index.
    - **Range Scan**: Reads only a specified range within the index.
2. **Performance and Efficiency**:
    - **Index Scan**: Less efficient than range scans, especially for large indexes.
    - **Range Scan**: More efficient as it scans a limited range.
3. **Use Cases**:
    - **Index Scan**: Used when the query involves non-unique columns or less selective values.
    - **Range Scan**: Used for queries with range conditions or more selective value filtering.
4. **Optimization Considerations**:
    
    - **Index Scan**: Sometimes indicates a need for better index design or query optimization.
    - **Range Scan**: Generally indicates more effective use of indexes, but still requires monitoring and optimization for best performance.