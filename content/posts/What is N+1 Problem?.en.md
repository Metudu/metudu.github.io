---
date: '2025-11-11T13:59:04+03:00'
draft: false
title: 'What Is N+1 Problem?'
author: ["Metudu"]
tags: ["tech", "n+1", "database", "golang"]
cover:
    image: "/n+1.png"
showWordCount: true
hideSummary: true
---
## Understanding N+1
The N+1 problem occurs when you make **one** query to fetch _N_ objects, and then loop through those objects and make **N additional queries**. When combined, this results in a total of **N+1 queries**.

## Illustrating with an example
Consider the following example:

```go
books := db.getBooks() 
```
Here, we are simply retrieving all the books from the database. this is **one** simple query that returns **N** objects.

```go
for index, book := range books {
    authors = append(authors, db.getAuthor(book.author_id))
}
```
What we just did is iterate through the N books and fetch the author for each one. The `db.getAuthor()` function run **N times**, which means we executed **N additional queries**.

In total, we made **N+1 queries**.

## But, what is the problem here?
The real issue is that we often don't know what **N** represents. It might be small or it might be very large. If it’s small, the extra queries likely won’t affect performance much. But if **N grows**, the number of queries—and therefore the latency—can increase dramatically.

## Solution
To solve this issue, we need to eliminate the N individual queries and replace them with a **constant** number of queries.

### Solution 1: Utilizing joins
One of the most common—and often the most performant—solutions to the N+1 problem is using a **JOIN** in the initial query. JOIN operations allow you to combine rows from multiple tables based on a related column, retrieving all necessary data (books and their authors) in a single database request.

This transforms the 1 (books query) + N (author queries) pattern into one single query.

A simplified SQL query might look like this:
```sql
SELECT b.*, a.name AS author_name
FROM books b
JOIN authors a ON b.author_id = a.id;
```
The exact query may differ depending on your schema, but the essential part is the **JOIN**.

Your updated code might look like this:
```go
authors := db.getAuthorsAndBooks()
// This function performs the JOIN internally, so no loop-based queries are needed.
// Note that we now have one long, single query.
```

### Solution 2: Using WHERE IN clause
Another effective strategy is to reduce the N individual author queries into a single query by using the **WHERE...IN** clause. This approach is sometimes called _batching_ or _pre-loading_.

This reduces the 1 (books) + N (authors) queries into:
- 1 query to get all books
- 1 query to get all authors

Total: **2 constant queries**.

```go
books := db.getBooks() // This single query returns N books

var authorIDs []int 
for _, book := range books {
    authorIDs = append(authorIDs, book.author_id)
    // We are simply collecting author IDs here.
    // No queries are executed in this loop.
}

authorsMap := db.getAuthors(authorIDs)
// This runs one query to fetch all authors with matching IDs.
// We now made only 2 queries instead of N+1.
```

The updated `db.getAuthors()` function would run a query similar to:
```sql
SELECT * FROM authors WHERE id IN (); /* we need to inject the values we have in the slice. */
```

## Conclusion
The N+1 problem is a common performance pitfall in database-driven applications. While it may remain unnoticed with small datasets, it becomes a major bottleneck as data grows. Fortunately, solutions such as JOINs or **batching** with **WHERE IN** allow you to transform potentially thousands of queries into a small, constant number of efficient database calls. By understanding and applying these patterns, you can make your application significantly faster, more scalable, and more robust.
