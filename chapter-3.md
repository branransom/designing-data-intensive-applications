# Chapter 3: Storage and Retrieval

In order to tune a storage engine to perform well on your kind of workload, you need to have a rough idea of what it's doing under the hood.

## Data Structures That Power Your Database

In this book, the term "log" is used to mean an append-only sequence of records. It doesn't have to be human-readable.

The world's simplest database:

```
#!/bin/bash

db_set () {
  echo "$1,$2" >> database
}

db_get () {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

`db_set` will not overwrite any existing records. Instead, it will append the record to the end of the file. This is very performant.

`db_get` will find the last value for a given key in the file. This is not so performant for large datasets, since it requires the entire file to be read each time a record is queried. In algorithmic terms, the cost of a lookup is O(n).

An *index* is derived from the primary data. It does not affect the contents of the database -- only the performance of queries. Any kind of index usually slows down writes, because the index also needs to be updated every time data is written.

### Hash Indexes

With key-value data, a hash map can be kept where every key is mapped to a byte offset in the data file -- the location where the value can be found. The offset is updated any time data is written to the log file. This can be a viable approach, as long as all data can be contained within the available RAM. It is well suited for situations where there are not many distinct keys, but the values of the keys are updated frequently -- for instance, a scenario where the key is a URL to cat videos and the value is the number of times each video was watched.

With append-only solutions, the concern of running out of disk space must be considered. A good solution to this is to break the log into segments of a certain size by closing a segment file when it reaches a certain size, and making subsequent writes to a new segment file. We can then perform compaction on these files. Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key.

Since compaction makes the files smaller, the segments can also potentially be merged. Segments are never modified after they have been written, so the merged result is published to a new file. This process can be done in a background thread, so that incoming read requests can still be handled. After the merging process is complete, the read requests can use the new meged segment instead of the old segments -- at which point, the old segments can be deleted.

Lots of detail goes into making this simple idea work in practice:

- File format
    - Use a binary format that first encodes the length of a string in bytes, followed by the string
- Deleting records
    - In order to delete a record, a special record must be appended (often called a tombstone)
- Crash recovery
    - If the database is restarted, the in-memory hash maps are lost. A snapshot of each segment's hash map can be kept on disk to speed up reloading the data into memory. Bitcask does this.
- Partially written records
    - The database may crash while a record has only partially been appended. Bitcask uses a checksum to identify and ignore corrupted parts.
- Concurrency control
    - A common implementation choice is to have only one writer thread. Data file segments are append-only and otherwise immutable, so they can be read concurrently.

Limitations of hash table indexes:

- The hash table must fit in memory. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make this perform well. It requires a lot of random access I/O.
- Range queries are not efficient. You cannot easily scan over all keys between kitty00000 and kitty99999 -- you'd have to look up each key individually.

### SSTables and LSM-Trees

We can make a simple change to the format of our segment files: we require that the sequence of key-value pairs is *sorted by key*. We call this format *Sorted String Table*, or *SSTable* for short.

There are several advantages to this approach:

- Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is similar to mergesort.
- In order to find a particular key in a file, you no longer need to keep an index of all the keys in memory. You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse.
- It is possible to group records into a block and compress it before writing to disk. This saves disk space, and reduces I/O bandwidth.
