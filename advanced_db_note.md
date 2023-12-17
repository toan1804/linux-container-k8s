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


