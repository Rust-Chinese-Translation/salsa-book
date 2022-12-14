# Database

Continuing our dissection, the other thing which a user must define is a
**database**, which looks something like this:

```rust,ignore
{{#include ../../salsa/examples/hello_world/main.rs:database}}
```

The `salsa::database` procedural macro takes a list of query group
structs (like `HelloWorldStorage`) and generates the following items:

* a copy of the database struct it is applied to
* a struct `__SalsaDatabaseStorage` that contains all the storage structs for
  each query group. Note: these are the structs full of hashmaps etc that are
  generaetd by the query group procdural macro, not the `HelloWorldStorage`
  struct itself.
* an impl of `HasQueryGroup<G>` for each query group `G`
* an impl of `salsa::plumbing::DatabaseStorageTypes` for the database struct
* an impl of `salsa::plumbing::DatabaseOps` for the database struct

## Key constraint: we do not know the names of individual queries

There is one key constraint in the design here. None of this code knows the
names of individual queries. It only knows the name of the query group storage
struct. This means that we often delegate things to the group -- e.g., the
database key is composed of group keys. This is similar to how none of the code
in the query group knows the full set of query groups, and so it must use
associated types from the `Database` trait whenever it needs to put something in
a "global" context.

## The database storage struct

The `__SalsaDatabaseStorage` struct concatenates all of the query group storage
structs. In the hello world example, it looks something like:

```rust,ignore
struct __SalsaDatabaseStorage {
    hello_world: <HelloWorldStorage as salsa::plumbing::QueryGroup<DatabaseStruct>>::GroupStorage
}
```

We also generate a `Default` impl for `__SalsaDatabaseStorage`. It invokes
a `new` method on each group storage with the unique index assigned to that group.
This invokes the [inherent `new` method generated by the `#[salsa::query_group]` macro][new].

[new]: query_groups.md#group-storage

## The `HasQueryGroup` impl

The `HasQueryGroup` trait allows a given query group to access its definition
within the greater database. The impl is generated here:

```rust,ignore
{{#include ../../salsa/components/salsa-macros/src/database_storage.rs:HasQueryGroup}}
```

The `HasQueryGroup` impl combines with [the blanket impl] from the
`#[salsa::query_group]` macro so that the database can implement the query group
trait (e.g., the `HelloWorld` trait) but without knowing all the names of the
query methods and the like.

[the blanket impl]: query_groups.md#impl-of-the-hello-world-trait

## The `DatabaseStorageTypes` impl

Then there are a variety of other impls, like this one for `DatabaseStorageTypes`:

```rust,ignore
{{#include ../../salsa/components/salsa-macros/src/database_storage.rs:DatabaseStorageTypes}}
```

## The `DatabaseOps` impl

Or this one for `DatabaseOps`, which defines the for-each method to
invoke an operation on every kind of query in the database. It ultimately
delegates to the `for_each` methods for the groups:

```rust,ignore
{{#include ../../salsa/components/salsa-macros/src/database_storage.rs:DatabaseOps}}
```
