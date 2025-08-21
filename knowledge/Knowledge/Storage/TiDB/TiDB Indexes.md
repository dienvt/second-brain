Reference

TiDB

Indexed

Index Lưu ở đâu

Understand EXPLAIN output

Explain Statements Using Aggregation

  

# Reference

[https://docs.pingcap.com/tidb/v5.4/explain-overview](https://docs.pingcap.com/tidb/v5.4/explain-overview)

[https://docs.pingcap.com/tidb/v5.4/explain-indexes#point_get-and-batch_point_get](https://docs.pingcap.com/tidb/v5.4/explain-indexes#point_get-and-batch_point_get)

  

# TiDB

  

# Indexed

## Index Lưu ở đâu

Theo như explain thì index được lưu ở KV

[https://docs.pingcap.com/tidb/stable/tidb-best-practices#sql-on-kv](https://docs.pingcap.com/tidb/stable/tidb-best-practices#sql-on-kv)

  

[https://www.pingcap.com/blog/tidb-internal-data-storage/?_gl=1*1mro44n*_gcl_au*ODk5MjMzODIyLjE2OTU0NzcyNjY.&_ga=2.65183213.2131193294.1695477267-1350938938.1695477267](https://www.pingcap.com/blog/tidb-internal-data-storage/?_gl=1*1mro44n*_gcl_au*ODk5MjMzODIyLjE2OTU0NzcyNjY.&_ga=2.65183213.2131193294.1695477267-1350938938.1695477267)

  
  

```SQL
+-------------------------------+---------+-----------+--------------------------------+-----------------------------------+
| id                            | estRows | task      | access object                  | operator info                     |
+-------------------------------+---------+-----------+--------------------------------+-----------------------------------+
| IndexLookUp_10                | 1.00    | root      |                                |                                   |
| ├─IndexRangeScan_8(Build)     | 1.00    | cop[tikv] | table:t1, index:intkey(intkey) | range:[123,123], keep order:false |
| └─TableRowIDScan_9(Probe)     | 1.00    | cop[tikv] | table:t1                       | keep order:false                  |
+-------------------------------+---------+-----------+--------------------------------+-----------------------------------+
3 rows in set (0.00 sec)
```

## **Understand EXPLAIN output**

```SQL
Query OK, 0 rows affected (0.96 sec)

Query OK, 2 rows affected (0.02 sec)
Records: 2  Duplicates: 0  Warnings: 0

+-------------------------------+---------+-----------+---------------------+---------------------------------------------+
| id                            | estRows | task      | access object       | operator info                               |
+-------------------------------+---------+-----------+---------------------+---------------------------------------------+
| IndexLookUp_10                | 10.00   | root      |                     |                                             |
| ├─IndexRangeScan_8(Build)     | 10.00   | cop[tikv] | table:t, index:a(a) | range:[1,1], keep order:false, stats:pseudo |
| └─TableRowIDScan_9(Probe)     | 10.00   | cop[tikv] | table:t             | keep order:false, stats:pseudo              |
+-------------------------------+---------+-----------+---------------------+---------------------------------------------+
3 rows in set (0.00 sec)
```

The following describes the output of the `EXPLAIN` statement above:

- `id` describes the name of an operator, or sub-task that is required to execute the SQL statement. See [Operator overview](https://docs.pingcap.com/tidb/v5.4/explain-overview#operator-overview) for additional details.
- `estRows` shows an estimate of the number of rows TiDB expects to process. This number might be based on dictionary information, such as when the access method is based on a primary or unique key, or it could be based on statistics such as a CMSketch or histogram.
- `task` shows where an operator is performing the work. See [Task overview](https://docs.pingcap.com/tidb/v5.4/explain-overview#task-overview) for additional details.
- `access object` shows the table, partition and index that is being accessed. The parts of the index are also shown, as in the case above that the column `a` from the index was used. This can be useful in cases where you have composite indexes.
- `operator info` shows additional details about the access. See [Operator info overview](https://docs.pingcap.com/tidb/v5.4/explain-overview#operator-info-overview) for additional details.

  

## **Explain Statements Using Aggregation**

When aggregating data, the SQL Optimizer will select either a Hash Aggregation or Stream Aggregation operator. To improve query efficiency, aggregation is performed at both the coprocessor and TiDB layers. Consider the following example:

```SQL
SHOW TABLE t1 REGIONS;
```