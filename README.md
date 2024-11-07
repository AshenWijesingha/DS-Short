# Lec 1

### 1. **Background of Relational and Object-Oriented Models**
   - **Relational Model (1970s)**: Known for tables, keys, and SQL, it’s ideal for structured, business-related data but lacks support for multimedia or complex data structures.
   - **Object-Oriented Model (1980s)**: Introduced complex object support, inheritance, and object IDs to address relational model limitations. Although it gained attention, it had limited adoption.

### 2. **Object-Relational Database Systems (ORDB)**
   - Emerged in the 1990s as a bridge between relational and object-oriented models. ORDB systems, like Oracle and IBM, expanded SQL standards to include object-oriented features (e.g., SQL:99).

### 3. **Limitations of Relational Databases**
   - Relational databases struggle with set-valued attributes (e.g., a person’s multiple phone numbers). These structures require multiple joins, which can be inefficient.

   **Example**:
   - A relational table for a person’s phone numbers may need to be split into separate tables (e.g., `Person`, `Phone`) and joined together, complicating queries for retrieving all numbers for a person.

### 4. **Object Extensions in SQL**
   - ORDB adds object-oriented features, including User-Defined Types (UDTs), which allow more complex data structures and methods to be associated with data.

   **Example**:
   ```sql
   CREATE TYPE BarType AS OBJECT (
       name CHAR(20),
       addr CHAR(20)
   );
   CREATE TABLE bars OF BarType;
   ```
   Here, `BarType` is a custom object type for storing bar information.

### 5. **Object References and Relationships**
   - ORDB enables references (using `REF`), similar to pointers, to link related objects across tables. These references simplify querying hierarchical or interrelated data.

   **Example**:
   ```sql
   CREATE TYPE MenuType AS OBJECT (
       bar REF BarType,
       price FLOAT
   );
   CREATE TABLE Sells OF MenuType;
   ```
   The `bar` attribute is a reference to `BarType` objects, enabling straightforward access to related data.

### 6. **Using REF and DEREF**
   - The `REF` keyword provides a reference to an object, while `DEREF` (or dot notation) allows access to referenced objects’ attributes.
   - This enables easy querying of linked attributes (e.g., accessing a bar’s name through a menu entry).

   **Example**:
   ```sql
   SELECT s.beer.name
   FROM Sells s
   WHERE s.bar.name = 'Joe''s Bar';
   ```
   This retrieves beer names for a specific bar by dereferencing `bar`.

### 7. **Handling Nulls and Constraints**
   - Like relational models, ORDBs allow nulls in attributes, objects, and collections, offering flexibility for optional data. Constraints can also be applied to maintain data integrity.

### 8. **Summary of ORDB Concepts**
   - ORDBs extend relational models with object capabilities, supporting user-defined types, object references, inheritance, and constraints, making them well-suited for complex applications requiring advanced data structures and relationships.

This lecture emphasizes the advantages of ORDB in handling complex, real-world data and offers SQL syntax examples for defining and manipulating these data structures in Oracle.

# Lecture 2

### 1. **Introduction to Object-Relational Database Systems (ORDB)**
   ORDB combines the relational database (RDB) model with object-oriented (OO) concepts. This fusion supports more complex data types and behaviors, extending relational models to handle real-world applications beyond simple data. 

   - **Relational Model Limitations**: It uses flat tables which make it difficult to handle complex data (e.g., multimedia).
   - **Object-Oriented Extensions**: Added capabilities include inheritance, encapsulation, and user-defined types (UDTs).

### 2. **Object Types and User-Defined Types (UDTs)**
   UDTs allow users to define complex types, enabling structured data storage. In ORDB, UDTs are defined similarly to classes in object-oriented programming, and they can include methods.

   **Example**:
   ```sql
   CREATE TYPE BarType AS OBJECT (name CHAR(20), addr CHAR(20));
   CREATE TABLE bars OF BarType;
   ```

   - **Methods**: Functions or procedures can be added to UDTs to perform operations. Methods are defined in `CREATE TYPE BODY` statements.

### 3. **Collection Types (VARRAYs and Nested Tables)**
   ORDB allows modeling one-to-many relationships using collections:
   - **VARRAYs**: Arrays of a defined maximum size. Useful for storing ordered data like a list of prices.
   
     **Example**:
     ```sql
     CREATE TYPE price_arr AS VARRAY(10) OF NUMBER(12,2);
     ```
   
   - **Nested Tables**: Similar to multi-row tables within a row, allowing storage of a collection of objects in a single column.

     **Example**:
     ```sql
     CREATE TYPE BeerType AS OBJECT (name CHAR(20), kind CHAR(10));
     CREATE TYPE BeerTableType AS TABLE OF BeerType;
     CREATE TABLE Manfs (name CHAR(30), addr CHAR(50), beers BeerTableType) NESTED TABLE beers STORE AS beer_table;
     ```

### 4. **Encapsulation and Methods in ORDB**
   ORDB allows encapsulation, where each UDT can have methods (member functions) to operate on its data.

   **Example**:
   ```sql
   CREATE TYPE MenuType AS OBJECT (
       bar REF BarType, price FLOAT,
       MEMBER FUNCTION priceInYen(rate IN FLOAT) RETURN FLOAT
   );
   CREATE TYPE BODY MenuType AS 
       MEMBER FUNCTION priceInYen(rate FLOAT) RETURN FLOAT IS 
       BEGIN RETURN rate * SELF.price; END; 
   END;
   ```

   The `priceInYen` method calculates the price of a menu item in Yen based on a conversion rate.

### 5. **Object Comparison (Map and Order Methods)**
   - **Map Methods**: Used for ordering objects by mapping them to a scalar value (e.g., area for a rectangle).
   
     **Example**:
     ```sql
     CREATE TYPE Rectangle_type AS OBJECT (
         length NUMBER, width NUMBER, 
         MAP MEMBER FUNCTION area RETURN NUMBER
     );
     ```

   - **Order Methods**: Allow direct comparison of objects by defining custom logic to sort objects.

     **Example**:
     ```sql
     CREATE TYPE Customer_typ AS OBJECT (
         id NUMBER, name VARCHAR2(20),
         ORDER MEMBER FUNCTION match (c Customer_typ) RETURN INTEGER
     );
     ```

### 6. **Inheritance in ORDB**
   ORDBs support inheritance, allowing the creation of subtypes from a base type. This helps model real-world hierarchies like employees and managers.
   
   **Example**:
   ```sql
   CREATE TYPE Person_type AS OBJECT (pid NUMBER, name VARCHAR2(30)) NOT FINAL;
   CREATE TYPE Student_type UNDER Person_type (deptid NUMBER, major VARCHAR2(30));
   ```
   Here, `Student_type` inherits all attributes from `Person_type`.

### 7. **Multilevel Collection Types**
   ORDBs support complex structures with nested collections, like planets and their satellites within a star system.
   
   **Example**:
   ```sql
   CREATE TYPE sat_t AS OBJECT (name VARCHAR2(20), orbit NUMBER);
   CREATE TYPE sat_ntt AS TABLE OF sat_t;
   CREATE TYPE planet_t AS OBJECT (name VARCHAR2(20), mass NUMBER, satellites sat_ntt);
   ```

   Multilevel collections provide a hierarchical organization of data, enabling storage and queries across multiple levels.

### 8. **Instance Management and Queries on Collections**
   - **Creating and Retrieving Collection Instances**: Use type constructors to insert data into collection types.
   - **DML Operations on Collections**: Includes inserting, updating, and deleting rows within nested tables and VARRAYs.
   - **Unnesting Collections**: Collections can be "flattened" in queries using the `TABLE()` function.

### 9. **Final and Non-Instantiable Types and Methods**
   - **NOT FINAL** types allow inheritance, while **FINAL** types prevent further inheritance.
   - **Non-Instantiable Types/Methods**: Defined for abstract purposes; instances cannot be created directly. Useful for defining a hierarchy where only subtypes are instantiated.

### 10. **Overloading and Overriding Methods**
   - **Overloading**: Allows multiple methods with the same name but different parameters.
   - **Overriding**: Subtypes can provide specific implementations of methods defined in supertypes.

   **Example of Overloading**:
   ```sql
   CREATE TYPE MyType AS OBJECT (MEMBER FUNCTION fun(x NUMBER), ...);
   CREATE TYPE MySubType UNDER MyType (MEMBER FUNCTION fun(x DATE), ...);
   ```

# Lecture 3

This lecture focuses on **file organization** methods and **indexing** techniques used in database systems to store, retrieve, and manage data efficiently. Here’s an overview:

---

### 1. **File Organization**
   File organization refers to the structure and method used to store records in a file. Common methods include:

   - **Heap File Organization**: Records are stored in random order. This is useful for quick insertion at the end but requires scanning the whole file for searches.
   - **Sequential File Organization**: Records are sorted, allowing efficient range queries but making insertion slower due to reordering.
   - **Hashed File Organization**: Uses a hashing function to distribute records into “buckets.” This method is effective for equality searches but not for range searches.

   **Example**: If a database stores customer records, a sequential file could store records sorted by customer ID, allowing faster range queries (e.g., IDs 100 to 200).

---

### 2. **Indexes**
   Indexes are additional structures that provide fast access to records based on certain fields known as **search keys**.

   - **Clustered Index**: Orders the data records to match the index, allowing efficient range queries. Only one clustered index can exist per table.
   - **Unclustered Index**: Separates the data from the index, which may result in slower access for range queries.

   **Example**: An index on a library’s catalog allows quick lookups by title or author, where searching without an index would require scanning each record.

---

### 3. **Data Entry Alternatives in Indexing**
   There are three main ways to store entries in indexes:
   
   - **Alternative 1**: Directly storing data records.
   - **Alternative 2**: Storing the search key and a reference to the data record.
   - **Alternative 3**: Storing the search key with a list of references.

---

### 4. **B+ Trees**
   B+ Trees are commonly used for indexing as they are balanced and provide efficient search, insertion, and deletion.

   - **Structure**: Each node in a B+ Tree has keys and pointers, directing searches to the correct subtree or leaf node.
   - **Advantages**: Allows for both equality and range queries. For example, finding records between two values is efficient in a B+ Tree due to its sorted structure.
   - **Example Operations**:
     - **Insertion**: New keys are added to the correct leaf node. If the node overflows, it splits, potentially increasing tree height.
     - **Deletion**: Keys are removed, and if a node underflows, redistribution or merging occurs.

---

### 5. **Hashing**
   Hashing provides a method to directly access a record based on a hash function that maps the search key to a specific bucket.

   - **Static Hashing**: The number of buckets is fixed, which can lead to overflow if a bucket fills up, resulting in slower searches.
   - **Extendible Hashing**: Dynamically increases the number of buckets by splitting the directory when needed, which optimizes space usage.

   **Example**: In a database of student records, a static hash function could map student IDs to specific buckets, while extendible hashing would handle growing data more flexibly.

---

### **Key Takeaways**
- File organization and indexing optimize data storage and retrieval.
- B+ Trees are optimal for queries requiring range scans, while hashing is better for direct access via specific keys.
- Efficient indexing structures such as clustered/unclustered indexes and B+ Trees play a critical role in improving query performance.

# Lecture 4

Let's explore query processing concepts with examples that include mock data tables for clarity.

---

### Key Concepts and Stages of Query Processing

1. **Parsing and Translation**:
   - This stage validates the query's syntax and semantics and converts it into an internal form such as a query tree or graph (usually in relational algebra).
   - **Example Query**: 
     ```sql
     SELECT sname
     FROM Sailors S, Reservations R
     WHERE S.sid = R.sid AND R.bid = 'B2';
     ```
     This query selects sailor names (`sname`) for sailors who have a reservation with a specific `bid`.

     **Relational Algebra Representation**:
     \[
     \pi_{\text{sname}}(\sigma_{R.bid = 'B2'}(S \bowtie R))
     \]

2. **Query Optimization**:
   - During optimization, the DBMS chooses the most efficient way to execute a query. This could involve using indices or rearranging joins to minimize disk I/O.
   - **Example**:
     Suppose `Sailors` has 1000 rows, and `Reservations` has 5000 rows. Rather than calculating a full join and filtering afterwards, the optimizer would filter `Reservations` where `bid = 'B2'` first, reducing the number of rows before the join.

3. **Evaluation**:
   - The query is executed according to the chosen plan, using different algorithms like nested loops, sort-merge, or hash joins.

---

### Example Tables

Consider the following example tables:

#### Table: Sailors
| sid | sname  | rating | age  |
|-----|--------|--------|------|
| 22  | Dustin | 7      | 45.0 |
| 28  | Yuppy  | 9      | 35.0 |
| 31  | Lubber | 8      | 55.5 |
| 44  | Guppy  | 5      | 35.0 |
| 58  | Rusty  | 10     | 35.0 |

#### Table: Reservations
| sid | bid | day       | rname |
|-----|-----|-----------|-------|
| 28  | 103 | 12/4/96   | Guppy |
| 28  | 103 | 11/3/96   | Yuppy |
| 31  | 101 | 10/10/96  | Dustin|
| 31  | 102 | 10/12/96  | Lubber|
| 31  | 101 | 10/11/96  | Lubber|
| 58  | 103 | 11/12/96  | Dustin|

---

### Example of Query Optimization Techniques

1. **Heuristic Optimization**:
   - The optimizer breaks down complex queries into simpler parts and orders them for efficiency. For instance:
     - **Cascade of Selections**: Selections can be split and applied one after another.
     - **Example**: Instead of computing `Sailors × Reservations` (a Cartesian product) and filtering, the optimizer uses conditions early to reduce intermediate results.

2. **Cost Estimation**:
   - Each operation is assessed for cost based on factors like disk I/O and memory use, and the plan with the lowest estimated cost is chosen.

3. **Equivalence Rules**:
   - Transformations like commutative selection allow the optimizer to rearrange query operations to avoid expensive operations.

### Examples of Join Algorithms

1. **Nested Loops Join**:
   - In a nested loops join, each tuple in one relation is compared with each tuple in the other, suitable for smaller datasets.
   - **Example Execution**:
     - For each `Sailors` row, scan through `Reservations` rows. If `sid` matches, include the row in the result.
   - **Cost**: The cost grows with the size of both tables, making it costly for large datasets.

2. **Sort-Merge Join**:
   - Both tables are sorted by the join column (`sid`) and then merged to find matches.
   - **Example**:
     - Sort `Sailors` and `Reservations` on `sid`. Scan through both tables to find rows where `sid` matches.
   - **Cost**: Sorting and merging reduce the complexity, making it more efficient than nested loops.

3. **Block Nested Loops Join**:
   - The outer table is processed in blocks, and for each block, the inner table is scanned. This reduces disk I/O by loading more data at once.
   - **Example Execution**:
     - Load 10 pages of `Sailors` and scan each `Reservations` page for matching `sid` values.
   - **Cost**: Reduced I/O due to fewer scans, especially if the smaller table is chosen as the outer loop.

4. **Index Nested Loops Join**:
   - If there is an index on the join column of one table, it is used to directly locate matching rows.
   - **Example**: 
     - Assume an index exists on `Reservations.sid`. For each `Sailors.sid`, use the index to quickly find matching rows in `Reservations`.
   - **Cost**: Cost depends on index efficiency and clustering. If clustered, fewer I/Os are required.

5. **Sort-Merge Join with Example Calculation**:
   - **Tables**: `Sailors` (1000 pages) and `Reservations` (500 pages).
   - **Sort Phase**: Sort both tables on `sid`.
   - **Merge Phase**: Use two pointers to scan sorted `Sailors` and `Reservations` to merge matching rows based on `sid`.

   **Example Cost**:
   - Sorting both tables: `O(M log M) + O(N log N)`.
   - Merging: `M + N` I/Os.  

---

### Size Estimation and Reduction Factors

- **Reduction Factors** are used to estimate the size of results. For example, if 10% of `Reservations` match `bid = 'B2'`, the result will be approximately 10% of the total `Reservations` rows.
- **Statistics and Catalogs** store these estimates and relation statistics, helping optimize query plans based on real data characteristics.

---

### Summary

Query processing optimizes query execution by reducing unnecessary calculations and utilizing effective algorithms. DBMSs balance between multiple plans, ultimately selecting the one with the lowest estimated cost based on the query workload and available resources.

# Lecture 5

This lecture focuses on *Physical Database Design and Tuning*, emphasizing how to optimize database performance by designing physical structures and selecting efficient configurations.

### Key Concepts:
1. **Physical Database Design and Database Tuning**:
   - **Goal**: Improve database performance on specific workloads, including important queries, updates, and expected performance goals.
   - Physical Database Design occurs during development with estimated workloads, while Database Tuning happens in actual use.

2. **Index Selection**:
   - Key aspects of index selection include knowing DBMS capabilities, available indexing techniques, and understanding the query optimizer's workings.
   - Indexes impact update operations; they can accelerate retrieval but may slow down updates due to added maintenance.
  
### Index Selection Guidelines:
- **G1: Index Necessity** - Only index relations that benefit from frequent queries or updates (e.g., an `Emp` table for employee data).
- **G2: Choice of Search Key** - Attributes in `WHERE` clauses often become search keys. Hash indexes suit equality selections (`e.g., e.empno = 123`), while B+ Tree indexes suit range queries (e.g., `e.salary > 20000`).
- **G3: Multi-Attribute Search Keys** - When `WHERE` clauses include multiple attributes, using multi-attribute keys may improve performance.
- **G4: Clustering** - Useful for range queries, clustering should be applied carefully as only one index per relation can be clustered. It's beneficial when many tuples are retrieved.
- **G5: Hash vs. Tree Index** - B+ Tree indexes support both equality and range queries, while hash indexes perform best for equality searches without range queries.
- **G6: Index Maintenance Cost** - For frequent updates, avoid indexes that heavily impact update performance.
- **G7: Join Conditions** - For joins, a hash index on the inner table helps in index nested loops, while a clustered B+ tree index benefits sort-merge joins.

### Examples:
- **Example 1**: Query for employee names and department manager details.
  - A hash index on `Dept<dname>` speeds up retrieval when filtering by department name.
  - A clustered index on `Emp<dno>` supports index nested loops for join queries, reducing disk I/O costs.

- **Example 2**: Query for employee names and department names based on salary and hobby.
  - Here, hash indexing on `Emp<hobby>` may be more efficient for equality conditions, while a B+ Tree on `Emp<salary>` supports range conditions.

### Index-Only Plans:
- Certain queries can be answered by scanning index records alone, which is highly efficient. For instance, using an index-only scan, a B+ Tree index on `Emp<dno, eid>` may be sufficient for some joins, even without retrieving full tuples.

### Tools for Database Tuning:
- Automated tools, such as SQL Server’s Database Tuning Advisor, help database administrators with index selection and tuning, streamlining the optimization process.

This lecture emphasizes strategic index selection, query optimization, and an understanding of database structures to maximize performance in both database design and tuning stages.

# Lecture 6

The lecture notes cover **Transactions** and **Concurrency Control** in database systems, focusing on the concepts, properties, potential issues, and management techniques to ensure reliable and consistent transaction processing in a database.

### 1. **Transactions and Their Properties (ACID)**

A **transaction** is a sequence of database operations (reads and writes) viewed as a single unit by the DBMS. For a transaction to be valid, it must meet the **ACID** properties:

- **Atomicity**: Ensures a transaction completes fully or not at all. For example, if transferring money between accounts, either the entire transfer happens, or none of it does.
- **Consistency**: Transactions must lead from one valid database state to another. For instance, a transfer between accounts should maintain the total funds in both accounts before and after the transaction.
- **Isolation**: Concurrent transactions should not interfere, appearing as if they execute sequentially. For example, two users accessing a bank account do not affect each other’s transactions.
- **Durability**: Once a transaction is committed, it remains so even in the event of a crash. This is achieved by using logs that can restore committed transactions.

### 2. **Schedules and Concurrency**

A **schedule** is the ordered list of actions (reads, writes, commits, aborts) of multiple transactions. Concurrency allows overlapping actions of different transactions to improve performance. 

#### Types of Schedules:
- **Serial Schedule**: Each transaction completes before the next starts; simple but inefficient.
- **Equivalent Schedule**: Two schedules are equivalent if they result in the same database state.
- **Serializable Schedule**: A non-serial schedule that provides the same outcome as a serial one, ensuring consistency.

### 3. **Anomalies in Interleaved Execution**

Concurrent transactions can cause **anomalies**:
- **Dirty Reads**: Transaction reads uncommitted data from another transaction (WR conflict).
- **Unrepeatable Reads**: A transaction re-reads data and finds it changed by another (RW conflict).
- **Overwriting Uncommitted Data**: One transaction overwrites uncommitted changes of another (WW conflict).

### 4. **Concurrency Control with Locking**

To handle concurrency safely, **locking mechanisms** control access to data.
  
#### Lock Types:
- **Shared Lock (S)**: Allows multiple transactions to read but not write.
- **Exclusive Lock (X)**: Allows only one transaction to read or write.

#### Protocol - **Strict Two-Phase Locking (Strict 2PL)**:
This protocol ensures serializability by acquiring locks in two phases:
1. **Growing Phase**: A transaction acquires all necessary locks but cannot release any.
2. **Shrinking Phase**: After releasing a lock, no new locks are acquired.

Strict 2PL prevents dirty reads, unrepeatable reads, and lost updates, enabling only serializable schedules.

### 5. **Aborting and Cascading Aborts**

When a transaction **aborts**, any partial changes are undone. However, if another transaction depends on the aborted one, it may also need to abort (**cascading abort**). Strict 2PL prevents cascading aborts by only committing changes when all locks are released.

#### Example:
- **T1** writes `A`.
- **T2** reads `A`.
- If **T1** aborts, **T2** must also abort to maintain consistency.

### 6. **Deadlocks and Prevention**

**Deadlock** occurs when two or more transactions wait for each other to release locks. There are two main strategies to handle this:

- **Deadlock Prevention**: Assigns priorities to transactions based on timestamps. **Wait-Die** and **Wound-Wait** are common policies.
- **Deadlock Detection**: Uses a waits-for graph to detect cycles (deadlocks). If detected, one transaction (the victim) is aborted.

### 7. **Handling Dynamic Databases and Phantoms**

In dynamic databases, new records can create issues like **phantoms**, where transactions assume they have locked all relevant data but miss newly inserted records.

#### Solutions:
- **Predicate Locking**: Locks all records that satisfy a specific condition, but this can be costly.
- **Index Locking**: Ensures consistent views by locking index pages.

### 8. **Multiple-Granularity Locking**

This approach allows locking at different levels (database, table, page, or row) with intention locks (IS, IX) to manage hierarchical data structures efficiently.

### 9. **Transactions in SQL**

SQL supports various **isolation levels** that control the degree of visibility of uncommitted data:
- **Serializable**: Strictest, preventing all anomalies.
- **Repeatable Read**: Prevents dirty and unrepeatable reads but allows phantoms.
- **Read Committed**: Prevents dirty reads but allows others.
- **Read Uncommitted**: Least strict, permitting all anomalies, used only for read-only transactions.

### 10. **Other Concurrency Control Protocols**

Beyond locking, other protocols such as **Optimistic Concurrency Control** (checking for conflicts before committing) and **Timestamp-Based Concurrency Control** offer alternative ways to manage transactions.

---

### Example Tables for Transaction Scenarios

1. **Dirty Read Example**:
   | Transaction T1        | Transaction T2       |
   |-----------------------|----------------------|
   | W(A, 100)             |                      |
   |                       | R(A) (reads 100)     |
   | Abort                 |                      |
   | R(A) shows 50         |                      |

2. **Serializable Schedule Example**:
   | T1                | T2               |
   |-------------------|------------------|
   | R(A)              |                  |
   | W(A)              |                  |
   | Commit            |                  |
   |                   | R(A)             |
   |                   | W(A)             |
   |                   | Commit           |

These tables help illustrate the operations and impact on data across transactions, showcasing how concurrency control mechanisms like locking prevent conflicts.

# Lecture 7

The lecture provides a comprehensive overview of NoSQL databases, focusing on their relevance in managing large volumes of unstructured data and scalability for modern applications. Here’s a complete summary with examples:

### 1. **Current Trends in Data Management**
   - With the growth of *Big Data* from sources like social networks, IoT devices, and web logs, traditional relational databases face limitations.
   - Examples: A single flight by a Boeing airplane generates 640 TB of data, illustrating the need for flexible storage solutions.

### 2. **What is NoSQL?**
   - NoSQL refers to a category of databases designed for distributed data storage without relying on a rigid schema.
   - Key features include flexibility, horizontal scalability, and easy replication. Many NoSQL databases are open source, enhancing accessibility and customization.

### 3. **Relational vs. NoSQL Databases**
   - **RDBMS**: Structured schema, supports complex queries, ACID-compliant for reliability.
   - **NoSQL**: Schema-free, optimized for fast, scalable operations in distributed environments, often using eventual consistency instead of ACID.
   - Example: *MongoDB* allows adding fields dynamically to documents, unlike relational databases that need predefined columns.

### 4. **CAP Theorem and BASE Concept**
   - **CAP Theorem**: NoSQL databases balance *Consistency*, *Availability*, and *Partition Tolerance*, though not all three can be fully achieved simultaneously.
   - **BASE Model**: Focuses on *eventual consistency*, where the system ensures data consistency over time, prioritizing availability over strict consistency.
   - Example: An online retailer may prioritize availability for user data but accept slight delays in synchronization across data centers.

### 5. **Data Models in NoSQL**
   - NoSQL databases vary in data structure, primarily categorized into:
     - **Key-Value Stores**: Simple key-value pairs for efficient lookups (e.g., *Redis*, *DynamoDB*).
     - **Document Stores**: JSON-like documents that can include complex structures (e.g., *MongoDB*).
     - **Column-Family Stores**: Columnar storage optimized for read-heavy applications (e.g., *Cassandra*).
     - **Graph Databases**: Nodes and edges to handle complex relationships (e.g., *Neo4j*).
   - Aggregates, such as collections of related data, are central in aggregate-oriented databases, improving data locality and reducing complexity.

### 6. **Scalability and Distribution**
   - NoSQL databases handle data growth through:
     - **Vertical Scalability**: Expanding storage on a single powerful machine, which is costlier and has limits.
     - **Horizontal Scalability**: Distributing data across multiple machines, commonly used by NoSQL systems.
   - **Replication Models**:
     - *Master-Slave*: One node handles updates (master), while others read data (slaves).
     - *Peer-to-Peer*: All nodes are equal, and data is replicated without a central master.
   - **Sharding**: Divides data across nodes based on criteria like user geography to enhance performance.

### 7. **Types of NoSQL Databases and Use Cases**
   - **Key-Value Stores**: Ideal for session storage, user profiles, and shopping carts.
   - **Document Stores**: Used in content management, e-commerce, and real-time analytics.
   - **Column-Family Stores**: Suitable for applications with heavy write volumes or maintaining counters.
   - **Graph Databases**: Perfect for social networks, recommendation systems, and routing information.

### 8. **Advantages and Drawbacks of NoSQL**
   - **Advantages**: High scalability, flexibility, and rapid development.
   - **Drawbacks**: Lack of standardization, limited support for complex querying, and a focus on eventual rather than strict consistency.

By leveraging NoSQL, organizations can efficiently manage large, varied, and dynamically changing datasets. For example, companies like Facebook and Google use NoSQL to store and quickly retrieve vast amounts of unstructured data generated by their services.

# Lecture 8

This lecture note introduces MongoDB, a NoSQL database that uses a document-oriented structure. Here's a summary of the main points, with examples:

### 1. **What is a Document Database?**
   - Document databases store semi-structured data as "documents" instead of using tables with rows and columns, providing flexible schemas.
   - **Example:** JSON-like structures are common in document databases, allowing dynamic fields within each document.

### 2. **Introduction to MongoDB**
   - MongoDB is a scalable, high-performance document database that uses BSON (a binary JSON format) to store data. Developed by MongoDB Inc., it became open-source in 2009.
   - **Example:** Instead of tables, MongoDB organizes data into collections of documents, allowing flexibility in data structure.

### 3. **Why Use MongoDB?**
   - It allows direct storage of application objects, eliminating the need for complex joins and multi-document transactions, as it supports embedded documents and arrays.
   - **Example:** An e-commerce site could store user information with embedded order details in a single document.

### 4. **Best Uses for MongoDB**
   - It is ideal for web applications, real-time analytics, high-speed logging, caching, and applications requiring high scalability.
   - **Example:** Social media platforms often use MongoDB to manage semi-structured user content.

### 5. **MongoDB Basics**
   - MongoDB uses instances that contain databases, collections, and documents with fields. The result of data retrieval is returned as a cursor.
   - **Example:** Documents in a `users` collection may vary in structure, storing different fields for each user.

### Connect to Mongo DB

Here's a sample MongoDB connection string:

```javascript
const { MongoClient } = require('mongodb');

// Replace <username>, <password>, and <dbname> with your MongoDB credentials
const uri = "mongodb+srv://<username>:<password>@cluster0.mongodb.net/<dbname>?retryWrites=true&w=majority";

const client = new MongoClient(uri, { useNewUrlParser: true, useUnifiedTopology: true });

async function connectToMongoDB() {
    try {
        await client.connect();
        console.log("Connected to MongoDB!");
        // Perform database operations here
    } catch (error) {
        console.error("Connection failed", error);
    } finally {
        await client.close();
    }
}

connectToMongoDB();
```

### Explanation:
- **`mongodb+srv://`** specifies a connection to a MongoDB Atlas cluster. Replace it with `mongodb://` if connecting to a local MongoDB instance.
- **`<username>` and `<password>`** are placeholders for your MongoDB credentials.
- **`<dbname>`** is the name of your database.
- **`retryWrites=true&w=majority`** enables automatic retry of write operations and specifies the write concern level.

This example demonstrates how to connect using the official MongoDB Node.js driver. Adjust parameters according to your environment and credentials.

### 6. **CRUD Operations**
   - **Create:** Add new documents using `insert`.
     ```javascript
     db.unicorns.insert({ name: 'Aurora', gender: 'f', weight: 450 });
     ```
   - **Read:** Retrieve documents with `find`.
     ```javascript
     db.unicorns.find({ gender: 'm' });
     ```
   - **Update:** Modify existing documents using `$set`, `$inc`, etc.
     ```javascript
     db.unicorns.update({ name: 'Roooooodles' }, { $set: { weight: 590 } });
     ```
   - **Delete:** Remove documents using `remove`.
     ```javascript
     db.unicorns.remove({ weight: { $gt: 500 } });
     ```

### 7. **Getting Started with MongoDB**
   - MongoDB can be installed locally or accessed via online platforms like MongoDB’s shell at TutorialsPoint.
   - **Example Commands:**
     - `use wonderland`: Creates and switches to the `wonderland` database.
     - `db.getCollectionNames()`: Lists collections in the current database.

### 8. **Advanced Operations**
   - **Projections:** Control fields returned in queries.
     ```javascript
     db.unicorns.find({}, { name: 1, _id: 0 });
     ```
   - **Upserts:** Insert a document if it doesn’t exist during an update operation.
     ```javascript
     db.unicorns.update({ name: 'Walala' }, { $inc: { vampires: 1 } }, { upsert: true });
     ```
   - **Multi-Update:** Enable updates across multiple documents.
     ```javascript
     db.unicorns.update({}, { $set: { vaccinated: true } }, { multi: true });
     ```

### 9. **Example Queries for Practice**
   - Find male unicorns weighing over 700 pounds:
     ```javascript
     db.unicorns.find({ gender: 'm', weight: { $gt: 700 } });
     ```
   - Find unicorns without a `vampire` field:
     ```javascript
     db.unicorns.find({ vampire: { $exists: false } });
     ```
   - Sort unicorns by weight descending:
     ```javascript
     db.unicorns.find().sort({ weight: -1 });
     ```

### References
   - Suggested readings include "The Little MongoDB Book" and MongoDB's CRUD operations documentation.

This overview highlights MongoDB's flexibility and practical applications, particularly for web applications and data models that benefit from semi-structured storage.

# Lecture 9

This lecture on "Introduction to Big Data and Hadoop" provides an overview of Big Data concepts, the Hadoop platform, its architecture, features, history, and ecosystem. Here’s a summary:

### 1. **Introduction to Big Data**
   - **Definition**: Big Data refers to large, complex datasets growing exponentially, which traditional data management tools struggle to process efficiently.
   - **Data Scaling**: Data grows from bits to yottabytes, illustrating the vast scales Big Data can reach.

### 2. **Types of Data**
   - **Structured Data**: Organized data with a defined pattern (e.g., SQL databases).
   - **Unstructured Data**: No clear format or structure, making it harder to process (e.g., audio, video).
   - **Semi-Structured Data**: Contains tags or markup elements (e.g., JSON files), falling between structured and unstructured.

### 3. **The Four V’s of Big Data**
   - **Volume**: The amount of data.
   - **Velocity**: The speed of data generation and processing.
   - **Variety**: Different data types and formats.
   - **Veracity**: The uncertainty or reliability of data.

### 4. **Big Data Technologies**
   - **Hadoop**: An open-source platform that enables the storage, processing, and analysis of massive datasets.
   - **IMC (In-Memory Computing)**: Analyzes data in RAM for faster access.
   - **Big Data Cloud**: Uses cloud services like Amazon and Google for scalable data processing.

### 5. **Introduction to Apache Hadoop**
   - **Definition**: Hadoop is an open-source ecosystem designed to manage large-scale data efficiently by distributing storage and processing across multiple machines.
   - **Key Attributes**:
     - **Redundant and Reliable**: Stores data across multiple nodes.
     - **Batch Processing**: Focuses on batch-oriented data processing.
     - **Runs on Commodity Hardware**: Can operate on affordable hardware setups.

### 6. **Hadoop Architecture**
   - **HDFS (Hadoop Distributed File System)**: The primary storage system that splits data across nodes for reliability.
   - **YARN (Yet Another Resource Negotiator)**: Manages job scheduling and resource allocation.
   - **MapReduce**: The computational framework that processes data in parallel across nodes.
   - **Common Utilities**: Provide additional support for other Hadoop modules.

### 7. **Key Features of Hadoop**
   - **Ease of Programming**: Hadoop abstracts complexities, so programmers don’t manage data location or failure handling.
   - **Scalability**: Allows the addition or removal of nodes without disruption.
   - **Reliability**: By default, it replicates data across nodes (often three) for redundancy.
   - **Linear Processing Capability**: Increases capacity proportionally with the number of nodes.

### 8. **Hadoop History**
   - Created by Doug Cutting, Hadoop was initially part of the Apache Lucene project and named after his son’s toy elephant.

### 9. **Hadoop Ecosystem Components**
   - **HDFS, YARN, MapReduce**: Core components.
   - **Spark**: Enables in-memory data processing for faster performance.
   - **PIG & HIVE**: Allow SQL-like querying of data.
   - **HBase**: A NoSQL database for managing large tables.
   - **Machine Learning Libraries**: Mahout and Spark MLLib offer machine learning capabilities.
   - **Zookeeper**: Manages cluster coordination.
   - **Oozie**: Schedules Hadoop jobs for workflow automation.

### **Example Use Case**
For example, a retailer might use Hadoop to analyze terabytes of sales data (Volume) in real-time (Velocity), from multiple sources like social media and transactions (Variety), while addressing inconsistencies (Veracity) to uncover sales trends and predict demand. 

This lecture highlights Hadoop’s importance in handling Big Data challenges, particularly by offering reliable, scalable, and flexible solutions for large-scale data processing.

# Lecture 10

### Summary of Lecture on Hadoop and MapReduce

#### 1. **Introduction to MapReduce**
   - **Definition**: MapReduce is a programming model used to process large data sets (multi-terabyte) stored across a distributed network of computers using Hadoop Distributed File System (HDFS).
   - **Structure**: MapReduce splits data processing into two main functions:
     - **Map**: Processes and filters data, generating intermediate key-value pairs.
     - **Reduce**: Summarizes the mapped data, combining the results into a smaller set.

#### 2. **Benefits of MapReduce**
   - **Simplicity**: Can be written in multiple languages (Java, C++, Python).
   - **Scalability**: Scales across large clusters, handling petabytes of data.
   - **Speed**: Parallel processing reduces time from days to minutes/hours.
   - **Fault Tolerance**: Automatically recovers from node failures.
   - **Efficient Data Handling**: Data processing occurs on the nodes where data resides, minimizing network load.

#### 3. **MapReduce Phases**
   - **Input Splits**: Divides input data into chunks that each map task processes independently.
   - **Mapping**: Each data split is mapped to produce output values; for example, counting word occurrences in a dataset.
   - **Shuffling**: Consolidates mapping outputs, grouping related data (e.g., combining word counts by word).
   - **Reducing**: Aggregates shuffling results into the final output, creating summarized data (e.g., total word counts).

#### 4. **Input/Output Formats in MapReduce**
   - **InputFormat**: Defines how data is split and read; examples include:
     - **TextInputFormat**: Default format, reads lines from text files.
     - **KeyValueInputFormat**: Parses lines into key-value pairs.
     - **SequenceFileInputFormat**: High-performance binary format.
   - **OutputFormat**: Specifies how output data is written:
     - **TextOutputFormat**: Writes data in key-value text form.
     - **SequenceFileOutputFormat**: Outputs binary files for subsequent MapReduce jobs.
     - **NullOutputFormat**: Ignores input, useful for specific tasks.

#### 5. **How MapReduce Organizes Work**
   - **Entities**:
     - **Job Tracker**: The master controller that manages the execution of jobs.
     - **Task Trackers**: Slaves that execute individual tasks on data nodes.
     - **Name Node**: Stores file location information.
     - **Data Nodes**: Store the actual data in HDFS.
   - **Job Execution**: Job Tracker divides tasks, assigns them to Task Trackers, and monitors progress, ensuring all tasks complete.

#### 6. **Example: WordCount Program**
   - A common example demonstrating MapReduce, the WordCount program:
     - **Map Phase**: Reads input text, splitting each line into words and outputs pairs like ("word", 1).
     - **Shuffle Phase**: Groups identical words.
     - **Reduce Phase**: Sums counts for each word, yielding the total count for each word across the entire input.

#### 7. **MapReduce Flowchart**
   - Illustrates the step-by-step process from data input, mapping, shuffling, reducing, to final output generation, encapsulating the entire MapReduce workflow.

This framework is essential for efficiently processing massive data volumes in parallel, making Hadoop and MapReduce a popular choice in big data analytics.
