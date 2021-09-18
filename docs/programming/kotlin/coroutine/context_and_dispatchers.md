# Coroutine context and dispatchers

# Coroutine context

Coroutines always execute in some context represented by a value of the `CoroutineContext` type

The coroutine context is a set of various elements:

- Job: A cancellable piece of work, which has a defined lifecycle
- ContinuationInterceptor: A mechanism which listens to the continuation within a coroutine and intercepts its resumption.
- CoroutineExceptionHandler: A construct which handles exceptions in coroutines.

```kt
fun main() {
    val defaultDispatcher = Dispatchers.Default
    val emptyParentJob = Job()
    val combinedContext = defaultDispatcher + emptyParentJob
    GlobalScope.launch(context = combinedContext) {
        println(Thread.currentThread().name)
    }
    Thread.sleep(50)
}
```

## Dispatchers and threads

The coroutine context includes a coroutine dispatcher that determines what thread or threads the corresponding coroutine uses for its execution.

All coroutine builders like `launch` and `async` accept an optional `CoroutineContext` parameter that can be used to explicitly specify the dispatcher for the new coroutine and other context elements.

```kt
/**
 * When `launch { ... }` is used without parameters, it inherits the context from the `CoroutineScope` it is being launched from.
 * In this case, it inherits the context of the main `runBlocking` coroutine which runs in the main thread.
 */
launch { // context of the parent, main runBlocking coroutine
    println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
}
// Dispatchers.Unconfined is a special dispatcher that also appears to run in the main thread.
launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
    println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
}
launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher
    println("Default               : I'm working in thread ${Thread.currentThread().name}")
}
launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
    println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
}
```

It produces the following output (maybe in different order):

```
Unconfined            : I'm working in thread main
Default               : I'm working in thread DefaultDispatcher-worker-1
newSingleThreadContext: I'm working in thread MyOwnThread
main runBlocking      : I'm working in thread main
```

|                                         | Thread used      | Max number of threads                                                                      | Useful when                                                   |
| --------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| Dispatchers.Default                     | thread pool      | Number of CPU cores                                                                        | Executing CPU-bound code                                      |
| Dispatchers.IO                          | thread pool      | Defined in IO_PARALLELISM_PROPERTY_NAME. The default value is max(64, number of cpu cores) | Executing IO-heavy code                                       |
| Dispatchers.Main                        | main             | 1                                                                                          | Working with UI elements                                      |
| executorService.asCoroutineDispatcher() | thread pool      | Defined by ExecuteService config                                                           | Need to limit number of threads that the coroutine can run in |
| runBlocking {...}                       | Currently thread | 1                                                                                          | Bridging blocking and suspending code                         |

## Unconfined vs confined dispatcher

The Dispatchers.Unconfined coroutine dispatcher starts a coroutine in the caller thread, but only until the first suspension point. After suspension it resumes the coroutine in the thread that is fully determined by the suspending function that was invoked.

```kt
launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
    println("Unconfined      : I'm working in thread ${Thread.currentThread().name}")
    delay(500)
    println("Unconfined      : After delay in thread ${Thread.currentThread().name}")
}
launch { // context of the parent, main runBlocking coroutine
    println("main runBlocking: I'm working in thread ${Thread.currentThread().name}")
    delay(1000)
    println("main runBlocking: After delay in thread ${Thread.currentThread().name}")
}
```

Produces the output:

```
Unconfined      : I'm working in thread main
main runBlocking: I'm working in thread main
Unconfined      : After delay in thread kotlinx.coroutines.DefaultExecutor
main runBlocking: After delay in thread main
```

So, the coroutine with the context inherited from `runBlocking {...}` continues to execute in the `main` thread, while the unconfined one resumes in the default executor thread that the `delay` function is using.
