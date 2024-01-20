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

<img title="" src="images\roaring_bitmap.PNG" alt="" data-align="inline">

###### Delta Encoding

Recording the difference between values that follow each other in the same column.

- Store base value in-line or  in a separate look-up table.

- Combine with RLE to get event better compression ratios.

<img title="" src="images\delta_encoding.PNG" alt="" data-align="inline">

###### Bit Packing

If the values for an integer attribute is smaller than the range of its given data type size, then reduce the number of bits to represent each value.

Use bit-shifting tricks to operate on multiple values in a single word.

- Like in BitWeaving/Vertical.

<img src="images\bit_packing.PNG" title="" alt="" data-align="center">

###### Mostly Encoding

A variation of bit packing for when an attribute's values are "mostly" less than the largest size, store them with smaller data type.

- The remaining values that cannot be compressed are stored in their raw form.

<img title="" src="images\mostly_encoding.PNG" alt="" data-align="inline">

#### Intermediate Results

After the evaluating a predicate on compressed data, the DBMS will decompress it as it moves from the scan operator to the next operator.

- Example: Execute a hash join on two tables that use different compression schemes.

The DBMS (typically) does not recompress data during query execution. Otherwise, the system needs to embed decompression logic throughout the entire execution engine.

Dictionary encoding is not always the most effective compression scheme, but it is the most used.

The DBMS can combine different approaches for even better compression.

---

### Query Execution & Processing Models

#### Execution Optimization

DBMS engineering is an orchestration of a bunch of optimizations that seek to make full use of hardware. There is not a single technique that is more important than others.

Top-3 Optimizations:

- Data Parallelization (Vectorization)

- Task Parallelization (Multi-threading)

- Code Specialization (Compilation)

#### Optimization Goals

- Approach #1: Reduce Instruction Count
  
  - Use fewer instructions to do the same amount of work.

- Approach #2: Reduce Cycles per Instruction
  
  - Execute more CPU instructions in fewer cycles.

- Approach #3: Parallelize Execution
  
  - Use multiple threads to compute each query in parallel.

#### Query Execution

A query plan is a  DAG of `operators`.

An `operator instance` is an invocation of an  operator on a unique segment of data.

A `task` is a sequence of one or more operator instances (also sometimes referred to as a `pipeline`).

<img title="" src="images/query_execution.png" alt="" data-align="center">

#### CPU Overview

CPUs organize instructions into `pipeline stages`.

The goal is to keep all parts of the processor busy at each cycle by masking delays from instructions that cannot complete in a single cycle.

Super-scalar CPUs support multiple pipelines.

- Execute multiple instructions in parallel in a single cycle if they are independent (out-of-order execution).

Everything is fast until there is a mistake...

##### DBMS / CPU Problems

Problem #1: Dependencies

- If one instruction depends on another instruction, then it cannot be pushed immediately into the same pipeline.

Problem #2: Branch Prediction

- The CPU tries to predict what branch the program will take and fill in the pipeline with its instructions.

- If it gets it wrong, it must throw away any speculative work and flush the pipeline.

##### Branch Misprediction

Because of long pipelines, CPU will speculatively execute branches. This potentially hides the long stalls between dependent instructions.

The most executed branching code in a DBMS is the filter operation during a sequential scan. But this is (nearly) impossible to predict correctly.

##### Selection Scans

From a example:

```sql
SELECT * FROM table
WHERE key > $(low) AND key < $(high)
```

- Scalar (Branching):
  
  ```python
  i = 0
  for t in table:
      key = t.key
      if (key>low) && (key<high):
          copy(t, output[i])
          i = i + 1
  ```

- Scalar (Branchless):
  
  ```python
  i = 0
  for t in table:
      copy(t, output[i])
      key = t.key
      delta = key>low ? 1 : 0) & (key<high ? 1 : 0)
      i = i + delta
  ```

`Note`: Use branchless for better performances.

#### Excessive Instructions

The DBMS needs to support different data types, so it must check a values type before it performs any operation on that value.

- This is usually implemented as giant switch statements.

- Also creates more branches that can be difficult for the CPU to predict reliably.

Example: Postgres addition for `NUMERIC` types.

#### Processing Model

A DBMS's `processing model` defines how the system executes a query plan.

- Different trade-offs for workloads (OLTP vs. OLAP).

Approach #1: **Iterator Model**

Approach #2: **Materialization Model**

Approach #3: **Vectorized / Batch Model**

###### Iterator Model

Each query plan operator implements a *next* function.

- On each invocation, the operator returns either a single tuple or a null marker if there are no more tuples.

- The operator implements a loop that calls next on its children to retrieve their tuples and then process them.

Also called *Volcano* or *Pipeline* Model.

Example flow:

<img title="" src="images/iterator_model_ex1.png" alt="" data-align="inline">

<img title="" src="images/iterator_model_ex2.png" alt="" data-align="inline">

This is used in almost every DBMS. Allows for tuple **pipelining**.

Some operators must block until their children emit all their tuples.

- Joins, Subqueries, Order By

Output control works easily with this approach.

###### Materialization Model

Each operator processes its input all at once and then emits its output all at one.

- The operator "materializes" its output as a single result.

- The DBMS can push down hints (e.g., LIMIT) to avoid scanning too many tuples.

- Can send either a materialized row or a single column.

The output can be either whole tuples (NSM) or subsets of columns (DSM).

Example flow:

<img title="" src="images/materialization_model_ex.png" alt="" data-align="inline">

Better for OLTP workloads because queries only access a small number of tuples at a time.

- Lower execution / coordination overhead.

- Fewer function calls.

Not good for OLAP queries with large intermediate results.

###### Vectorization Model

Like the Iterator Model where each operator implements a `next` function, but...

Each operator emits a **batch** of tuples instead of a single tuple.

- The operator's internal loop processes multiple tuples at a time.

- The size of the batch can vary based on hardware or queries properties.

Example flow:

<img src="images/vectorization_model_ex.png" title="" alt="" data-align="center">

Ideal for OLAP queries because it greatly reduces the number of invocations per operator.

Allows for operators to more easily use vectorized (SIMD) instructions to process batches of tuples.

#### Plan Processing Direction

**Approach #1: Top-to-Bottom (Pull)**

- Start with the root and "pull" data up from its children.

- Tuples are always passed with function calls.

**Approach #2: Bottom-to-Top (Push)**

- Start with leaf nodes and "push" data to their parents.

##### Push-Based Iterator Model

<img src="images/push_base_iterator.png" title="" alt="" data-align="center">

##### Plan Processing Direction

**Approach #1: Top-to-Bottom (Pull)**

- Easy to control output via `LIMIT`.

- Parent operator blocks until its child returns with a tuple.

- Additional overhead because operators' `next` functions are implemented as virtual functions.

- Branching costs on each `next` invocation.

**Approach #2: Bottom-to-Top (Push)**

- Allows for tighter control of caches/registers in pipelines.

- Difficult to control output via `LIMIT`.

- Difficult to implement Sort-Merge Join.

#### Parallel Execution

The DBMS executes multiple tasks simultaneously to improve hardware utilization.

- Active tasks do not need to belong to the same query.

**Approach #1: Inter-Query Parallelism**

**Approach #2: Intra-Query Parallelism**

##### Inter-Query Parallelism

Improve overall proformance by allowing multiple queries to execute simultaneously.

- Most DBMSs use a simple first-come, first-served policy.

OLAP queries have parallelizable and non-parallelizable phases. The goal is to always keep all cores active.

##### Intra-Query Parallelism

Improve the performance of a single query by executing its operators in parallel.

**Approach #1: Intra-Operator (Horizontal)**

**Approach #2: Inter-Operator (Vertical)**

These techniques are not mutually exclusive.

There are parallel algorithms for every relational operator.

##### Intra-Operator Parallelism

**Approach #1: Intra-Operator (Horizontal)**

- Operators are decomposed into independent instances that perform the same function on different subsets of data.

The DBMS inserts an `exchange` operator into the query plan to coalesce results from children operators.

<img src="images/intra-operator_pal.png" title="" alt="" data-align="center">

##### Inter-Operator Parallelism

**Approach #2: Inter-Operator (Vertical)**

- Operations are overlapped in order to pipeline data from one stage to the next without materialization.

- Workers execute multiple operators from different segments of a query plan at the same time.

- Still need exchange operators to combine intermediate results from segments.

Also called **pipelined parallelism**.

<img title="" src="images/inter-operator_pal.png" alt="" data-align="inline">

---

### Query Scheduling

#### Scheduling

For each query plan, the DBMS must provide where, when, and how to execute it.

- How many tasks should it use?

- How many CPU cores should it use?

- What CPU core should the tasks execute on?

- Where should a task store its output?

The DBMS *always* knows more than the OS.

##### Scheduling Goals

**Goal #1: Throughput**

- Maximize the # of completed queries.

**Goal #2: Fairness**

- Ensure that no query is starved for resources.

**Goal #3: Query Responsiveness**

- Minimize tail latencies (especially for short queries)

**Goal #4: Low Overhead**

- Workers should spend most of their time executiong tasks not figuring out what task to run next.

##### Process Model

A DBMS's **process model** defines how the system is architected to support concurrent requests from a multi-user application.

A **worker** is the DBMS component that is responsible for executing tasks on behalf of the client and returning the results.

We will assume that the DBMS is multi-threaded.

##### Worker Allocation

**Approach #1: One Worker per Core**

- Each core is assigned one thread that is pinned to that core in the OS.

- See [sched_setaffinity](https://man7.org/linux/man-pages/man2/sched_setaffinity.2.html).

**Approach #2: Multiple Workers per Core**

- Use a pool of workers per core (or per socket).

- Allows CPU cores to be fully utilized in case one worker at a core blocks.

##### Task Assignment

**Approach #1: Push**

- A centralized dispatcher assigns tasks to workers and monitors their progress.

- When the worker notifies the dispatcher that it is finished it is given a new task.

**Approach #2: Pull**

- Workers pull the next task from a queue, process it, and then return to get the next task.

*Notes:*

- Regardless of what worker allocation or task assignment policy the DBMS uses, it's important that workers operate on local data.

- The DBMS's scheduler must be aware of its hardware memory layout.
  
  - Uniform vs. None-Uniform Memory Access

**Uniform Memory Access (Used in Old Systems)**

<img title="" src="images/uniform_memory_access.png" alt="" data-align="inline">

**Non-Uniform Memory Access (Used in Modern Systems)**

<img title="" src="images/non-uniform_memory_access.png" alt="" data-align="inline">

#### Data Placement

The DBMS can partition memory for a database and assign each partition to a CPU.

By controlling and tracking the location of partitions, it can schedule operators to execute on workers at the closest CPU core.

See Linux’s [move_pages](https://man7.org/linux/man-pages/man2/move_pages.2.html) and [numactl](https://linux.die.net/man/8/numactl)

##### Memory Allocation

What happens when the DBMS calls `malloc`?

- Assume that the allocator doesn't already have a chunk of memory that it can give out.

Almost nothing:

- The allocator will extend the process' data segment.

- But this new virtual memory is not immediately backed by physical memory.

- The OS only allocates physical memory when there is a page fault on access.

Now after a page fault, where does the OS allocate physical memory in a NUMA (Non-Uniorm Memory Access) system?

##### Memory Allocation Location

**Approach #1: Interleaving**

- Distribute allocated memory uniformly across CPUs.

**Approach #2: First-Touch**

- At the CPU of the thread that accessed the memory location that caused the page fault.

The OS can try to move memory to another NUMA region from observed access patterns.

<img title="" src="images/data_placement_compare.png" alt="a" data-align="inline">

##### Partitioning Vs. Placement

A **partitioning** scheme is used to split the database based on some policy.

- Round-robin

- Attribute Ranges

- Hashing

- Partial/Full Replication

A **placement** scheme then tells the DBMS where to put those partitions.

- Round-robin

- Interleave across cores

###### Observation

We have the following so far:

- Task Assigment Model

- Data Placement Policy

But how to we decide how to create a set of tasks from a logical query plan?

- This is relatively easy for OLTP queries. (because it's maybe one pipeline, one task to assign to a worker and be done with it)

- Much harder for OLAP queries... (the queries can be more complicated and can depencencies between these tasks or these pipelines)

##### Static Scheduling

The DBMS decides how many threads to use to execute the query when it generates the plan.

It does *not* change while the query executes.

- The easiest approach is to just use the same # of tasks as the # of cores.

- Can still assign tasks to threads based on data location to maximize local data processing.

##### Morsel-Driven Scheduling

Dynamic scheduling of tasks that operate over horizontal partitions called "morsels" distributed across cores.

- One worker per core.

- One morsel per task.

- Pull-based task assignment.

- Round-robin data placement.

Supports parallel, NUMA-aware operator implementations.

###### Hyper - Architecture

No separate dispatcher thread.

The workers perform cooperative scheduling for each query plan using a single task queue.

- Each worker tries to select tasks that will execute on morsels that are local to it.

- If there are no local tasks, then the worker just pulls the next task from the global work queue.

###### Hyper - Data Partitioning

<img title="" src="images/hyper_data_partitioning.png" alt="hyper" data-align="inline">

###### Hyper - Execution Example

- Morsel read their local data to build a hash table:
  
  <img title="" src="images/hyper_morsel_read_local_data.png" alt="hyper" data-align="inline">

- Morsel write it to Buffer Memory:
  
  <img title="" src="images/hyper_morsel_write_to_buffer.png" alt="hyper" data-align="inline">

- Wait for all tasks are done:
  
  <img title="" src="images/hyper_morsel_wait_for_all_tasks_done.png" alt="hyper" data-align="inline">

- Execute the next one:
  
  <img title="" src="images/hyper_morsel_execute_the_next_one.png" alt="hyper" data-align="inline">

- Say this one guy finishes:
  
  <img title="" src="images/hyper_morsel_say_this_one_guy_finished.png" alt="hyper" data-align="inline">

- then it goes up this Global task queue and it can pick another task run, if there's no task that prefers or wants to run on this new region, we pulled out, poach it, steal it and actually run it:
  
  <img title="" src="images/hyper_morsel_run_another_task_when_done.png" alt="hyper" data-align="inline">

Because there is only one worker per core and one morsel per task, HyPer must use work stealing because otherwise threads could sit idle waiting for stragglers.

The DBMS uses a lock-free hash table to maintain the global work queues.

Tasks can be have different execution costs per tuple.

- Example: Simple Selection vs. String Matching.

HyPer also has no notion of execution priorities.

- All query tasks are executed with the same.

- Short-running queries get blocked behind long-running queries.

##### UMBRA - Morsel Scheduling 2.0

Tasks are not created statically at runtime.

Each task may contain multiple morsels.

Modern implementation of stride scheduling.

Priority decay.

###### UMBRA - Stride Scheduling

Each worker maintains its own thread-local meta-data about the available tasks to execute.

- **Active Slots:** Which entries in the global slot array have active task sets available.

- **Change Mask:** Indicates when a new task set is added to the global slot array.

- **Return Mask:** Indicates when a worker completes a task set.

Workers perform CaS updates to TLS meta-data to broadcast changes.

<img title="" src="images/umbra_ex1.png" alt="umbra" data-align="center">

- When a worker completes the last morsel for a query's active task set:

<img src="images/umbra_sub_task_done.png" title="" alt="umbra" data-align="center">

- It inserts the next task set into global slot array:

<img src="images/umbra_inserts_next_task_set.png" title="" alt="umbra" data-align="center">

- And updates the return mask for all workers:
  
  <img src="images/umbra_update_the_return_mask.png" title="" alt="umbra" data-align="center">

##### SAP HANA - NUMA-Aware Scheduler

Pull-based scheduling with multiple worker threads that are organized into groups (pools).

- Each CPU can have multiple groups.

- Each group has a soft and hard priority queue.

Uses a seperate "watchdog" thread to check whether groups are saturated and can reassign tasks dynamically.

##### SAP HANA - Thread Groups

DBMS maintain `soft` and `hard` priority task queues for each thread group.

- Threads can steal tasks from other group's soft queues.

Four different pools of thread per group:

- **Working:** Active executing a task.

- **Inactive:** Blocked inside of the kernel due to a latch.

- **Free:** Sleeps for a little, wake up to see whether there is a new task to execute.

- **Parked:** Waiting for a task (like a free thread) but blocked in the kernel until the watchdog thread wakes it up.

##### SAP HANA - NUMA-Aware Scheduler

Dynamically adjust thread pinning based on whether a task is CPU or memory bound.

- Allow more cross-region stealing if DBMS is CPU-bound.

SAP found that work stealing was not as beneficial for systems with a larger number of sockets.

- HyPer (2-4 sockets) vs. HANA (64 sockets)

Using thread groups allows cores to execute other tasks instead of just only queries.

Example:

<img title="" src="images/hana_example.png" alt="hana" data-align="inline">

In the above example, the first tasks we want to execute go to the `Soft Queue`. And the garbage collector that we wanted this only run on local memory, we'll put these to the `Hard Queue` and this will prevent anybody from stealing them.

Now our working threads can wake up and they go pull things out of `Soft Queue` and can start running:

<img title="" src="images/hana_working_threads_wake_up.png" alt="hana" data-align="inline">

Other threads are waiting for some latch.

The free threads will wake up and it'll look in these queues and finds something in the `Hard Queue`:

<img title="" src="images/hana_free_threads_wake_up.png" alt="hana" data-align="inline">

So then it changes into the working pool and it can execute:

<img title="" src="images/hana_free_thread_execute.png" alt="hana" data-align="inline">

##### SQL Server - SQLOS

**SQLOS** is a user-mode NUMA-aware OS layer that runs inside of the DBMS and manages provisioned hardware resources.

- Determines which tasks are scheduled onto which threads.

- Also manages I/O scheduling and higher-level concepts like logical database locks.

Non-preemptive thread scheduling through instrumented DBMS code.

**SQLOS** quantum is 4ms but the scheduler cannot enforce that.

```sql
SELECT * FROM R WHERE R.val = ?
```

Approximate Plan:

```python
for t in R:
    if eval(predicate, tuple, params):
        emit(tuple)
```

DBMS developers must add explicit `yield` calls in various locations in the source code.

```python
last = now()
for tuple in R:
    if now() - last > 4ms:
        yield # this is the different for yield
        last = now()
    if eval(predicate, tuple params):
        emit(tuple)
```

Other Examples:

- ScyllaDB

- FaunaDB

- CoroBase

#### Observation

- If requests arrive at the DBMS faster than it can execute them, then the system becomes overloaded.

- The OS cannot help us here because it does not know what threads are doing:
  
  - CPU Bound: Do nothing
  
  - Memory Bound: OOM

- Easiest DBMS Solution: Crash

##### Flow Control

**Approach #1: Admission Control**

- Abort new requests when the system believes that it will not have enough resources to execute that request.

**Approach #2: Throttling**

- Delay the responses to clients to increase the amount of time between requests.

- This assumes a synchronous submission scheme.

---
