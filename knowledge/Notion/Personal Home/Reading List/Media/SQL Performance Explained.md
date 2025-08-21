---
Status: Finished
Author:
  - Markus Winand
Score /5: ⭐️⭐️⭐️⭐️⭐️
Type: Book
"\bGenre":
  - IT
Start date: Invalid date
---
# How do I know it?

How do I know it?

Summary

Heap Structure and B-Tree Structure

Index type

Full Table Scan

The scanned index range

The Query Optimizer

function-based index (FBI)

Data grow

Insert, Delete & Update

Indexing Equivalent Logic

Take away notes

Review

# Summary

> Write some thing about this book? what does it talk about? how author organize?

## Heap Structure and B-Tree Structure

  

Unlike the index, **the table data** is stored in a heap structure and is not sorted at all. There is neither a relationship between the rows stored in the same table block nor is there any connection between the blocks.

  

**The index leaf** nodes are stored in an arbitrary order—the position on the  
disk does not correspond to the logical position according to the index  
order. It is like a telephone directory with shuffled pages. If you search  
for “Smith” but first open the directory at “Robinson”, it is by no means  
granted that Smith follows Robinson. A database needs a second structure the B-tree.  
  

## Index type

- INDEX UNIQUE SCAN: The INDEX UNIQUE SCAN performs the tree traversal only. The Oracle database uses this operation if a unique constraint ensures that the  
    search criteria will match no more than one entry.  
    
- INDEX RANGE SCAN: The INDEX RANGE SCAN performs the tree traversal _and_ follows the leaf node chain to find all matching entries. This is the fallback operation if multiple entries could possibly match the search criteria.
- TABLE ACCESS BY INDEX ROWID: The TABLE ACCESS BY INDEX ROWID operation retrieves the row from the table. This operation is (often) performed for every matched record from a preceding index scan operation.

## Full Table Scan

The operation TABLE ACCESS FULL, also known as _full table scan_, can  
be the most efficient operation in some cases anyway, in particular  
when retrieving a large part of the table.  

This is partly due to the overhead for the index lookup itself, which  
does not happen for a TABLE ACCESS FULL operation. This is mostly  
because an index lookup reads one block after the other as the  
database does not know which block to read next until the current  
block has been processed. A FULL TABLE SCAN must get the entire table  
anyway so that the database can read larger chunks at a time (  
_multi  
block read  
_). Although the database reads more data, it might need to  
execute fewer read operations.  

  

## The scanned index range

  

## The Query Optimizer

The query optimizer, or query planner, is the database component  
that transforms an SQL statement into an execution plan. This  
process is also called  
_compiling_ or _parsing_. There are two distinct  
optimizer types.  

_Cost-based optimizers_ (CBO) generate many execution plan variations  
and calculate a  
_cost_ value for each plan. The cost calculation is based  
on the operations in use and the estimated row numbers. In the  
end the cost value serves as the benchmark for picking the “best”  
execution plan.  

_Rule-based optimizers_ (RBO) generate the execution plan using a hard-  
coded rule set. Rule based optimizers are less flexible and are seldom  
used today.  

  

## function-based index (FBI)

An index whose definition contains functions or expressions is a so-called

> CREATE INDEX emp_up_name  
> ON employees (  
> **UPPER(last_name)**);

  
  

## Data grow

```Plain
-----------------------------------------------------
| Id | Operation | Name | Rows | Cost |
------------------------------------------------------
| 0 | SELECT STATEMENT | | 1 | 972 |
| 1 | SORT AGGREGATE | | 1 | |
|* 2 | INDEX RANGE SCAN| SCALE_SLOW | 3000 | 972 |
------------------------------------------------------
Predicate Information (identified by operation id):
   2 - access("SECTION"=TO_NUMBER(:A))
       filter("ID2"=TO_NUMBER(:B))
```

The definition of the SACLE_SLOW index must start with the column SECTION otherwise it could not be used as access predicate. The condition on ID2 is  
not an access predicate so it can not follow SECTION in the index definition.  
That means the SCALE_SLOW index must have minimally three columns  
where SECTION is the first and ID2 not the second. That is exactly how it is  
in the index definition used for this test:  

> CREATE INDEX **scale_slow** ON scale_data (**section**, id1, **id2**);

The database cannot use ID2 as access predicate due to column ID1 in the second position.

```Plain
------------------------------------------------------
| Id   Operation         | Name       | Rows  | Cost |
------------------------------------------------------
|  0 | SELECT STATEMENT  |            |     1 |
|  1 |  SORT AGGREGATE   |            |     1 |
|* 2 |   INDEX RANGE SCAN| SCALE_FAST |  3000 |
------------------------------------------------------
Predicate Information (identified by operation id):
   2 - access("SECTION"=TO_NUMBER(:A) AND "ID2"=TO_NUMBER(:B))
```

The definition of the SCALE_FAST index must have columns SECTION and ID2  
in the first two positions because both are used for access predicates. We  
can nonetheless not say anything about their order. The index that was  
used for the test starts with the SECTION column and has the extra column  
ID1 in the third position:  

> CREATE INDEX **scale_fast** ON scale_data (**section**, **id2**, id1);

The column ID1 was just added so this index has the same size as  
SCALE_SLOW otherwise you might get the impression the size causes the  
difference.  

**Rule of thumb: index for equality first—then for ranges.**

## Insert, Delete & Update

The fewer indexes a table has, the better the **insert**, **delete** and **update** performance  
  

## Indexing Equivalent Logic

A logical condition can always be expressed in different ways. You  
could, for example, also implement the above shown skip logic as  
follows:  

```SQL
  WHERE (
           (sale_date < ?)
         OR
           (sale_date = ? AND sale_id < ?)
)
```

This variant only uses including conditions and is probably easier to  
understand—for human beings, at least. Databases have a different  
point of view. They do not recognize that the  
**where** clause selects all  
rows starting with the respective SALE_DATE/SALE_ID pair—provided  
that the SALE_DATE is the same for both branches. Instead, the  
database uses the entire  
**where** clause as filter predicate. We could  
at least expect the optimizer to “factor the condition SALE_DATE <= ?  
out” of the two or-branches, but none of the databases provides this  
service.  

Nevertheless we can add this redundant condition manually—even  
though it does not increase readability:  

  

```Plain
WHERE sale_date <= ?
	AND (
					(sale_date < ?)
         OR
           (sale_date = ? AND sale_id < ?)
			)
```

Luckily, all databases are able to use the this part of the **where  
  
**clause as access predicate. That clause is, however, even harder to  
grasp as the approximation logic shown above. Further, the original  
logic avoids the risk that the “unnecessary” (redundant) part is  
accidentally removed from the  
**where** clause later on.

  

## Take away notes

Note that **between** always includes the specified values, just like using the  
less than or equal to (<=) and greater than or equal to (>=) operators  

Only the part before the first wildcard serves as an access predicate. The remaining characters do not narrow the scanned index range— non-matching entries are just left out of the result. For Example: Joh%n → only **Joh** using as narrow

Avoid LIKE expressions with leading wildcards (e.g., '%TERM').

# Review

> Personal thinking? How this book help me? Who should read it?