### Modern Analytical Database System

### OLAP Commoditization

One recent trend of the last decade is the breakout OLAP engine sub-systems into standalone open-source components.

- This is typically done by organizations not in the business of selling DBMS software.

###### Examples:

- System Catalogs

- Query Optimizers

- File Format/ Access Libraries

- Execution Engines

#### System Catalogs

A DBMS tracks a database's schema (table, columns) and data files in its catalog.

- If the DBMS is on the data ingestion path, then it can maintain the catalog incrementally.

- If an external process adds data files, then it also needs to update the catalog so that the DBMS is aware of them.

Noteable implementations:

- HCatalog

- Google Data Catalog

- Amazon Glue Data Catalog

#### Query Optimizers

Extendible search engine framework for heuristic and cost-based query optimization.

- DBMS provides transformation rules and cost estimates.

- Framework returns either a logical or physical query plan.
  
  **This is the hardest part to build in any DBMS.**

Noteable implementations:

- Greenplum Orca

- Apache Calcite

#### File Formats

Most DBMS use a proprietary on-disk binary file format for their databases. The only way to share data between systems is to convert data into a common text-based format

- Examples: CSV, Json, XML

There are open-source binary file formats that make it easier to access data across systems and libraries for extracting data from files.

- Libraries provide an iterator interface to retrive (batched) columns from files.

- Examples: Parquet, Arrow, ...

#### Execution Engines

Standalone libraries for executing vectorized query operators on columnar data.

- Input is a DAG of physical operators.

- Require external scheduling and orchestration.

Noteable implementations:

- Velox

- DataFusion

- Intel OAP

---

### Analytical Database Indexes

#### Observation

OLTP DBMSs use indexes to find individual tuples without performing sequential scans.

- Tree-based indexes (B+Trees) are meant for queries with low selectivity predicates.

- Alse need to accommodate incremental updates.

But OLAP queries don't necessarily need to find individual tuples and data files are read-only.

How can we speed up sequential scans?

#### Sequential Scan Optimizations

Data Prefetching

Task Parallelization / Multi-threading

Clustering / Sorting

Late Materialization

Materialized Views / Result Caching

Data Skipping

Data Parallelization / Vectorization

Code Specialization / Compilation

#### Data Skipping

Approach #1: Approximate Queries (Lossy)

- Execute queries on a sampled subset of the entire table to produce approximate results.

- Examples: BlinkDB, Redshift, ComputeDB, ...

Approach #2: Data Pruning (Loseless)

- Use auxiliary data structures for evaluating predicates to quickly identify portions of a table that the DBMS can skip instead of examining tuples individually.

- DBMS must consider trade-offs between `scope` vs. `filter efficacy`, `manual` vs. `automatic`.

#### Data Considerations

Predicate Selectivity

- How many tuples will satisfy a query's predicates.

Skewness

- Whether an attribute has all unique values or contain many repeated values.

Clustering /  Sorting

- Whether the table is pre-sorted on the attributes accessed in a query's predicates.

#### Zone Maps

Pre-computed aggregates for the attribute values in  a block of tuples. DBMS checks the zone map first to decide whether it wants to access the block.

- Originally called **Small Materialized Aggregates (SMA)**

- DBMS automatically creates/maintains this meta-data.

Examples:

*Original Data*

| val |
|:---:|
| 100 |
| 200 |
| 300 |
| 400 |

=> *Zone Map*

| type    | val  |
|:-------:|:----:|
| *MIN*   | 100  |
| *MAX*   | 400  |
| *AVG*   | 250  |
| *SUM*   | 1000 |
| *COUNT* | 4    |

Trade-off between scope vs. filter efficacy

- If the scope is too large, the the zone maps will be useless.

- If the scope is too small, then the DBMS will spend too much checking zone maps.

Zone Maps are only useful when the target attribute's position and values are correlated.

#### Bitmap Indexes

Store a separate Bitmap for each unique value for an attribute where an offset in the vector corresponds to a tuple.

- The iᵗʰ position in the Bitmap corresponds to the iᵗʰ tuple in the table.

Typically segmented into chunks to avoid allocating large blocks of contiguous memory.

- Example: One per row group in PAX.

###### Bitmap Index: Design Choices

Encoding Scheme

- How to represent and organize data in a Bitmap.

Compression

- How to reduce the size of sparse Bitmaps.

###### Bitmap Index: Encoding

Approach #1: Equality Encoding

- Basic scheme with one Bitmap per unique value.

Approach #2: Range Encoding

- Use one Bitmap per interval instead of one per value.

- Example: PostgreSQL BRIN

Approach #3: Hierarchical Encoding

- Use a tree to identify empty key ranges.

Approach #4: Bit-sliced Encoding

- Use a Bitmap per bit location across all values.




