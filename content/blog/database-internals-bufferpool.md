---
title: 'Database Internals: BufferPool'
date: 2026-06-02T00:07:00
tags:
  - database-internals go-programming software-engineering
summary: A somewhat deep dive into my understanding of the bufferpool in a database system.
---

Following my promise and commitment I made on LinkedIn, this is the full article detailing how I completed the second internal system of a database engine, the BufferPool. 

Firstly, what is the BufferPool (bp) in a database exactly? This is a cache that handles returning database pages when a query is run from the "frontend" of the database. What do I mean by this? 

When you run a query, say;

`SELECT * FROM User WHERE id = 1;`

The data that is returned from this query is stored in a page (a file), on disk. This page also holds other data records and is usually about 4KB. For this data to be sent back to you, the database engine needs to crawl all the pages efficiently to retrieve this data from disk, and it can be taxing to do this for every single READ or UPDATE query you make to the database.

That is where the bp comes in. The bp is a cache that stores pages in memory for faster reads when data is queried. The workflow is that when you query for data, the engine checks the bp to see if it has that page cached in memory. If it does, it returns it from there without needing to go to disk to read the data. However, if the data is not in the cache, it reads from disk and writes to the cache.

![data-travel-path-with-buffer-pool](/images/uploads/20260603-003759.png "How Data is Retrieved from the Database")

### How does the BufferPool Work

Now that we understand what the bp is, what is it actually made of and how does it work? Like I mentioned earlier, it is used to store frequently read data. But it cannot hold all the data that is read from disk forever. The bp manages memory space by designating a strict buffer size that holds data and evicts the least frequently used data.

For example, let's say our bp can only hold 4 pages at a time, and you send 4 queries that require reading 4 different pages. These pages will be cached in memory, and when you send a fifth query that requires another page, the least recently used page in the cache will be evicted, i.e the first page of the initial 4 queries.

![buffer-pool-holding-data-pointing-to-pages](/images/uploads/20260603-010820.png "Queries referencing Pages in the BufferPool")

This way, the bp maintains frequently accessed data and doesn't compromise on space to store a large quantity of data in memory. 

After reading this initially, I thought that this functionality will be handled by writing an LRU to maintain the least recently used page and evict it accordingly; however, I came across a simpler way to handle this functionality was implemented and some reasons why it trumps using a standard LRU implementation.

The implementation of the LRU page is done with using an array and a clock-based counter. When a new page is added to the bp, that page has metadata attached to it to identify if it was used recently or not, think of it as a number or a boolean field. With this, a clock counter is initialized as 0 and starts checking from the first page in the bp. If the metadata of the bp shows it has not been accessed, we evict that page. However, if it has been accessed, we change it's status to "not-accessed" and keep moving. When we get to the end of the buffer, we reset our clock-counter to 0 and start again to check for the LRU page in the buffer.

With this implemenatation, we don't have to worry about the complexity of writing a modified LRU for this usecase.

### Why use Clock-Based Counter Over LRU for BufferPool Implementation

Here are some advantages of using the Clock-Based Counter over the LRU.

- **Locality**: with pages located closely to each other, we can traverse the array in a contiguous block of memory to get our pages rather than travelling to different memory addresses that may be spread apart to locate the next page in a bp.

-
