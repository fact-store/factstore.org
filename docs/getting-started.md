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
    payload = "This is some fact payload".toByteArray(UTF_8), // payload as byte stream
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

Here is a simple example:

```kotlin
val appendRequest = AppendRequest(
    facts = listOf(fact1),
    idempotencyKey = IdempotencyKey(),
    condition = AppendCondition.None
)
```

Next, just pass the append request object to the `append` function like this:

```kotlin
val appendResult = factStore.append(appendRequest)
```

The return value is of type `AppendResult`, which is just a sealed interface.
The concrete implementation can be of the following types, depending on the append operation's outcome:
- `Appended`: Returned if the facts were appended successfully.
- `AppendConditionViolated`: Returned if the specified append condition is not satisfied. 
- `AlreadyApplied`: Returned if the append request was already processed (see idempotency key)

Please see [Appending Facts](appending-facts.md) for further details.

## Finding Facts

The _FactStore_ specification provides multiple functions to query and find facts, including:

```kotlin
suspend fun findById(factId: FactId): Fact?

suspend fun existsById(factId: FactId): Boolean

suspend fun findInTimeRange(start: Instant, end: Instant = Instant.now()): List<Fact>

suspend fun findBySubject(subjectType: String, subjectId: String): List<Fact>

suspend fun findByTags(tags: List<Pair<String, String>>): List<Fact>

suspend fun findByTagQuery(query: TagQuery): List<Fact>
```

For example, to find all facts of the last 10 minutes, you can do:

```kotlin
val now = Instant.now()
val timestamp10MinutesAgo = now.minusMinutes(10)

val foundFacts = factStore.findInTimeRange(start, now)
println(foundFacts)
```

For more details, see [Finding Facts](finding-facts.md).

## Streaming Facts

You can also stream facts in real-time using the functions defined in `FactStreamer`.
The function `streamAll()` takes no arguments and returns a `Flow` of facts. The facts are streamed in the order they were appended, starting with the oldest fact and proceeding to the newest facts.
Once the function has "caught up" on all facts stored in the fact store, the flow continues and facts are returned as they are appended to the fact store.

The following example streams all facts and prints them to the console:
```kotlin
store.streamAll().collect {
    println(it)
}
```

If you want or need more control over the streaming, e.g. starting from a specific fact, consider using `StreamingOptionSet`:

```kotlin
data class StreamingOptionSet(
    val lastSeenId: FactId? = null,
    val batchSize: Int = 128,
    val pollDelayMs: Long = 250L
)
```

For example, you can start streaming after a specific fact ID. In this case, the fact ID acts as a kind of checkpoint.

```kotlin
store
    .streamAll(StreamingOptionSet(lastSeenId = fact1.id))
    .collect { println(it) }
    
```

Please refer to [Streaming Facts](streaming-facts.md) for more details.