# Definition

> Coroutine is a light-weight thread. Like threads, coroutines can run in parallel, wait for each other and communicate. The biggest difference is that coroutines are very cheap, almost free: we can create thousands of them, and pay very little in terms of performance. True threads, on the other hand, are expensive to start and keep around. A thousand threads can be a serious challenge for a modern machine.

> A lightweight thread means it doesn’t map on the native thread, so it doesn’t require context switching on the processor, so they are faster.

> Coroutines do not replace threads, it’s more like a framework to manage it.

> **The exact definition of Coroutines:** A framework to manage concurrency in a more performant and simple way with its lightweight thread which is written on top of the actual threading framework to get the most out of it by taking the advantage of cooperative nature of functions.

# Why we need

Eg:
```
fun fetchUser(): User {
    // make network call
    // return user
}

fun showUser(user: User) {
    // show user
}

fun fetchAndShowUser() {
    val user = fetchUser()
    showUser(user)
}
```

When using Coroutine, our code is asynchronous but looks like synchronous:
```
suspend fun fetchAndShowUser() {
     val user = fetchUser() // fetch on IO thread
     showUser(user) // back on UI thread
}
```

# How to implement in Android

Using Gradle, Google please!

# Conception

- `Dispatcher`
- `suspend`
- `GlobalScope`
- `async`
- `await`

### Dispatcher

Declare the thread which the work will run. There are majorly 3 types:
- `Dispatcher.IO`: is used to do the I/O related works
- `Dispatcher.Default`: same as `Schedulers.computation()` in *RxJava*, is used to do the CPU intensive works
- `Dispatcher.Main`: main thread.

### suspend

Suspend function is a function that could be started, paused, and resume. We can only call suspend function in another suspend function or in a coroutine.
```
GlobalScope.launch(Dispatchers.Main) {
    val user = fetchUser() // fetch on IO thread
    showUser(user) // back on UI thread
}
```

### Coroutine builder

`launch()`, `async()`, `withContext()`

##### launch

> Fire and forget

We use `launch()` in case we just want to perform a task but not want a result.
```
GlobalScope.launch(Dispatchers.Main) {
    fetchUserAndSaveInDatabase() // do on IO thread
}
```

`launch()` return a `Job` to cancel the coroutine.

##### async

> Perform a task and return a result

We use `async()` in case we want to perform a task and get a result
```
GlobalScope.launch(Dispatchers.Main) {
    val userOne = async(Dispatchers.IO) { fetchFirstUser() }
    val userTwo = async(Dispatchers.IO) { fetchSecondUser() }
    showUsers(userOne.await(), userTwo.await()) // back on UI thread
}
```

`async()` return a `Deffered<T>` object, which has an `await()` function to get the result.

##### withContext

> Another way of writing async but there is no need to use `await()`

```
suspend fun fetchUser(): User {
    return withContext(Dispatchers.IO) {
        // make network call
        // return user
    }
}
```

**Note:**: The thumb rules:
- Use `withContext` when you do not need the parallel execution.
- Use `async` only when you need the parallel execution.
- Use `withContext` to return the result of a single task.
- Use `async` for results from multiple tasks that run in parallel.

### Scope

We use Scope in Coroutins to cancel the background task as soon as the activity is destroyed.
