# Getting Started

This guide demonstrates the FactStore API and its basic operations, like appending new facts, finding stored facts, and streaming facts in real-time.

## Setup 

Todo: add maven dependencies and sample import statements once available

## Initializing a FactStore

The `FactStore` interface is the main entry point when interacting with any FactStore. Before you can start using it, you need to obtain an instance of the `FactStore` interface. Supported implementation provide a way to obtain an instance. 

The following snippet shows how to initialize a fact store backed by FoundationDB. It assumes that you already have a FoundationDB cluster running and a cluster file available at the specified location.  

```kotin
FDB.selectAPIVersion(FDB_API_VERSION)
val factStore = buildFdbFactStore(
    clusterFilePath = "/etc/foundationdb/fdb.cluster",
    name = "getting-started"
)
```

Alternatively, you can instantiate an in-memory `FactStore`like this:

```kotlin
TODO (Not yet implemented)
```

## Appending Facts

### Simple Appends

To append new facts, you can use the `append` functions available. The most straightforward way is to pass the fact to append directly to the `append` function, like that:

```kotlin
val factToAppend = Fact(
    id = FactId.generate(), // generates a new identifier
    type = "order-placed", // references the fact type 
    payload = "This is some event payload".toByteArray(UTF_8), // payload as byte stream
    subjectRef = SubjectRef( // facts can reference a higher-level entities they belong to  
        type = "orders", // the type/category of the subject
        id = "order-123" // the identifier of the subject within that type
    ),
    createdAt = Instant.now(), // timestamp
)


factStore.append(factToAppend)
```

If you need to append more than one fact, you can also pass a list of facts instead. This operation is atomic, so either all facts will be appended, or none will.

```kotlin
val factsToAppend = listOf(fact1, fact2, fact3)

factStore.append(factsToAppend)
```

### More Control with `AppendRequest`

So far, the facts were appended without any condition. Additionally, if you were to retry the operation after a crash but didn't know whether the previous operation failed or succeeded, potentially resulting in the same facts being appended twice. 

In order to address these concerns, you'd need more control about the append operation. This is where the `AppendRequest` comes in. 

Using `AppendRequest` allows you to 

- define append conditions which are evaluated before appending facts
- define an idempotency key that makes append operations safer by ensuring multiple append requests with the same key are processed at most one; put simply, it makes the append operation idempotent across retries.