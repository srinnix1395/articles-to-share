1. Định nghĩa
> Coroutine is a light-weight thread. Like threads, coroutines can run in parallel, wait for each other and communicate. The biggest difference is that coroutines are very cheap, almost free: we can create thousands of them, and pay very little in terms of performance. True threads, on the other hand, are expensive to start and keep around. A thousand threads can be a serious challenge for a modern machine.

> A lightweight thread means it doesn’t map on the native thread, so it doesn’t require context switching on the processor, so they are faster.

> Coroutines do not replace threads, it’s more like a framework to manage it.

> **The exact definition of Coroutines:** A framework to manage concurrency in a more performant and simple way with its lightweight thread which is written on top of the actual threading framework to get the most out of it by taking the advantage of cooperative nature of functions.

2. Tại sao cần

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

Khi sử dụng Coroutine, code là async nhưng nhìn như sync (dễ hiểu hơn):
```
suspend fun fetchAndShowUser() {
     val user = fetchUser() // fetch on IO thread
     showUser(user) // back on UI thread
}
```

4. Các khái niệm

- `suspend`
- *Coroutine builder*
- *Coroutine scope*
- *Coroutine context*

### suspend

Suspend function là một function có thể start, pause và resume. Chúng ta chỉ có thể gọi một suspend fucntion bên trong một suspend function khác hoặc bên trong một coroutine

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

Sử dụng `launch` trong trường hợp không cần kết quả
```
GlobalScope.launch(Dispatchers.Main) {
    fetchUserAndSaveInDatabase() // do on IO thread
}
```

`launch()` trả về một đối tượng `Job` để ta có thể cancel coroutine

##### async

> Perform a task and return a result

Sử dụng `async()` trong trường hợp muốn có kết quả trả về.
```
GlobalScope.launch(Dispatchers.Main) {
    val userOne = async(Dispatchers.IO) { fetchFirstUser() }
    val userTwo = async(Dispatchers.IO) { fetchSecondUser() }
    showUsers(userOne.await(), userTwo.await()) // back on UI thread
}
```

`async()` trả về một object `Deffered<T>`, chúng ta sẽ sử dụng function `await()` để lấy kết qủa.

##### withContext

Giống với `async()` nhưng không cần gọi `await()`

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

### CoroutineScope

Using scope to keep track of coroutines created inside scope. So to cancel many coroutines, use `scope.cancel()`. Coroutine scope take a coroutine

### CoroutineContext

##### Dispatcher

Declare the thread which the work will run. There are majorly 3 types:
- `Dispatcher.IO`: is used to do the I/O related works
- `Dispatcher.Default`: same as `Schedulers.computation()` in *RxJava*, is used to do the CPU intensive works
- `Dispatcher.Main`: main thread.
