# Định nghĩa
> Coroutine is a light-weight thread. Like threads, coroutines can run in parallel, wait for each other and communicate. The biggest difference is that coroutines are very cheap, almost free: we can create thousands of them, and pay very little in terms of performance. True threads, on the other hand, are expensive to start and keep around. A thousand threads can be a serious challenge for a modern machine.

> A lightweight thread means it doesn’t map on the native thread, so it doesn’t require context switching on the processor, so they are faster.

> Coroutines do not replace threads, it’s more like a framework to manage it.

> **The exact definition of Coroutines:** A framework to manage concurrency in a more performant and simple way with its lightweight thread which is written on top of the actual threading framework to get the most out of it by taking the advantage of cooperative nature of functions.

# Tại sao cần

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

# Các khái niệm

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
- Sử dụng `withContext` khi bạn KHÔNG CẦN các suspend function chạy song song
- Sử dụng `async` chỉ khi bạn CẦN các suspend function chạy song song
- Sử dụng `withContext` để trả về kết quả của MỘT single task
- Sử dụng `async` để trả về kết quả của NHIỀU task song song

### CoroutineScope

CoroutineScope có nhiệm vụ theo dõi tất cả những coroutine được tạo ra thông qua nó. Từ đó, có thể cancel toàn bộ coroutine bằng cách gọi `scope.cancel()`. Một số scope phổ biến
- GlobalScope
- viewModelScope
- lifecycleScope

Ngoài ra, bạn có thể tự tạo ra một scope bằng cách khởi tạo trực tiếp
```
val scope = CoroutineScope(Job() + Dispatchers.Main)
```

### CoroutineContext

CoroutineContext là một tập các phần tử định nghĩa behavior của coroutine. CoroutineContext là tham số khi tạo CoroutineScope. CoroutineContext bao gồm:
- Job - kiểm soát vòng đời của coroutine
- CoroutineDispatcher - xác định thread mà coroutine sẽ chạy
- CoroutineName - tên của coroutine, thường sử dụng cho việc debug
- CoroutineExceptionHandler - xử lý uncaught exception

Khi một CoroutineContext mới được tạo, một `Job` mới sẽ được tạo ra, các tham số còn lại sẽ được kế thừa thừ CoroutineContext cha. Để thê hiện các thành phần của CoroutineContext, chúng ta sử dụng dấu +

### CoroutineDispatcher

- `Dispatcher.IO`: sử dụng cho các tác vụ I/O: đọc ghi file, network...
- `Dispatcher.Default`: tương tự `Schedulers.computation()` trong *RxJava*, sử dụng với các tác vụ tính toán nặng
- `Dispatcher.Main`: main thread.

# Cancel a coroutine

### Multiple coroutine

Cancel CoroutineScope sẽ cancel toàn bộ con của nó - các coroutine
```
val job1 = scope.launch { … }
val job2 = scope.launch { … }

scope.cancel() // all coroutines will be cancelled
```

> Cancelling the scope cancels its children

### A single coroutine

- Sử dụng `Job` hoặc `Deffered` để cancel từng coroutine.
- Khi cancel một coroutine, sẽ có một exception được throw ra cho parent của nó: `CancellationException`.

```
val job1 = scope.launch { … }
val job2 = scope.launch { … }

// First coroutine will be cancelled and the other one won’t be affected
job1.cancel()
```
> A cancelled child doesn’t affect other siblings

### Điều gì xảy ra khi chúng ta cancel một coroutine

- Coroutine không dừng lại ngay lập tức khi bị cancel. Nó vẫn sẽ chạy tiếp cho đến khi hoàn thành công việc. Bởi vậy, chúng ta cần check trạng thái trước khi thực hiện công việc:
  + `job.isActive` hoặc `ensureActive()`
  + `yieald()`
- `withContext` hoặc `delay` là những coroutine có thể cancel được thì ta không cần check như trên

```
val job = launch {
    for(file in files) {
        // TODO check for cancellation
        readFile(file)
    }
}
```

> Cancellation of coroutine code needs to be cooperative!

### Xử lý khi coroutine bị cancel

* Sử dụng `isActive`

```
while (i < 5 && isActive) {
    // print a message twice a second
    if (…) {
        println(“Hello ${i++}”)
        nextPrintTime += 500L
    }
}// the coroutine work is completed so we can cleanup
println(“Clean up!”)
```

* Sử dụng try catch finally

```
val job = launch {
   try {
      work()
   } catch (e: CancellationException){
      println(“Work cancelled!”)
    } finally {
      println(“Clean up!”)
    }
}
```

Tuy nhiên, một coroutine đã bị cancel thì sẽ không có khả năng suspend được nữa. Nên nếu bạn muốn thực hiện một tác vụ suspend bên trong finally block, chúng ta cần sử dụng `NonCancellable` CoroutineContext

```
val job = launch {
   try {
      work()
   } catch (e: CancellationException){
      println(“Work cancelled!”)
    } finally {
      withContext(NonCancellable){
         delay(1000L) // or some other suspend fun
         println(“Cleanup done!”)
      }
    }
}
```

# Xử lý Exception trong coroutines

Khi một coroutine throw ra một exception, exception này sẽ được đẩy lên cho parent của nó. Khi đó
1. Parent sẽ cancel toàn bộ các coroutine con còn lại
2. Parent sẽ cancel chính nó
3. Exception sẽ tiếp tục được đẩy lên parent cao hơn.

![alt text](https://miro.medium.com/max/700/0*UcEpsF2X-ihztU2Z)

Vì behavior này, một số coroutine nằm trong cùng scope sẽ bị cancel theo. Để tránh trường hợp này, chúng ta cần sử dụng `SupervisorJob()`

### SupervisorJob

Khi sử dụng `SupervisorJob` các behavior được mô tả phía trên đều không xảy ra. Thay vào đó, chính coroutine gây ra exception đó phải catch lỗi hoặc CoroutineContext cần có CoroutineExceptionHandler để catch lỗi. Nếu không, app sẽ crash.

Ngoài cách khởi tạo một CoroutineScope và truyền vào một `SupervisorJob`, chúng ta có thể sử dụng extension function `supervisorScope` để có kết quả tương tự

![alt text](https://miro.medium.com/max/700/0*Mrf17HLbWQPfTt1I)

### Dealing with exceptions

Sử dụng `try/catch` để catch exception

Tuy *uncaught exception sẽ luôn luôn được throw* nhưng cách throw lại phụ thuộc vào coroutine builder

##### launch

Exception được throw ngay khi nó xảy ra. Bởi vậy, chúng ta có thể sử dụng `try/catch` với toàn bộ đoạn code:
```
scope.launch {
    try {
        codeThatCanThrowExceptions()
    } catch(e: Exception) {
        // Handle exception
    }
}
```

##### async

Exception sẽ được throw khi bạn gọi `await()`. Bởi vậy, chúng ta có thể chỉ wrap đoạn code `await()` vào catch block
```
supervisorScope {
    val deferred = async {
        codeThatCanThrowExceptions()
    }    try {
        deferred.await()
    } catch(e: Exception) {
        // Handle exception thrown in async
    }
}
```

**Note:**
- Trong ví dụ phía trên, chúng ta sử dụng `supervisorScope` (`SupervisorJob`) thì exception mới được tự xử lý. Nếu bạn sử dụng `Job`, exception sẽ được đẩy lên parent trên thay vì có thể catch được ngay tại chỗ
```
coroutineScope {
    try {
        val deferred = async {
            codeThatCanThrowExceptions()
        }
        deferred.await()
    } catch(e: Exception) {
        // Exception thrown in async WILL NOT be caught here
        // but propagated up to the scope
    }
}
```
- Ngoài ra, trong trường hợp parent của coroutine không phải là scope mà là một coroutine khác, exception cũng mặc định được đẩy lên trên thay vì có thể catch được

### CoroutineExceptionHandler

CoroutineExceptionHandler là một thành phần của CoroutineContext dùng để xử lý các uncaught exception.
```
val handler = CoroutineExceptionHandler {
    context, exception -> println("Caught $exception")
}
```

Exception sẽ được bắt khi
- **When:** Exception bị throw bởi một coroutine tự động throw exception (`launch` thay vì `async`)
- **Where:** Handler được khai báo ở root CoroutineScope hoặc root coroutine

```
val scope = CoroutineScope(Job())
scope.launch(handler) {
    launch {
        throw Exception("Failed coroutine")
    }
}
```

Trong trường hợp tiếp theo, exception sẽ không được CoroutineExceptionHanlder caught:
```
val scope = CoroutineScope(Job())
scope.launch {
    launch(handler) {
        throw Exception("Failed coroutine")
    }
}
```

Đó là bởi vì, CoroutineExceptionHandler không được khai báo ở root coroutine hoặc root scope. Để exception có thể được caught, chúng ta có thể khai báo CoroutineExceptionHandler như sau:
```
val scope = CoroutineScope(Job())
scope.launch(handler) {
    launch {
        throw Exception("Failed coroutine")
    }
}
```

hoặc
```
val scope = CoroutineScope(Job() + handler)
scope.launch {
    launch {
        throw Exception("Failed coroutine")
    }
}
```
