# Lecture 5: Columnar Databases & Compression


## 1- Database Workloads

### OLTP: Online Transaction Processing
An OLTP workload is characterized by:
- fast, short running operations, simple queries that operate on single entity at a time, and repetitive operations.
- An OLTP workload will typically handle more `writes` than `reads`.

- An example of an OLTP workload is the Amazon storefront. 
  - Users can add things to their cart, they can make purchases, but the actions only affect their account.

- This is usually the kind of application
that people build first.

### OLAP: Online Analytical Processing
An OLAP workload is characterized by:
- long running, complex queries, reads on large portions of the database. 

- In OLAP worklaods, the database system is analyzing and deriving new data from existing data collected on the `OLTP` side.

- An example of an OLAP workload would be Amazon computing the most bought item in Pittsburgh on a day when its raining.

- Usually you execute these workloads on the data you have collected from your `OLTP` application(s).
  
### HTAP: Hybrid Transaction + Analytical Processing
A new type of workload which has become popular recently is HTAP, which is like a combination which tries to do `OLTP` and `OLAP` together on the same database.


### Observation
The relational model does not specify that the DBMS must store all a tuple's attributes together in a single page.

This may not actually be the best layout for some workloads…

---
## 2- Storage Models
*There are different ways to store tuples in pages. We have assumed the n-ary (aka `"row storage"`) storage model so far.*

### N-Ary Storage Model (NSM)
In the `n-ary` storage model:
- the DBMS stores all of the attributes for a single tuple `contiguously` in a single page. 

- This approach is ideal for `OLTP workloads` where requests are `insert-heavy` and transactions tend to operate only an individual entity. 
  - It is ideal because it takes only one fetch to be able to get all of the attributes for a single tuple.

![](/assets/images/cmu/db/intro-course/nsm-disk-page.png)

#### Advantages:
- Fast inserts, updates, and deletes.
- Good for queries that need the entire tuple.

#### Disadvantages:
- Not good for scanning large portions of the table and/or a subset of the attributes.

![](/assets/images/cmu/db/intro-course/nsm-useless-data.png)

### Decomposition Storage Model (DSM)

In the decomposition storage model:
- the DBMS stores a single attribute `(column)` for all tuples contiguously in a block of data. Thus, it is also known as a `“column store.”` 

![](/assets/images/cmu/db/intro-course/dsm-column-store.png)

- This model is ideal for `OLAP workloads` with many `read-only queries` that perform large scans over a `subset of the table’s attributes`.

![](/assets/images/cmu/db/intro-course/dsm-disk-page.png)

#### Advantages:
- Reduces the amount of I/O wasted because the DBMS only reads the data that it needs for that query.
- Better query processing and data compression

#### Disadvantages:
- Slow for point queries, inserts, updates, and deletes because of tuple splitting/stitching.

#### Tuple Identification
To put the tuples back together when using a column store, there are two common approaches:

- Choice #1: **Fixed-length Offsets**
  - The most commonly used
  - Each value is the same length for an attribute.
    - Here, you know that the value in a given column will match to another value in another column at the same offset, they will correspond to the same tuple.
    - Therefore, every single value within the column will have to be the same length.

- Choice #2: **Embedded Tuple Ids**
  - less common approach
  - Each value is stored with its tuple id in a column.
    - Here, for every attribute in the columns, the DBMS stores a tuple id (ex: a primary key) with it. 
    - The system then would also store a mapping to tell it how to jump to every attribute that has that id. 
    - Note that this method has a large storage overhead because it needs to store a tuple id for every attribute entry.

![](/assets/images/cmu/db/intro-course/tuple-identification.png)

### Observation

- I/O is the main bottleneck if the DBMS fetches data from disk during query execution.
- The DBMS can `compress` pages to increase the utility of the data moved per I/O operation.

- Key trade-off is speed vs. compression ratio
  - Compressing the database reduces DRAM requirements.
  - It may decrease CPU costs during query execution.

---
## 3- Database Compression
Compression is widely used in disk-based DBMSs. Because disk I/O is (almost) always the main bottleneck.

Thus, compression in these systems improve performance, especially in `read-only` analytical workloads.

The DBMS can fetch more useful tuples if they have been compressed beforehand at the cost of greater computational overhead for compression and decompression.

**In-memory DBMSs** more complicated since they do not have to fetch data from disk to execute a query.
- Memory is much faster than disks, but compressing the database reduces DRAM requirements and processing. 

- They have to strike a balance between speed vs. compression ratio.
- Compressing the database reduces DRAM requirements. It may decrease CPU costs during query execution.

### Real-world Data Characteristics
If data sets are completely random bits, there would be no ways to perform compression. However, there
are key properties of real-world data sets that are amenable to compression:

- Data sets tend to have highly `skewed` distributions for attribute values.
  - **Example:** Zipfian distribution of the Brown Corpus
- Data sets tend to have high `correlation` between attributes of the same tuple.
  - **Example:** Zip Code to City, Order Date to Ship Date.


### Comperession Goals
We want a database compression scheme to have the following properties:
#### 1. Must produce fixed-length values.
- The only exception is var-length data stored in separate pools.
- This because the DBMS should follow word-alignment and be able to access data using offsets.

#### 2. Allow the DBMS to postpone decompression as long as possible during query execution 
- known as `late materialization`.

#### 3. Must be a lossless scheme.
- because people do not like losing data. 
- Any kind of lossy compression has to be performed at the application level.

### Compression Granularity
Before adding compression to the DBMS, we need to decide what kind of data we want to compress.

This decision determines compression schemes are available. 

There are four levels of compression granularity:
- **Block Level:** Compress a block of tuples for the same table.
- **Tuple Level:** Compress the contents of the entire tuple `(NSM only)`.
- **Attribute Level:** Compress a single attribute value within one tuple. 
  - Can target multiple attributes for the same tuple.
- **Columnar Level:** Compress multiple values for one or more attributes stored for multiple tuples `(DSM only)`. 
  - This allows for more complicated compression schemes.

---
## 4- Naive Compression
The DBMS compresses data using a general purpose algorithm (e.g., `gzip`, `LZO`, `LZ4`, `Snappy`, `Brotli`, `Oracle OZIP`, `Zstd`). 

Although there are several compression algorithms that the DBMS could use, engineers often choose ones that often provides `lower compression ratio` in exchange for `faster compress/decompress`.

An example of using naive compression is in **MySQL InnoDB**.
The DBMS compresses disk pages, pad them to a power of `two KBs` and stored them into the buffer pool. 

However, every time the DBMS tries to read data, the compressed data in the buffer pool has to be decompressed.

Since accessing data requires decompression of compressed data, this limits the scope of the compression scheme. 

If the goal is to compress the entire table into one giant block, using naive compression schemes would be impossible since the whole table needs to be compressed/decompressed for every access. 

Therefore, for **MySQL**, it breaks the table into smaller chunks since the compression scope is limited.

![mysql innodb compression](/assets/images/cmu/db/intro-course/mysql-innodb-compression.png)

Another problem is that these naive schemes also do not consider the high-level meaning or semantics of the
data. 

The algorithm is oblivious to neither the structure of the data, nor how the query is planning to access the data. 

Therefore, this gives away the opportunity to utilize `late materialization`, since the DBMS will not be able to tell when it will be able to delay the decompression of data.


### Observation
Ideally, we want the DBMS to operate on compressed data without decompressing it first.

![magical database](/assets/images/cmu/db/intro-course/magic-database.png)

---
## 5- Columnar Compression

### 1. Run-Length Encoding (RLE)
RLE compresses runs of the same value in a single column into `triplets`:
- The value of the attribute
- The start position in the column segment
- The number of elements in the run
The DBMS should sort the columns intelligently beforehand to maximize compression opportunities.
This clusters duplicate attributes and thereby increasing compression ratio.

![run length encoding](/assets/images/cmu/db/intro-course/run-length-encoding.png)

### 2. Bit-Packing Encoding
When values for an attribute are always less than the value’s declared largest size, store them as smaller data type.

![bit packing encoding](/assets/images/cmu/db/intro-course/bit-packing-encoding.png)

### 3. Mostly Encoding
Bit-packing variant that uses a special marker to indicate when a value exceeds largest size and then maintain a look-up table to store them.

![mostly encoding](/assets/images/cmu/db/intro-course/mostly-encoding.png)

### 4. Bitmap Encoding
The DBMS stores a separate bitmap for each unique value for a particular attribute where an offset in the vector corresponds to a tuple.

- The ith position in the bitmap corresponds to the ith tuple in the table to indicate whether that value is present or not. 

- The bitmap is typically segmented into chunks to avoid allocating large blocks of contiguous memory.
- This approach is only practical if the value cardinality is low, since the size of the bitmap is linear to the cardinality of the attribute value. 

- If the cardinalty of the value is high, then the bitmaps can become larger than the original data set.

![bitmap encoding](/assets/images/cmu/db/intro-course/bitmap-encoding.png)

### 5. Delta Encoding
Instead of storing exact values, record the difference between values that follow each other in the same column. 

- The base value can be stored in-line or in a separate look-up table. 
- We can also use `RLE` on the stored deltas to get even better compression ratios.

![delta encoding](/assets/images/cmu/db/intro-course/delta-encoding.png)

### 6. Incremental Encoding
This is a type of `delta encoding` avoids duplicating common prefixes/suffixes between consecutive tuples. This works best with sorted data.

![incremental encoding](/assets/images/cmu/db/intro-course/incremental-encoding.png)

### 7. Dictionary Compression
- The most common database compression scheme. 
- The DBMS replaces frequent patterns in values with smaller codes. 
- It then stores only these codes and a data structure (i.e., dictionary) that maps these codes to their original value. 
- A dictionary compression scheme needs to support fast encoding/decoding, as well as range queries.

![dictionary encoding](/assets/images/cmu/db/intro-course/dictionary-encoding.png)

#### Encoding and Decoding:
the dictionary needs to decide how to encodes (convert uncompressed value into its compressed form)/decodes (convert compressed value back into its original form) data. 
- It is not possible to use hash functions.
- The encoded values also need to support sorting in the same order as original values. 
- This ensures that results returned for compressed queries run on compressed data are consistent with uncompressed queries run on original data.
- This order-preserving property allows operations to be performed directly on the codes.

![order preserving encoding](/assets/images/cmu/db/intro-course/order-preserving-encoding.png)