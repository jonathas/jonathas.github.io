---
layout: post
title: "Optimizing Data Access: A Deep Dive into Database Indexes"
excerpt: ""
tags: [database, indexes, sql, mysql, postgres]
date: 2023-11-23T17:30:08+01:00
comments: true
image:
  feature: posts/database-indexes/cover-database.jpg
  credit: PngTree
  creditlink: https://pngtree.com/free-backgrounds
---

When talking about database optimization, the efficiency of retrieving information is crucial. One key player in this quest for efficiency is the database index. Let's dive into the world of database indexes and uncover what happens under the hood.

## In this article

- What are Database Indexes?
- Types of Indexes
- Trade-offs and Considerations
- Practical examples
  - Inserting some data
  - Examples of index usage
- What should be considered when improving index usage

## What are Database Indexes?

At its core, a database index is a data structure that provides a faster way to locate rows in a table based on the values in one or more columns. Imagine a well-organized library where books are arranged by genres or authors. This way or organizing them significantly accelerates the process of finding a specific book. In a similar way, a database index works as a roadmap, guiding the database engine to the relevant rows in a table.

Most database indexes utilize a B-tree (balanced tree) structure. This structure allows for efficient insertion, deletion, and search operations. Each entry in the index is a key-value pair, with the key representing the indexed column(s) and the value pointing to the corresponding row in the table.

## Types of Indexes

Database indexes can be created manually by administrators or automatically by the database management system (DBMS). Indexes can be applied to one or more columns, and there are various types to cater to different needs:

- Unique Indexes: Ensure the uniqueness of values in the indexed columns, preventing duplicate entries.
- Clustered Indexes: Determine the physical order of data rows in the table based on the index, influencing data storage layout.
- Non-clustered Indexes: Create a separate structure for the index, leaving the data rows unchanged.

When a query involving the indexed column(s) is executed, the database engine utilizes the index to quickly locate the relevant rows. This process reduces the need to scan the entire table, resulting in a significant reduction in disk I/O operations. B-tree indexes, often sorted, enable efficient range queries, making them particularly powerful in enhancing read operation performance.

## Trade-offs and Considerations

While database indexes offer immense benefits in terms of query performance, they come with trade-offs. Indexes consume additional disk space, and their maintenance can impact the performance of write operations such as inserts, updates, and deletes. Careful consideration is needed when deciding which columns to index, taking into account the selectivity of the index and the specific needs of the database.

Updating or deleting data in indexed columns requires corresponding updates to the index to maintain accuracy. Periodic rebuilding or reorganizing of indexes may be necessary to optimize their performance over time.

Indexes with high a large number of unique values are more effective in improving query performance.

Composite indexes involve multiple columns, optimizing queries that filter on multiple criteria.

## Practical examples

Theory yada-yada.. let's see some examples:

Continuing on the library idea that  was mentioned before, we have the following tables:

### Authors

```sql
CREATE TABLE authors (
  id INT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50)
);
```

### Publishers

```sql
CREATE TABLE publishers (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);
```

### Books

The books table includes information about each book, such as title, publication year, ISBN, and the publisher.

```sql
CREATE TABLE books (
  id INT PRIMARY KEY,
  title VARCHAR(255),
  publication_year INT,
  isbn VARCHAR(13),
  publisher_id INT,
  FOREIGN KEY (publisher_id) REFERENCES publishers(id)
);
```

### AuthorsBooks (Association Table)

This table represents the many-to-many relationship between authors and books. It allows multiple authors to be associated with multiple books.

```sql
CREATE TABLE authors_books (
  author_id INT,
  book_id INT,
  PRIMARY KEY (author_id, book_id),
  FOREIGN KEY (author_id) REFERENCES authors(id),
  FOREIGN KEY (book_id) REFERENCES books(id)
);
```

- Each book can have multiple authors (many-to-many relationship).
- Each author can write multiple books (many-to-many relationship).
- Each book is associated with one publisher (many-to-one relationship).

### Inserting some data

```sql
-- Inserting authors
INSERT INTO authors VALUES (1, 'J.K.', 'Rowling');
INSERT INTO authors VALUES (2, 'George R.R.', 'Martin');
INSERT INTO authors VALUES (3, 'Jane', 'Austen');

-- Inserting publishers
INSERT INTO publishers VALUES (1, 'Penguin Books');
INSERT INTO publishers VALUES (2, 'HarperCollins');

-- Inserting books
INSERT INTO books VALUES (1, 'Harry Potter and the Philosopher''s Stone', 1997, '978-0747532743', 1);
INSERT INTO books VALUES (2, 'A Game of Thrones', 1996, '978-0553103540', 2);
INSERT INTO books VALUES (3, 'Pride and Prejudice', 1813, '978-1503290563', 1);

-- Inserting author-book associations
INSERT INTO authors_books VALUES (1, 1);  -- J.K. Rowling wrote Harry Potter
INSERT INTO authors_books VALUES (2, 2);  -- George R.R. Martin wrote A Game of Thrones
INSERT INTO authors_books VALUES (3, 3);  -- Jane Austen wrote Pride and Prejudice
INSERT INTO authors_books VALUES (1, 2);  -- J.K. Rowling also wrote A Game of Thrones
```

### Retrieving all books written by J.K. Rowling

```sql
SELECT books.title
FROM authors
JOIN authors_books ON authors.id = authors_books.author_id
JOIN books ON authors_books.book_id = books.id
WHERE authors.first_name = 'J.K.' AND authors.last_name = 'Rowling';
```

### Retrieving the details of the authors of "A Game of Thrones"

```sql
SELECT authors.first_name, authors.last_name
FROM books
JOIN authors_books ON books.id = authors_books.book_id
JOIN authors ON authors_books.author_id = authors.id
WHERE books.title = 'A Game of Thrones';
```

### Examples of index usage

Some good examples of index usage are:

- The primary key index in the authors table, on the "id" attribute, as it is crucial for efficiently locating specific authors based on their unique id.
- The foreign key index in the books table, on the "publisher_id" attribute, as it facilitates quick retrieval of books associated with a particular publisher, optimizing join operations.
- The composite index on the authors_books association table, which is made of author_id and book_id and supports the many-to-many relationship between authors and books. It enhances the performance of queries involving specific author-book associations.
- Index on frequently queried columns, like for example on the "title" attribute of the books table. If queries frenquently search for books based on their titles, creating an index on the "title" column can significantly improve query performance.

Some bad examples of index usage are:

- Over-indexing: Creating an index on both first_name and last_name on the authors table might be unnecessary if queries rarely involve searching for authors based on both criteria. It could lead to index bloat and increased maintenance overhead without substantial performance gains.
- Indexing on columns with low selectivity: In the books table, if most books have the same publication year, indexing on this column might not significantly enhance query performance. Indexing columns with high selectivity is generally more beneficial.
- Unused indexes: If there are no queries that involve searching or joining based on the publishers table, creating an index on this table might be unnecessary and add unnecessary overhead.
- Overreliance on clustered index: In the books table, while clustering on the primary key (id) is common, overreliance on the clustered index for all types of queries might lead to suboptimal performance for certain query patterns.

## What should be considered when improving index usage

- Selective indexing: Choose indexes based on the selectivity of columns. Indexing highly selective columns enhances the efficiency of the index.
- Query patterns: Consider the types of queries that are commonly executed. Indexes should align with the patterns of data retrieval in your application.
- Balance between read and write operations: While indexes can significantly improve read performance, they may impact write operations. Evaluate the trade-offs based on your application's requirements.
- Regular monitoring and maintenance: Periodically review and optimize indexes based on the evolving usage patterns of your application. Unused or redundant indexes can be identified and removed.
- Use tools for analysis: Utilize database management tools to analyze query execution plans and identify areas where indexes can be beneficial. This helps in making informed decisions about index creation.

## Conclusion

When considering to optimize your database, understanding how indexes work is essential. They are very important for efficient data rtrieval, but their effectiveness depends on factors such as selectivity, indexing strategies and mantenance challenges.
