[Android] Dagger 2 - Phần II: Into the Dagger 2

Bài viết là phần thứ II của series bài học vỡ lòng về *Dagger 2*. Nếu bạn chưa đọc phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Basic principles](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)
2. [Android] Dagger 2 - Phần II: Into the Dagger 2
3. [[Android] Dagger 2 - Phần III: The time of our dependencies]()

# Trong bài học trước...

Chúng ta đã nói một chút về việc khởi tạo và quản lý dependency. Tiếp đó, chúng ta ngờ ngợ ra những vấn đề khi ứng dụng được scale up lên. Cuối cùng, chúng ta được giác ngộ với những design principle và design pattern có thể giải quyết giả thiết của bài toán ban đầu.

# Đi vào bài học hôm nay...
Chúng ta sẽ tìm hiểu sâu hơn về *Dagger 2*: trước là lý thuyết và sau là từng bước implement một chương trình đơn giản.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/y1bh782i5k_fredrik-ohlander-MU2pWu95UqA-unsplash.jpg">
  Photo by <a href="https://unsplash.com/@fredrikohlander?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Fredrik Öhlander</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</p>

Một chút kiến thức lịch sử, *Dagger* là một library được *Square* tạo ra để implement *dependency injection* trong *Java* (*Android* là một trường hợp cụ thể hơn). *Dagger 1* là một *dynamic, run-time DI framework* và đã deprecated. *Dagger 1* khởi tạo các dependency "động", tức là việc tạo ra dependency được thực hiện lúc run-time thông qua java reflection. Bởi vậy, nó có nhược điểm là chậm và có thể có run-time exception xảy ra khi chạy ứng dụng.

*Dagger 2* được tiếp nối bởi *Google* và là một *fully static, compile-time DI framework*. Để khắc phục những nhược điểm của người tiền nhiệm, *Dagger 2* sử dụng *annotation processor* (a code generator using annotation) để "viết" code cho chúng ta khi compile. Bởi vậy, nếu có lỗi gì, *Dagger* sẽ báo cho chúng ta và dừng quá trình build chương trình. Cùng với đó, nguyên tắc để gen ra các đoạn code này là cố gắng bắt chước những đoạn code mà người dùng thực sự sẽ viết. Từ đó, code cũng sẽ đơn giản và khả thi hơn khi trace.

Để bắt đầu với *Dagger 2*, chúng ta cần nói qua về một khái niệm quan trọng trong *Dagger* mà chúng ta hay được nghe tới nhưng lại ít được giải thích tường minh. Đó là *dependency graph*.

# Dependency graph

*Dependency graph* trong *Dagger* có thể hiểu là một cái graph biểu diễn những mối quan hệ của các class với những dependency của nó. *Dependency graph* được biểu diễn bằng một *Directed A-cyclic Graph* - đồ thị có hướng không tuần hoàn (hay chính là *DAG* trong *Dagger*).

Chúng ta sẽ sửa đổi một chút ví dụ bài trước và tạo ra một *dependency graph* để hiểu rõ hơn khái niệm này:
```
class Student {

    var book: TextBook

    var pen: Pen

    constructor(book: TextBook, pen: Pen) {
       this.book = book
        this.pen = pen
    }

    fun learn() {
        println("Using a ${pen.ink.color} pen  to learning ${book.getSubjectName()}")
    }
}

data class Pen (val ink: Ink)

data class Ink(val color: String = "Black")
```

... và đây là *dependency graph* tương ứng:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/p2g5hyoqsd_Screenshot%20from%202021-03-25%2017-47-37.png">
</p>

Ở đây, mỗi mũi tên có hướng trong đồ thị biểu diễn một mối quan hệ phụ thuộc: `Student`phụ thuộc vào `TextBook` và `Pen`, `Pen` phụ thuộc vào `Ink`. Vì những sự phụ thuộc này, ta cần khởi tạo các module cấp thấp trước rồi mới có thể khởi tạo được các module cấp cao: khởi tạo `Ink` trước rồi mới đến `Pen`. Và cũng bởi vì cần có thứ tự khởi tạo trước sau như vậy, *dependency graph* không thể tồn tại những vòng lặp đóng hay *a circular dependency* vì *Dagger* sẽ không biết đâu là điểm khởi tạo đầu tiên.

Khi implement *Dagger*, mục tiêu của chúng ta là cần fulfill *dependency graph* trước - cung cấp cho *Dagger* đầy đủ cách khởi tạo của các phần tử trong *dependency grapgh* và giao phần việc còn lại cho *Dagger*. VD: để khởi tạo `Pen`, ta cần khởi tạo `Ink` trước nên ta cần chỉ cho *Dagger* biết cách khởi tạo `Ink` như thế nào. Nếu ta không thêm `Ink` vào *dependency graph*, *Dagger* sẽ không biết khởi tạo `Ink` như thế nào.

Vậy, làm thế nào để fulfill *dependency graph* trong *Dagger 2*? Câu trả lời là *Dagger 2* sẽ cung cấp cho chúng ta các annotation để làm việc đó.

# Annotation trong Dagger 2

*Annotation* là một dạng **chú thích** hoặc một dạng **metadata** được dùng để cung cấp thông tin cho mã nguồn *Java*. *Dagger 2* sẽ sử dụng các thông tin có được thông qua truy vấn các annotation để gen code khi compile.

Các annotation cơ bản trong *Dagger 2* là:
* `@Component` - đánh dấu một interface/abstract class là *injector class*, cầu nối giữa cung - `@Module` và cầu - `@Inject`.
* `@Inject` - đánh dấu đâu là constructor để khởi tạo dependency hoặc đâu là nơi cần dependency.
* `@Module` - đánh dấu một class/interface, nơi cung cấp các dependency.
* `@Provides` - đánh dấu các method nằm bên trong `@Module` và thể hiện cách khởi tạo các dependency.
* `@Qualifier` - định danh để phân biệt các dependency có cùng kiểu dữ liệu với nhau.
* `@Scope` - thể hiện vòng đời (scope) của dependency, từ đó giúp ta tạo ra dependency phù hợp với những trường hợp khác nhau.

Nếu đây là lần đầu tiên (có thể là lần thứ n :D) tiếp cận với một loạt các annotation như thế này, hẳn là chúng ta sẽ thấy bị lạc lối, không biết nên bắt đầu thế nào. Tuy nhiên, chúng ta có thể thực hành với một chương trình nhỏ rồi quay lại nghiền ngẫm lý thuyết thì bức tranh sẽ sáng sủa hơn rất nhiều. Lego, chúng ta sẽ implement *Dagger 2*

# The very first basic program

Chúng ta sẽ có một ứng dụng đơn giản sử dụng mô hình MVP như sau:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/u1dxoya5qb_DaggerMvpBasic.jpg">
</p>

**Note**: Những bạn đã sử dụng mô hình MVP (hoặc đã thấm nhuần tư tưởng của... *Dependency inversion*) chắc sẽ thắc mắc tại sao việc giao tiếp giữa các layer chẳng có interface gì cả!?! Tuy nhiên, mình xin phép bắt đầu với một ứng dụng "cộc lốc" này trước. Sau đó, chúng ta sẽ dần dần trả món "nợ kỹ thuật" này bằng cách implement đầy đủ các interface nhằm thỏa mãn *DIP* để ứng dụng sát với thực tế nhất.

Nhìn vào mối quan hệ giữa các class, ta thấy cần phải build một *dependency graph* với các mối quan hệ sau:
* Presenter là dependency của Activity
* Repository là dependency của Presenter
* ApiHelper, PreferenceHelper và DbHelper là dependency của Repository

Và để bắt đầu, chúng ta sẽ đến với annotation cơ bản đầu tiên trong *Dagger 2*: `@Inject`

### @Inject

Mục đích đầu tiên của `@Inject` mà ta sẽ sử dụng tới là để thêm một class vào *dependency graph*. Để làm điều đó, chúng ta sẽ thêm `@Inject` vào constructor của class.
```
class MainPresenter @Inject constructor(var repository: MainRepository) { ... }
```

Tuy nhiên, sau khi thử build project phát, chúng ta lại gặp lỗi sau:
<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/h33194h6xa_Screenshot%20from%202021-04-08%2009-54-39.png">
  Không provide `MainRepository` thì tao khởi tạo `MainPresenter` bằng niềm tin ah~~
</p>

Error này có kiểu là một `[Dagger/MissingBinding]` error. Giải thích một cách nôm na là: *Dagger* không biết cung cấp `MainRepository` như thế nào vì không tìm thấy nó trong *dependency grapgh*. Bởi vậy, ta thêm `@Inject` cho constructor của `MainRepository`:
```
class MainRepository @Inject constructor(var apiHelper: ApiHelper,
                                         var preferenceHelper: PreferenceHelper,
                                         var dbHelper: DbHelper) { ... }
```

Tương tự với các dependency của `MainRepository`, chúng ta cũng cần thêm `@Inject` vào constructor để  *dependency graph* thêm phần đầy đặn:
```
class ApiHelper @Inject constructor() { ... }
class PreferenceHelper @Inject constructor() { ... }
class DbHelper @Inject constructor() { ... }
```

**Note**: Nhớ lại một chút phần I về [Các kiểu inject](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Cac-khai-niem-co-ban#toc-c-c-ki-u-inject), chúng ta nhận ra rằng chúng ta đang sử dụng *constructor injection*.

Vậy là chúng ta đã hoàn thành việc khai báo những phần tử có mặt trong *dependency graph*. Dây chính là các *service class* đã nói trong phần I. Tiếp theo, chúng ta cần khai báo *injector class*, đó là một class trung gian và có nhiệm vụ kết nối *service class* - nơi cung cấp dependency và *client class* - nơi cần dependency. Để tạo ra một *injector class* trong *Dagger 2*, chúng ta sử dụng annotation `@Component`

### @Component

Component trong *Dagger 2* là một interface được annotate với `@Component`. *Dagger* sẽ sử dụng component và các thông tin chúng ta khai báo thông qua `@Inject` và build lên *dependency graph* thỏa mãn các mối quan hệ mà chúng ta đã khai báo. Bên trong component này, chúng ta có thể khai báo các function trả về các dependency mà chúng ta cần(`MainPresenter`).
```
@Component
interface MainComponent {
    fun mainPresenter(): MainPresenter
}
```

**Note**: *Dagger* không quan tâm tên của function này là gì, nó chỉ cần kiểu mà function này trả về để expose ra đúng function mà nó gen ra.

Tiếp đó, chúng ta cần phải build project để *Dagger* gen code cho chúng ta. Sau khi build xong, ta sẽ thấy code được *Dagger* gen ra trong thư mục `app/build/generated/source`, các bạn có thể đọc để thấy code cũng tương đối dễ hiểu ;). Và class mà chúng ta cần quan tâm là `DaggerMainComponent`. Class này được gen ra từ interface component ở trên với format tên là *Dagger* + *Component name* . Class này sẽ implement interface component và override lại các function mà chúng ta khai báo bên trong interface. Thông qua những function này, chúng ta có thể lấy ra dependency cần thiết mà không cần quan tâm các dependency này được khởi tạo ở đâu và quản lý như thế nào.
```
class MainActivity : FragmentActivity() {

    private lateinit var mainPresenter: MainPresenter

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        // initialize the component
        val mainComponent = DaggerMainComponent.create()

        // ... and get the dependency
        mainPresenter = mainComponent.mainPresenter()
    }
}
```

Tuy nhiên, dependency cũng có dependency this, dependency that, không phải class nào cũng do chúng ta tạo ra hay có kiểu là một class có thể khởi tạo được(interface/abstract class). Bởi vậy, chúng ta cần thêm một "cái kho", nơi chúng ta chỉ cho *Dagger* biết cách khởi tạo các dependency that này. Cái kho đó trong *Dagger 2* gọi là các module.

### @Module

Module trong *Dagger 2* có thể là một class hoặc một abstract class được annotate với `@Module`, nơi chúng ta cung cấp những dependency muốn thêm vào *dependency graph*. Khi build *dependency graph*, Dagger component ngoài tìm kiếm ở những constructor có annotation `@Inject` như chúng ta đã làm ở trên, nó sẽ tìm thêm trong các module được gắn với nó.

**Note**: Có một misconception rằng không có module thì *Dagger 2* không gáy được :|. Tuy nhiên, chương trình đang xét cho ta thấy rằng không nhất thiết cần có các module trong trường hợp dependency đều là những class có thể khởi tạo được thông qua constructor. Chỉ cho *Dagger* biết cách khởi tạo một dependency thông qua module là 1 cách nhưng không phải là duy nhất.

Tiếp tục ví dụ ở trển với một requirement mới, chúng ta cần thêm library *Retrofit* để call API và sử dụng *Gson* để parse object. Bởi vậy, chúng ta sẽ tạo một API service là `MainService` chứa các API của màn hình `MainActivity`. Các bước để config *Retrofit* và tạo `MainService` là:
```
val baseUrl = "..."
val gson = GsonBuilder().setDateFormat("ddMMyyyy").create()
val retrofit = Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create(gson))
            .baseUrl(baseUrl)
            .build()

val mainService: MainService = retrofit.create(MainService::class.java)
```

Ta thấy `Retrofit`, `Gson` và `MainService` đều là những "dependency that" đã được nói tới. Vậy thì tạo ra một module ngay thôi chứ còn chờ đợi chi? Để một class được coi là một module, ta chỉ cần thêm annotation `@Module` vào trước phần khai báo class đó.
```
@Module
class ApiModule { ... }
```

##### @Provides

Bên trong module, chúng ta cần chỉ cho *Dagger* biết cách khởi tạo dependency bằng cách khai báo các function được annotate với `@Provides` và trả về kiểu của dependency mà chúng ta cần. Với đoạn code config *Retrofit* và khởi tạo `MainService` ở trên, chúng ta có thể tách ra thành 4 function riêng rẽ trả về 4 dependency chúng ta mong muốn:`baseUrl`, `gson`, `retrofit` và `mainService` để sau này nếu có chỗ khác cần, code sẽ không bị lặp.
```
@Provides
fun provideRetrofit(baseUrl: String, gson: Gson): Retrofit {
    return Retrofit.Builder()
        .addConverterFactory(GsonConverterFactory.create(gson))
        .baseUrl(baseUrl)
        .build()
}

@Provides
fun provideBaseUrl(): String {
    return "..."
}

@Provides
fun provideGson(): Gson {
    return GsonBuilder().setDateFormat("ddMMyyyy").create()
}

@Provides
fun provideMainService(retrofit: Retrofit): MainService {
    return retrofit.create(MainService::class.java)
}
```

**Note**: Tên của các provide function và thứ tự của các function đó trong module không quan trọng mà quan trọng là kiểu trả về của các function đó, *Dagger* sẽ dựa vào đó mà thêm các class vào *dependency graph*. Trong trường hợp trên: để provide `MainService`, chúng ta cần một object `Retrofit`. Bởi vậy, chúng ta sẽ provide thêm `Retrofit`. Để khởi tạo `Retrofit`, chúng ta lại cần có một `String` và một object `Gson`. Vì thế, chúng ta tiếp tục provide thêm `Gson` và `String`. Vậy là đã thỏa mãn được tất cả các dependency để có thể khởi tạo `MainService`.

Ngoài ra, *Dagger* cho phép chúng ta gắn nhiều module vào một component giúp cho các module đó được thông với nhau nên dependency cung cấp ở module này có thể provide cho dependency ở module kia. Bởi vậy, các bạn nên nhóm các dependency liên quan vào một module để code không bị lặp. VD: `UtilsModule`
```
@Module
class UtilsModule {

    private var mContext: Context

    constructor(context: Context) {
        this.mContext = context
    }

    @Provides
    fun provideContext(): Context {
        return mContext
    }
}
```

Khác với `ApiModule` không có một dependency nào, `UtilsModule` cần một dependency có kiểu `Context`. Tuy nhiên, `Context` lại được tạo ra bởi Android system và do đó object `Context` nên được truyền vào khi khởi tạo module khi nó available.
```
val mainComponent = DaggerMainComponent.builder()
            .utilsModule(UtilsModule(this))
            .build()
mainPresenter = mainComponent.mainPresenter()
```

**Note**: Trong trường hợp module không cần một dependency nào từ bên ngoài:
- Chúng ta không nhất thiết phải tự khởi tạo và truyền vào cho component bởi component sẽ tự khởi tạo ở bên dưới.
- Chúng ta có thể khai báo nó là môt `object` class để module chỉ cần khởi tạo một lần duy nhất.
```
@Module
object ApiModule { ... }
```

##### @Bind

Quay lại với quả [bát họ kỹ thuật](https://buihuycuong.medium.com/technical-debt-n%E1%BB%A3-k%E1%BB%B9-thu%E1%BA%ADt-6a312eb5eb42) đã bốc từ đầu bài viết khi chúng ta chỉ sử dụng concrete type và từ chối abstract type nhằm giảm độ phức tạp của chương trình. Và vui mừng là chúng ta đã có đủ kiến thức để giải quyết vấn đề rồi. It's payback time!

Chúng ta sẽ tạo thêm các interface và sử dụng các interface đấy thay vì các concrete class:
```
interface MainPresenter { ... }
class MainPresenterImpl @Inject constructor(var repository: MainRepository) : MainPresenter { ... }

interface MainRepository { ... }
class MainRepositoryImpl @Inject constructor(var apiHelper: ApiHelper,
                                             var preferenceHelper: PreferenceHelper,
                                             var dbHelper: DbHelper) : MainRepository { ... }
```

Vậy là giờ đây, các dependency là các interface thay vì các class có thể khởi tạo được nên bắt buộc chúng ta phải provide chúng thông qua các module:
```
@Module
object PresenterModule {

    @Provides
    @JvmStatic
    fun provideMainPresenter(mainPresenterImpl: MainPresenterImpl): MainPresenter {
        return mainPresenterImpl
    }
}

@Module
object RepositoryModule {

    @Provides
    @JvmStatic
    fun provideMainRepository(mainRepositoryImpl: MainRepositoryImpl): MainRepository {
        return mainRepositoryImpl
    }
}
```

**Note**: Các dependency còn lại như `ApiHelper`, `PreferenceHelper` và `DbHelper` thì các bạn làm tương tự nhé: `ctrl-c`, `ctrl-vvvvvvvv` :D

Các bạn có thể thấy cách khai báo các dependency này là hoàn toàn giống nhau về format khi chúng ta provide một interface nhưng lại trả về implementation của interface đó. Trong thực tế, cách provide này là cực kỳ phổ biến khi hầu như bất kỳ ứng dụng nào sử dụng *Dagger 2* đều sẽ có những provide function như thế. Bởi vậy, *Dagger* cung cấp cho chúng ta một annotation để giảm những đoạn boilerplate code này đi: `@Binds`
```
@Module
abstract class PresenterModule {

    @Binds
    abstract fun provideMainPresenter(mainPresenterImpl: MainPresenterImpl): MainPresenter
}

@Module
abstract class RepositoryModule {

    @Binds
    abstract fun provideMainRepository(mainRepositoryImpl: MainRepositoryImpl): MainRepository
}
```

**Note**: Các binding function cần phải nằm trong một abstract class module và module này không được chứa lẫn lộn cả binding function và provides function. Đó là vì cách *Dagger* sử dụng thông tin có được từ 2 annotation này để khởi tạo dependency là khác nhau:
- Với `@Provides`, *Dagger* sẽ sử dụng chính function chúng ta khai báo để khởi tạo dependency. Màu vàng ở tên function thể hiện *Dagger* đang sử dụng function:
<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/wu9kw7664c_module_provide_function.png">
</p>

- Với `@Binds`, *Dagger* chỉ lấy thông tin của function: kiểu trả về và kiểu của tham số truyền vào thay vì dùng luôn function. Màu ghi ở tên function thể hiện function đang không được sử dụng ở đâu:
<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/n8qu24f7d0_module_bind_function.png">
</p>


Để hiểu kỹ hơn, các bạn có thể tham khảo [ở đây](https://dagger.dev/dev-guide/faq.html#why-is-binds-different-from-provides).

### @Qualifier

Khi provide các dependency cho *Dagger*, chúng ta có thể gặp phải tình huống 2 dependency khác nhau nhưng có cùng kiểu dữ liệu. Để giải quyết vấn đề này, *Dagger* cung cấp cho chúng ta annotaion `@Qualifier` nhằm phân biệt các dependency với nhau.

Có 2 cách để sử dụng annotaion này:
- Sử dụng một qualifier annotation có sẵn của *Dagger*: `@Named`
- Tự tạo ra một annotation và annotate nó với `@Qualifier`

##### @Named

Trong ví dụ ở trên, giả sử chúng ta cần 2 object `Retrofit` với 2 config khác nhau: `Authentication` và `No-Authentication`. Chúng ta có thể tạo thêm một provide function nữa và thêm `@Named` để phân biệt 2 dependency đó:
```
@Provides
@Named("No-Authentication")
fun provideRetrofitNoAuthentication(baseUrl: String, gson: Gson): Retrofit {
    return Retrofit.Builder()
        .baseUrl(baseUrl)
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build()
}

@Provides
@Named("Authentication")
fun provideRetrofitAuthentication(baseUrl: String, okHttpClient: OkHttpClient, gson: Gson): Retrofit {
    return Retrofit.Builder()
        .baseUrl(baseUrl)
        .addConverterFactory(GsonConverterFactory.create(gson))
        .client(okHttpClient)
        .build()
}

@Provides
fun provideOkHttp(): OkHttpClient { ... }
```

Cùng với đó, ở tất cả những chỗ sử dụng `Retrofit` cũng cần sử dụng `@Named` để chỉ rõ dependency cần dùng là loại nào
```
@Provides
fun provideMainService(@Named("No-Authentication") retrofit: Retrofit): MainService {
    return retrofit.create(MainService::class.java)
}
```

##### Tạo ra một qualifier annotation mới

Để tạo ra một qualifier annotation, chúng ta chỉ cần tạo ra một annotation và annotate nó với `@Qualifier`
```
@Qualifier
@MustBeDocumented
@kotlin.annotation.Retention(AnnotationRetention.RUNTIME)
annotation class AuthenticationRetrofit()

@Qualifier
@MustBeDocumented
@kotlin.annotation.Retention(AnnotationRetention.RUNTIME)
annotation class NoAuthenticationRetrofit()
```

Cách sử dụng annotation mới này chỉ đơn giản là chúng ta thay thế `@Named` bằng annotation mới:
```
@Provides
@AuthenticationRetrofit
fun provideRetrofitNoAuthentication(baseUrl: String, gson: Gson): Retrofit { ... }

@Provides
@NoAuthenticationRetrofit
fun provideRetrofitAuthentication(baseUrl: String, okHttpClient: OkHttpClient, gson: Gson): Retrofit { ... }

@Provides
fun provideMainService(@NoAuthenticationRetrofit retrofit: Retrofit): MainService { ... }
```

### Property injection

Trong phần I của series, chúng ta biết có [3 cách để inject](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles#toc-c-c-ki-u-inject) một dependency. Tuy nhiên từ đầu bài viết, chúng ta mới chỉ sử dụng 1 cách inject duy nhất. Đó là *constructor injection*. Cách inject này thì đơn giản và được khuyên dùng, nhưng chúng ta lại không thể sử dụng nó đối với những class mà việc khởi tạo không phải do chúng ta đảm nhiệm: `Activity`, `Fragment`, `Service`, etc. Ngoài ra, cách lấy dependency từ *Dagger component* ra còn một vấn đề là khi số lượng dependency tăng lên, chúng ta cũng phải nhớ mà lấy ra đầy đủ trước khi sử dụng để tránh làm chương trình crash. Để giải quyết điều này, chúng ta sẽ sử dụng đến một cách inject khác: *property injection*.

Đầu tiên, chúng ta cần annotate dependency bằng `@Inject` và để access modifier của dependency là package-private trở lên - `internal` hoặc `public` trong *Kotlin* (vì cơ chế của cách inject này thực chất là component gán giá trị trực tiếp cho dependency):
```
@Inject
lateinit var mainPresenter: MainPresenter
```

Tiếp đó, chúng ta cần khai báo thêm một function vào component để *Dagger* biết chúng ta muốn inject dependency vào class nào. Kiểu của tham số truyền vào function sẽ là class muốn được inject. Nếu có thêm class muốn được inject, chúng ta cần khai báo thêm các function tương tự với kiểu của tham số tương ứng.
```
@Component(modules = [UtilsModule::class, PresenterModule::class, RepositoryModule::class,  ApiModule::class])
interface MainComponent {
    fun inject(mainActivity: MainActivity)
}
```

**Note**: Đến đây, chúng ta thấy sẽ có 2 cách để giao tiếp với *dependency graph*:
- Khai báo một function không có tham số và trả về một class mà bạn muốn trực tiếp lấy ra.
```
fun mainPresenter(): MainPresenter
```
- Khai báo một function không trả về giá trị nào và có một tham số là class mà bạn muốn inject các dependecy bằng *property injection*
```
fun inject(mainActivity: MainActivity)
```

Cuối cùng, thay vì lấy dependency ra từ component, chúng ta gọi function vừa được khai báo trong component kia để *Dagger* inject tất cả các dependency mà đã được annotate với `@Inject`:
```
// initialize the component
val mainComponent = DaggerMainComponent.create()

// inject dependencies to this class
mainComponent.inject(this)
```

**Note**: Khi sử dụng *property inject* với Activity, việc khởi tạo vào inject nên được thực hiện trước `super.onCreate()` để tránh gặp phải issue restore của Fragment vì khi `super.onCreate()` được thực hiện, Activity có thể attach Fragment và các Fragment thì lại muốn truy cập đến các member của Activity. Bởi vậy, chúng ta nên follow best practice sau:
- Với Activity, inject Dagger bên trong method `onCreate()` nhưng trước khi gọi `super.onCreate()`.
- Với Fragment, inject Dagger bên trong method `onAttach()` nhưng sau khi gọi `super.onAttach()`.

### Cùng nhìn lại

Cuối cùng, sau khi đã hoàn thành việc khai báo các dependency không mấy khó khăn, phần việc nhàm chán còn lại là khởi tạo và quản lý các dependency, *Dagger* sẽ lo hết cho chúng ta. Diagram dưới đây sẽ thể hiển mối quan hệ giữa các thành phần trong *Dagger*:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/n2ii3pdtil_Dagger2_component.jpg">
</p>

Gòi xong, chúng ta đã đi qua hầu hết các kiến thức cơ bản của *Dagger 2*, hy vọng với những kiến thức nhiêu đây, bạn đã có thể bắt đầu sử dụng *Dagger 2* mà không cảm thấy lạc lối nữa. Tuy chương trình trên đây chỉ là một chương trình nhỏ, chúng ta có thể sẽ chưa thấy hết được sức mạnh của *Dagger 2* khi các dependency là chưa nhiều. Tuy nhiên, với một ứng dụng phức tạp hơn với nhiều màn hình hơn, mỗi màn hình sẽ sử dụng một loạt các dependency kèm theo thì việc khởi tạo và quản lý sẽ rất mất thời gian khi phải viết rất nhiều những đoạn boilerplate code và còn dễ gây ra lỗi nữa. *Dagger 2* chính là "the right tool" giúp chúng ta loại bỏ mốí quan tâm đấy và tập trung vào các phần quan trọng hơn của chương trình.
