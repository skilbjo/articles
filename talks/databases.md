# Databases

### History

Graph query language

Old technology

Relational algebra and set theory

## How it works

Database tables are files stored in a binary-like format (like Avro or ORC)
that enforce schema. Schema is stored in the meta database.

Database execution happens by connecting a handful of plan nodes that implement
small parts of query execution.

Examples are:
- file scan node: scan a database file
- projection node: select only certain fields from the data passed in
- selection node: given a test, test the data passsed in. If test passes,
  return that field. If fails, drop that field
- sort node: sorts the data based on a sort criteria
- aggregation node: essentially, count, sum, and average all implemented
  identically, but output different information depending on the requested
  aggregation. Perform similar operations. Just like in order to average a
  column, you need to sum them up and count all of the records.
- nested loops join: join two sets of data based on a join condition. Least
  performant way of joining
- merge join: same as nested loops join, but more performant implementation.
  However, data must be sorted, which depending on size, could be expensive.
  Additionally, since almost guaranteed files will be larger than memory, an
  "out of core \[memory\] (core memory is older term for RAM)" strategy must be
  implemented.
- hash join: same as nested loops join, but a more performant implementation.
  Usually better than merge join, if one of the tables can fit into memory.

## Sorting, Hashing, and Single Table Queries


## Joins


## Indexes

An index is similar to an index or table of contents in a paper book. If you
are looking for a topic, you scan a much smaller piece of information to tell
you whereabouts in the larger piece of information where you can find the topic
you're after.

Implemented the same as tables, with a separate file.

Levels of an index

B-tree vs hash index

Query optimizer will determine if using an index is necessary; and if multiple
indexes on a table, which one the query optimizer should use.

In order to have a query optimizer work effectively, need to have relatively
accurate table statistics. Table statistics don't need to be accurate, but
optimizer needs certain rough estimates of the table size and column value in
order to come up with the fastest plan.

## Relational Model and Query Optimization

## Query Parser

## Logging for Atomicity and Durability


## Acid


## Concurrency


## Distributed Databases


## Replication
