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

---

### Database Compression

#### Observation

I/O is (traditionally) the main bottleneck during query execution. If the DBMS still needs to read data, we need to ensure that it maximizes the amount of useful information it can extract from it.

Key trade-off is **speed** vs. **compression ratio**

- Compressing the data reduces DRAM requirements and processing.

Database compression: Reduce the size of the database physical representation to increase the # of values accessed and processed per unit of computation or I/O.

Goal #1: Must produce fixed-length values.

Goal #2: Must be a lossless scheme.

Goal #3: Ideally postpone decompression for as long as possible during query execution.

#### Lossless vs. lossy compression

When a DBMS uses compression, it is always lossless because people don't like losing data.

Any kind of lossy compression must be performed at the application level.

Reading less than the entire data set during query execution is sort of like of compression.

- Approximate Query Processing

#### Compression Granularity

Choice #1: Block-level

- Compress a block (e.g., database page, RowGroup) of tuples in a table.

Choice #2: Tuple-level

- Compress the contents of the entire tuple(NSM-only).

Choice #3: Attribute-level

- Compress a single attribute value within one tuple.

- Can target multiple attributes for the same tuple.

Choice #4: Column-level

- Compress multiple values for one or more attributes stored for multiple tuples (DSM-only).

*NSM stand for N-ary Storage Model, DSM stand for Decomposition Storage Model*

#### Naive Compression

Compress data using a general-purpose algorithm. Scope of compression is only based on the data provided as input. (Snappy, Zstd, ...)

Considerations:

- Computational overhead

- Compress vs. decompress speed

The DBMS must decompress data first before it can be read and (potentially) modified.

- Even if the algo uses dictionary compression, the DBMS cannot access the dictionary's contents.

- This limits the practical scope of the compression scheme.

These schemes alse do not consider the high-level meaning or semantics of the data.

#### Columnar Compression

Run-length Encoding

Dictionary Encoding

Bitmap Encoding

Delta Encoding

Bit Packing

#### Run-length Encoding

Compress runs of the same value in a single column into triplets:

- The value of the attribute.

- The start position in the column segment.

- The # of elements in the run.

Requires the columns to be sorted intelligently to maximize compression opportunities.

Sometimes alse called *null suppression* if the DBMS only tracks empty space.

#### Dictionary Compression

Replace frequent values with smaller fixed-length codes and then maintain a mapping (dictionary) from the codes to the original values

- Typically, one code per attribute value.

- Most widely used native compression scheme in DBMSs.

The ideal dictionary scheme supports fast encoding and decoding for both point and range queries.

###### Dictionary Construction

Choice #1: All-At-One

- Compute the dictionary for all the tuples at a given point of time.

- New tuples must use a separate dictionary, or the all tuples must be recomputed.

- This is easy to do if the file is immutable.

Choice #2: Incremental

- Merge new tuples in with an existing dictionary.

- Likely requires re-encoding to existing tuples.

###### Dictionary Scope

Choice #1: Block-level

- Only include a subset of tuples within a single table.

- DBMS must decompress data when combining tuples from different block (e.g., hash table for joins).

Choice #2: Table-level

- Construct a dictionary for the entire table.

- Better compression ratio, but expensive to update.

Choice #3: Multi-table

- Can be either subset or entire tables.

- Sometimes helps with joins and set operations.

###### Encoding / Decoding

Encode/Locate: For a given uncompressed value, convert it into its compressed form.

Decode/Extract: For a given compressed value, convert it back into its original form,

No magic hash function will do this for us.

Order-preserving ending:

- The encoded values need to support sorting in the same order as original values.

###### Dictionary Data Structures

Choice #1: Array

- Only array of variable length strings and another array with pointers that maps to string offsets.

- Expensive to update so only usable in immutable files.

Choice #2: Hash Table

- Fast and compact.

- Unable to support range and prefix queries.

Choice #3: B+Tree

- Slower than a hash table and takes more memory.

- Can support range and prefix queries.

###### Dictionary: Array

First sort the values and then store them sequentially in a byte array.

- Need to also store the size of the value if they are variable-length.

Replace the original data with dictionary codes that are the (byte) offset into this array.

###### Exposing Dictionary to DBMS

Parquet/ORC do not provide an API to directly access a file's compression dictionary.

This means the DBMS cannot perform predicate pushdown and operate directly on compressed data before decompressing it.

Google's Artus proprietary format for Procella supports this.

###### Bitmap Encoding

Using bitmaps to represent a column can reduce its storage size if the column's cardinality is low.

###### Bitmap Index: Compression

Approach #1: General Purpose Compression

- Use standard compression algorithms (e.g., Snappy, zstd).

- Must decompress entire data chunk before DBMS can use it to process a query.

Approach #2: Byte-aligned Bitmap Codes

- Structured run-length encoding compression.

Approach #3: Roaring Bitmaps

- Modern hybrid of run-length encoding and value lists.

###### ORACLE Byte-aligned Bitmap Codes

Divide bitmap into chunks that contain different categories of bytes:

- **Gap Byte**: All the bits are **0**s.

- **Tail Byte**: Some bits are **1**s.

Encode each **chunk** that consists of some **Gap Bytes** followed by some **Tail Bytes**.

- Gap Bytes are compressed with RLE. (Run-length Encoding)

- Tail Bytes are stored uncompressed unless it consists of only 1-byte or has only one non-zero bit.

###### Observation

Oracle's BBC is an obsolete format.

- Although it provides good compression, it is slower than recent alternatives due to excessive branching,

- Word-Aligned Hybrid (WAH) encoding is a patented variation on BBC that provides better performance.

None of these support random access.

- If you want to check whether a given value is present, you must start from the beginning and decompress the whole thing.

###### Roaring Bitmaps

Store 32-bit integers in a compact two-level indexing data structure.

- Dense chunks are stored using bitmaps.

- Sparse chunks use packed arrays of 16-bit integers.

Used in Lucene, Hive, Spark, Pinot.

<img title="" src="images\roaring_bitmap.png" alt="" data-align="inline">

###### Delta Encoding

Recording the difference between values that follow each other in the same column.

- Store base value in-line or  in a separate look-up table.

- Combine with RLE to get event better compression ratios.

<img title="" src="images\delta_encoding.PNG" alt="" data-align="inline">

###### Bit Packing

If the values for an integer attribute is smaller than the range of its given data type size, then reduce the number of bits to represent each value.

Use bit-shifting tricks to operate on multiple values in a single word.

- Like in BitWeaving/Vertical.

<img src="images\bit_packing.png" title="" alt="" data-align="center">

###### Mostly Encoding

A variation of bit packing for when an attribute's values are "mostly" less than the largest size, store them with smaller data type.

- The remaining values that cannot be compressed are stored in their raw form.

![](images\mostly_encoding.png)

#### Intermediate Results

After the evaluating a predicate on compressed data, the DBMS will decompress it as it moves from the scan operator to the next operator.

- Example: Execute a hash join on two tables that use different compression schemes.

The DBMS (typically) does not recompress data during query execution. Otherwise, the system needs to embed decompression logic throughout the entire execution engine.

Dictionary encoding is not always the most effective compression scheme, but it is the most used.

The DBMS can combine different approaches for even better compression.

---
