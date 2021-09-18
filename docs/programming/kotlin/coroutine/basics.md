# Coroutine

A coroutine is an instance of **suspendable** computation. It is conceptually similar to a thread. However, a coroutine is not bound to any particular thread. It may suspend its execution in one thread and resume in another one.

Coroutines can be thought of as **light-weight** threads, but there is a number of important differences that make their real-life usage very different from threads.

## Coroutine Builders

Coroutine builders are a way of creating coroutines. Since they are not suspending themselves, they can be called from non-suspending code or any other piece of code. They act as a link between the suspending and non-suspending parts of our code.

### launch

`launch` Launches a new coroutine without blocking the current thread and returns a reference to the coroutine as a Job. The coroutine is cancelled when the resulting job is cancelled.

```kt
fun main() {
  GlobalScope.launch {
    delay(200)
    println("World")
  }
  println("Hello")
  Thread.sleep(400)
}
```

The launch function is an extension function on the interface `CoroutineScope`. This is a part of an important mechanism called structured concurrency, with the purpose of building a relationship between parent coroutine and child coroutine. Coroutines are launched in coroutine builders and are bound by a coroutine scope. A lazy way to launch a coroutine would be to use the `GlobalScope`. This means that the coroutine would be running for as long as the application is running and, if there is no important reason to do this, I believe that it’s a way to wrongly use resources.

```kt
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job
```

`CoroutineContext` is a persistent dataset of contextual information about the current coroutine
`CoroutineStart` is the mode in which you can start a coroutine.

- DEFAULT: Immediately schedules a coroutine for execution according to its context.
- LAZY: Starts the coroutine lazily, only when it is needed.
- ATOMIC: Same as DEFAULT but cannot be cancelled before it starts.
- UNDISPATCHED: Runs the coroutine until its first suspension point.

### runblocking

`runBlocking` is a coroutine builder that blocks the current thread until all tasks of the coroutine it creates, finish

```kt
fun main() = runBlocking { // The method bridges the non-coroutine world and the code with coroutines
    launch { // launch a new coroutine without blocking the current thread
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}
```

### async

`async` is a coroutine builder which returns some value to the caller. async can be used to perform an asynchronous task which returns a value

`await` is a suspending function that is called upon the async builder to fetch the value of the Deferred object that is returned.

```kt
fun main() = runBlocking {
    val res = async {
        "Hello world"
    }

    println(res.await())
}
```

### withContext

Calls the specified suspending block with a given coroutine context, suspends until it completes, and returns the result.

### Structured Concurrency

If a coroutine is started on `GlobalScope`, the program will not wait for it. For example:

```kt
fun main() = runBlocking {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}

// Hello
```

Everything changes if we get rid of the `GlobalScope` here. `runBlocking` creates a `CoroutineScope` and provides it as its lambda expression receiver. Thus we can call `this.launch` or simply `launch`. As a result, `launch` becomes a child of `runBlocking`. As parents might recognize, parental responsibility is to wait for all its children, so `runBlocking` will suspend until all its children are finished.

## Scope builder

`CoroutineScope` is an interface that has a single abstract property called `coroutineContext`.

```kt
public interface CoroutineScope {
    public val coroutineContext: CoroutineContext
}
```

Whenever a new coroutine scope is created, a new job gets created and gets associated with it. Every coroutine created using this scope becomes the child of this job.

### GlobalScope

This is a singleton and not associated with any Job. This launches top-level coroutines and highly discouraged to use because if you launch coroutines in the global scope, you lose all the benefits you get from structured concurrency.

Reference: https://elizarov.medium.com/the-reason-to-avoid-globalscope-835337445abc

### MainScope

This is useful for UI components, it creates `SupervisorJob` and all the coroutines created with this scope runs on the Main thread.

### CoroutineScope(ctx)

This creates a coroutine scope from provided coroutine context and makes sure that a job is associated with this scope.

```kt
public fun CoroutineScope(context: CoroutineContext): CoroutineScope = ContextScope(if (context[Job] != null) context else context + Job())
```

### coroutineScope

Kotlin avoid thread blocking functions inside coroutine so due to this the use of `runBlocking` is discouraged from inside coroutines ans allow suspending operation instead of this. The suspending function is equivalent of `runBlocking` is the `coroutineScope` builder.

`CoroutineScope` suspends the currently working coroutine until all it’s child coroutines have finished their execution.

```kt
fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope {  // this: CoroutineScope
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```
