# Chapter 2: Data Models and Query Languages

Most applications are built by layering data models on top of each other. For example:

1. As an application developer, you look at the real world, and model it in terms of objects or data structures.
2. When you want to store those data structures, you express them in terms of a general-purpose data model, such as JSON or XML documents, tables in a relational database, or a graph model.
3. The engineers who built your database software decided how to represent data structures in terms of bytes of memory, on disk, or on a network.
4. On lower levels, hardware engineers figured out how to represent bytes in terms of electrical currents, pulses of light, magnetic fields, and more.

## Relational Model Versus Document Model

The best-known data model today is probably SQL, based on the relational data model proposed by Edgar Codd in 1970: data is organized into relations (called tabled in SQL), where each relation is an unordered collection of tuples (rows in SQL).

## The Birth of NoSQL

The name "NoSQL" doesn't actually refer to any particular technology -- it was originally intended as a catchy Twitter hashtag in 2009.

There are several driving forces behind the adoption of NoSQL databases:

- A need for greater scalability than relational databases, including very large datasets or very high write throughput.
- A widespread preference for free and open source software over commerical database products.
- Specialized query operations that are not well supported by the relational model.
- Frustration with the restrictiveness of relational schema, and a desire for a more dynamic and expressive data model.

## The Object-Relational Mismatch

If data is stored in relational tables, an awkward translation layer is often needed between application code and the data storage. This disconnect can sometimes be referred to as an impedence mismatch.

JSON can be a more appropriate alternative to relational models, especially if the stored object is a self-contained document. Some developers feel that the JSON model reduces the impedence mismatch between application code and the database layer. The JSON representation has better locality than a multi-table schema.

## Many-to-One and Many-to-Many Relationships

There are advantages to having a standardized list of options, and letting users choose from a drop-down list or autocompleter:

- Consistent style and spelling
- Avoiding ambiguity
- Ease of updating
- Localization support -- when a site is translater into other languages, the standardized lists can be localized
- Better search

Whether you store an ID or a text string is a question of duplication. When you store the text directly, you are duplicating the human-meaningful information in every record that uses it. Removing such duplication is the key idea behind normalization in databases.

## Relational Versus Document Databases Today

The main arguments in favor of the document data model are schema flexibility, better performance due to locality, and that for some applications it is closer to the data structures used by the application. The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.

### Which data model leads to simpler application code?

If the data in your application has a document-like structure (i.e. a tree of one-to-many relationships, where the entire tree is loaded at once), then it's probably a good idea to use a document model. The relational technique of shredding -- splitting a document-like structure into multiple tables -- can lead to cumbersome schemas and complicated application code.

### Schema flexibility in the document model

Document databases often have an implicit schema, which is not enforced by the database. This can be referred to as schema-on-read, as opposed to schema-on-write. Schema-on-read is similar to dynamic (runtime) type checking in programming languages, and schema-on-write is similar to static (compile-time) type checking.

### Data locality for queries

If your application often needs to access the entire document, there is a performance advantage to storage locality. This advantage only applies if you need large parts of the document at the same time. The database typically needs to load the entire document, even if you access only a small portion of it, which can be wasteful on large documents. On updates to a docuent, the entire document usually needs to be rewritten -- only modifications that don't change the encoded size of a document can easily be performed in place. For these reasons, it is generally recommended that you keep documents fairly small and avoid writes that increase the size of the document. These limitations significantly reduce the set of situations in which document databases are useful.

The idea of grouping related data together is not limited to document models. For example, Google's Spanner database accomplishes this in a relational data model, by allowing schemas to be nested. The column-family concept in Bigtable has a similar purpose.

### Convergence of document and relational databases

PostgreSQL since version 9.3, MySQL since version 5.7, and IBM DB2 since version 10.5 support JSON documents.

Some document databases support relational style joins (e.g. RethinkDB). Some MongoDB drivers support joins (effectively a client-side join, which will likely be slower than letting the database server handle it)

## Query Languages for Data

When the relational model was introduced, new ways of querying data were formed: SQL (declarative query language), IMS and CODASYL (imperative query languages).

A declarative query language is attractive because it is typically more concise and easier to work with than an imperative API. More importantly, it hides the implementation details of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries.

Declarative languages often lend themselves to parallel execution. Imperative code is very hard to parallelize across multiple cores/machines, because it specifies instructions that must be executed in a specific order. Declarative languages have a better chance of getting faster, because they only specify the pattern of the results, not the algorithm used to determine them.

## MapReduce Querying

MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google.

MongoDB supports the use of MapReduce. You can use JavaScript code to write the `map` and `reduce` functions within a query. Since this imperative approach would make optimizing difficult, MongoDB v2.2 introduced aggregation pipelines, which is a declarative alternative to MapReduce.

Map and Reduce functions must be pure functions (no side-effects).

NoSQL may be reinventing SQL in disguise.

## Graph-Like Data Models

As many-to-many relationships become more complex, it may make sense to model the data in a graph-like model.

A graph consists of two kinds of objects:

- vertices (also known as nodes or entities)
- edges (also known as relationships or arcs)

### Property Graphs

Each vertex consists of:

- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs)

Each edge consists of:

- A unique identifier
- The vertex at which the edge starts (the tail vertex)
- The vertex at which the edge ends (the head vertex)
- A label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)

You can think of a graph store as consisting of two relational tables, one for vertices and one for edges.

### Cypher Query Language

Cypher is a declarative query language for property graphs, created for the Neo4j graph database.

### Graph Queries in SQL

Graph models can be traversed using SQL, but it takes a lot more code. In SQL, the number of joins can be variable, so you must use recursive common table expressions (CTEs). This is supported in PostgreSQL, DB2, Oracle, and SQL Server.

### Triple-Stores and SPARQL

The triple-store is mostly equivalent to the property graph model, using different terminology. In triple-stores, information is stored in three-part statements: (subject, predicate, object). For example: (Jim, likes, bananas).

### The Foundation: Datalog

Datalog is much older than Cypher or SPARQL, but it's important because it serves as the foundation for these later query languages. Datalog was studied extensively by academics in the 1980s, and can be powerful for simple use-cases.
