[Android] Dagger 2 - Phần II: Into the Dagger 2

Bài viết là phần thứ 2 của series bài học vỡ lòng về Dagger 2. Nếu bạn chưa đọc phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-1--Cac-khai-niem-co-ban)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Các khái niệm cơ bản](https://kipalog.com/posts/Android--Dagger-2---Phan-1--Cac-khai-niem-co-ban)
2. [Android] Dagger 2 - Phần II: Into the Dagger 2 (Bạn đang ở đây)
3. [[Android] Dagger 2 - Phần III: Custom scope trong dagger 2]()

# Trong bài học trước...

Chúng ta đã nói một chút về việc khởi tạo và quản lý dependency. Tiếp đó, chúng ta ngờ ngợ ra những vấn đề khi ứng dụng được scale up lên. Cuối cùng, chúng ta tìm thấy những design principle và design pattern có thể giải quyết giả thiết của bài toán ban đầu.

# Đi vào bài học hôm nay...
Chúng ta sẽ tìm hiểu sâu hơn về *Dagger 2* - công cụ sẽ giúp chúng ta ...TODO

TODO: Ảnh trên unsplash

Một chút kiến thức lịch sử, *Dagger* là một library được Square tạo ra để implement *dependency injection* trong *Java* (Android là một trường hợp cụ thể hơn). *Dagger 1* là một *dynamic, run-time DI framework* và đã deprecated. *Dagger 1* khởi tạo các dependency "động", tức là việc tạo ra dependency được thực hiện lúc run-time thông qua java reflection. Bởi vậy, nó có nhược điểm là chậm và sẽ có run-time exception xảy ra khi chạy ứng dụng.

*Dagger 2* được tiếp nối bởi Google và là một *fully static, compile-time DI framework*. Để khắc phục những nhược điểm của người tiền nhiệm, *Dagger 2* sử dụng *annotation processor* (a code generator using annotation) để "viết" code cho chúng ta khi compile. Bởi vậy, nếu có lỗi gì, app sẽ không thể run được. Cùng với đó, nguyên tắc để gen ra các đoạn code này là cố gắng bắt chước những đoạn code mà người dùng thực sự sẽ viết. Từ đó, code cũng sẽ đơn giản và dễ trace hơn.

Để bắt đầu, chúng ta cần nói qua về một khái niệm quan trọng trong *Dagger* mà chúng ta hay được nghe tới nhưng lại ít được giải thích tường minh. Đó là *dependency graph*.

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
        println("Using a pen to learning ${book.getSubjectName()}")
    }
}

data class Pen (val ink: Ink)

data class Ink(val color: Int = Color.BLACK)
```

... và đây là *dependency graph* tương ứng:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/p2g5hyoqsd_Screenshot%20from%202021-03-25%2017-47-37.png">
</p>

Ở đây, mỗi mũi tên có hướng trong đồ thị biểu diễn một mối quan hệ phụ thuộc: `Student`phụ thuộc vào `TextBook` và `Pen`, `Pen` phụ thuộc vào `Ink`. Vì những sự phụ thuộc này, ta cần khởi tạo các module cấp thấp trước rồi mới có thể khởi tạo được các module cấp cao: khởi tạo `Ink` trước rồi mới đến `Pen`. Và cũng bởi vì cần có thứ tự khởi tạo trước sau như vậy, *dependency graph* không thể tồn tại những vòng lặp đóng hay *a circular dependency* vì *Dagger* sẽ không biết đâu là điểm khởi tạo đầu tiên.

Khi implement *Dagger*, chúng ta cần fulfill *dependency graph* trước khi chạy chương trình và giao phần việc còn lại cho *Dagger*. VD: để khởi tạo `Pen`, ta cần khởi tạo `Ink` trước. Nhưng nếu ta chưa thêm `Ink` vào *dependency graph*, *Dagger* sẽ không biết khởi tạo `Ink` như thế nào.

Vậy, làm thế nào để fulfill *dependency graph* trong *Dagger 2*? Câu trả lời là *Dagger 2* sẽ cung cấp cho chúng ta các annotation để làm việc đó.

# Annotation trong Dagger 2

*Annotation* là một dạng **chú thích** hoặc một dạng **metadata** được dùng để cung cấp thông tin cho mã nguồn *Java*. *Dagger 2* sẽ sử dụng các thông tin có được thông qua truy vấn các annotation để gen code khi compile.

Các annotation cơ bản trong *Dagger 2* là:
* `@Component` - đánh dấu một interface (dependency graph) là cầu nối giữa cung - `@Module` và cầu - `@Inject`.
* `@Inject` - đánh dấu đâu là nơi cần dependency.
* `@Module` - đánh dấu một class, nơi cung cấp các dependency
* `@Provides` - đánh dấu các method nằm bên trong `@Module` và thể hiện cách khởi tạo các dependency.
* `@Scope` - thể hiện vòng đời (scope) của dependency, từ đó giúp ta tạo ra dependency phù hợp với những trường hợp khác nhau.
* `@Qualifier` - định danh để phân biệt các dependency có cùng kiểu dữ liệu với nhau.

Gòi xong, vậy là lý thuyết về *Dagger 2* đã xong, chỉ có vậy à. Mình cá là các bạn vẫn chưa hiểu gì đâu :D. Đùa chút chứ cái này phải thực hành rồi từ đó nghiền ngẫm lại lý thuyết thì mới vỡ ra được. Vậy thì chúng ta sẽ tới ngay với phần thực hành implement *Dagger 2*

# The very first basic program

Chúng ta sẽ có một ứng dụng đơn giản sử dụng mô hình MVP như sau:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/u1dxoya5qb_DaggerMvpBasic.jpg">
</p>

**Note**: Những bạn đã sử dụng mô hình MVP (hoặc đã thấm nhuần tư tưởng của... *Dependency inversion*) chắc sẽ thắc mắc tại sao việc giao tiếp giữa các layer chẳng có interface gì cả!?! Tuy nhiên, mình xin phép bắt đầu với một ứng dụng "cộc lốc" này trước. Sau đó, chúng ta sẽ dần dần trả món "nợ kỹ thuật" này bằng cách implement đầy đủ để nó thỏa mãn *DIP* để ứng dụng gần với thực tế nhất để các bạn có thể tham khảo.

Nhìn vào mối quan hệ giữa các class, ta thấy cần phải xây dựng một *dependency graph* với các mối quan hệ sau:
* Presenter là dependency của Activity
* Repository là dependency của Presenter
* ApiHelper, PreferenceHelper và DbHelper là dependency của Repository

Để thêm một class vào *dependency graph*, chúng ta sử dụng annotation `@Inject`

### @Inject

Chúng ta sẽ thêm `@Inject` vào constructor của class muốn thêm vào *dependency graph*
```
class UserPresenter @Inject constructor(var repository: DemoRepository) { ... }
```

Tuy nhiên, vì `UserPresenter` cần `UserRepository` để có thể khởi tạo nên chúng ta phải thêm `@Inject` cả vào constructor của `UserRepository` nữa. Nếu không, khi compile, *Dagger* sẽ báo lỗi rằng:"`repository` không được provide nên không biết khởi tạo thế nào!"
```
class UserRepository @Inject constructor(var apiHelper: ApiHelper,
                                         var preferenceHelper: PreferenceHelper,
                                         var dbHelper: DbHelper) { ... }
```

Tương tự, chúng ta cũng cần thêm `@Inject` vào các dependency của `UserRepository`
```
class ApiHelper @Inject constructor() { ... }
class PreferenceHelper @Inject constructor() { ... }
class DbHelper @Inject constructor() { ... }
```

**Note**: Nhớ lại một chút phần I về [Các kiểu inject](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Cac-khai-niem-co-ban#toc-c-c-ki-u-inject), ta thấy đây là kiểu inject bằng constructor.

Vậy là chúng ta đã hoàn thành việc khai báo những phần tử có mặt trong *dependency graph*. Liên hệ với phần I, đó là các *service class*. Tiếp theo, chúng ta cần khai báo *injector class*, đó là một class trung gian và có nhiệm vụ inject *service class* vào *client class*. Để tạo ra một *injector class*, chúng ta sử dụng annotation `@Component`

### @Component

Component trong *Dagger 2* là một interface được annotate với `@Component`. *Dagger* sẽ sử dụng component và các thông tin chúng ta khai báo thông qua `@Inject` và build lên *dependency graph* thỏa mãn các mối quan hệ mà chúng ta đã khai báo. Bên trong component này, chúng ta có thể khai báo các function trả về các dependency mà chúng ta muốn lấy ra.
```
@Component
interface UserComponent {
    fun userPresenter(): UserPresenter
}
```

**Note**: tên của function này không quan trọng mà quan trọng là kiểu mà function này trả về.

Tiếp đó, chúng ta cần phải build project thì *Dagger* mới gen code ra cho chúng ta từ những thông tin trên. Từ đó, ta mới có thể truy cập được các dependency mà chúng ta đã khai báo. Sau khi build xong, ta sẽ thấy code được *Dagger* gen ra trong thư mục `app/build/generated/source`, các bạn có thể đọc để thấy code cũng tương đối dễ hiểu ;). Và class mà chúng ta cần quan tâm là `DaggerUserComponent`. Class này được gen ra từ interface component ở trên với format tên là *Dagger* + *Component name* . Class này sẽ implement component mà ta đã khai báo, đồng thời chứa các function override lại các function cung cấp dependency bên trong interface. Thông qua những function này, chúng sẽ có thể lấy ra dependency mà chúng ta cần mà không cần quan tâm các dependency này được khởi tạo ở đâu và quản lý như thế nào.
```
class UserActivity : FragmentActivity() {

    private lateinit var mUserPresenter: UserPresenter

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        val userComponent = DaggerUserComponent.create()
        mUserPresenter = userComponent.userPresenter()
    }
}
```

Tuy nhiên, không phải dependency nào cũng có kiểu là một class mà chúng ta viết ra hay có kiểu là một class có thể khởi tạo được. Bởi vậy, chúng ta cần một "cái kho" khác, nơi chúng ta sẽ cung cấp cho *Dagger* cách khởi tạo các dependency. Cái kho đó trong *Dagger 2* gọi là các module.

### @Module

Module trong *Dagger 2* có thể là một class hoặc một abstract class, nơi ta cung cấp những dependency ta muốn thêm vào *dependency graph*. Khi build *dependency graph*, Dagger component ngoài tìm kiếm ở những chỗ ta để annotation `@Inject`, nó sẽ tìm thêm trong các module được gắn với nó để thỏa mãn các dependency yêu cầu.

**Note**: Có một hiểu nhầm rằng *Dagger 2* bắt buộc cần các module mới có thể hoạt động. Tuy nhiên, như chúng ta đã thấy ở trên, chúng ta không cần khai báo một module để cung cấp các dependency mà có thể khởi tạo trực tiếp các dependency đó thông qua constructor.

Tiếp tục ví dụ ở trên, chúng ta sẽ thêm library *Retrofit* để call API và sử dụng *Gson* để parse object. Ở đây, ta sẽ có một API service là `UserServices` chứa các API liên quan đến user. Các bước để config *Retrofit* và tạo `UserServices` là
```
val baseUrl = "https://api.github.com/"
val gson = GsonBuilder().setDateFormat("ddMMyyyy").create()
val retrofit = Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create(gson))
            .baseUrl(baseUrl)
            .build()

val userServices: UserServices = retrofit.create(UserServices::class.java)
```

Ta thấy `Retrofit`, `Gson` là các class không phải do chúng ta tạo ra còn `UserServices` là một interface nên để provide những dependency này cho class `ApiHelper`, chúng ta cần tạo một module. Để một class được coi là một module, ta chỉ cần thêm annotation `@Module` vào trước class đó.
```
@Module
class ApiModule {

    @Provides
    fun provideRetrofit(baseUrl: String, gson: Gson): Retrofit {
        return Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create(gson))
            .baseUrl(baseUrl)
            .build()
    }

    @Provides
    fun provideBaseUrl(): String {
        return "https://api.github.com/"
    }

    @Provides
    fun provideGson(): Gson {
        return GsonBuilder().setDateFormat("ddMMyyyy").create()
    }

    @Provides
    fun provideUserServices(retrofit: Retrofit): UserServices {
        return retrofit.create(UserServices::class.java)
    }
}
```

Ngoài ra, với mỗi function mà ta muốn báo cho *Dagger* biết là ta muốn thêm một class vào *dependency graph*, ta cần sử dụng thêm annotation `@Provides`. Tên của các provide function và thứ tự của các function đó trong module không quan trọng mà quan trọng là kiểu trả về của các function đó, *Dagger* sẽ dựa vào đó mà thêm các class đó vào *dependency graph*. Trong trường hợp trên: để provide `UserServices`, chúng ta cần một object `Retrofit`. Bởi vậy, ta sẽ provide cho *Dagger* `Retrofit`. Để khởi tạo `Retrofit`, chúng ta lại cần có một `String` và một object `Gson`. Vì thế, chúng ta tiếp tục provide cho *Dagger* cả `Gson` và `String`.

Cuối cùng, chúng ta nối `ApiModule` với component để component biết cần tìm các dependency ở đâu.
```
@Component(modules = [ApiModule::class])
interface UserComponent {

    fun userPresenter(): UserPresenter
}
```

Tương tự như vậy, chúng ta có thể tạo thêm những module khác





























Với một project Android, ta sẽ cần nhiều dependency khác nhau với các vòng đời khác nhau:
* "Global" singleton là những dependency tồn tại trong toàn bộ vòng đời của ứng dụng: *Context* hay các utility class mà cần được sử dụng ở nhiều nơi trong ứng dụng.
* "Local" singleton là những dependency tồn tại trong các module nhỏ hơn: phụ thuộc vào vòng đời của activity/fragment hay tồn tại trong một phiên đăng nhập của user.

Ở phần 1 này, chúng ta sẽ đến với phần khởi tạo và quản lý "global" singleton bằng Dagger 2. Từ đó, chúng ta sẽ hiểu hơn cách Dagger 2 build ra được dependency graph.

### Module

Các module sẽ tồn tại trong toàn bộ vòng đời của ứng dụng là:

* ApplicationModule
```
@Module
class ApplicationModule(private val application: Application) {

    @Provides
    @ApplicationContext
    fun provideApplicationContext(): Context {
        return application
    }
}
```

* ApiModule
```
@Module
class ApiModule {

    companion object {
        const val ISO_8601_DATE_TIME_FORMAT_RECEIVE = "yyyy-MM-dd'T'HH:mm:ssZZ"
    }

    @Provides
    @Singleton
    fun provideGsonConverterFactory(): GsonConverterFactory {
        val gson = GsonBuilder()
            .setDateFormat(ISO_8601_DATE_TIME_FORMAT_RECEIVE)
            .serializeNulls()
            .create()
        return GsonConverterFactory.create(gson)
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        val httpClient = OkHttpClient.Builder()

        if (BuildConfig.DEBUG) {
            val logging = HttpLoggingInterceptor()
            logging.level = HttpLoggingInterceptor.Level.BODY
            httpClient.addInterceptor(logging)
        }

        return httpClient.build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(gsonConverterFactory: GsonConverterFactory, okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(Constant.BASE_URL)
            .addConverterFactory(gsonConverterFactory)
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .client(okHttpClient)
            .build()
    }
}
```

Như ở phần lý thuyết, ta sử dụng *@Module* để đánh dấu một class là nơi cấp các dependency và sử dụng *@Provides* để đánh dấu các method cung cấp dependency. Đối với các method này, phần tên của method là không quan trọng mà phần quan trọng là kiểu trả về của các method. Khi một module cần một dependency, Dagger sẽ tìm những dependency này trong các module dựa vào kiểu dữ liệu trả về của các provide method. Nếu trong một module mà có 2 method cùng cung cấp 1 kiểu dữ liệu thì cần có *@Qualifier* để Dagger có thể phẩn biệt được: *@ApplicationContext* ở *ApplicationModule*.

Ta có thể thấy ở method `provideRetrofit`, ta cần 2 tham số để khởi tạo *Retrofit* là *GsonConverterFactory* và *OkHttpClient*. Vì vậy, ta khai báo 2 tham số tương ứng là `gsonConverterFactory` và `okHttpClient`. Cùng với đó, ta cũng cần viết các provide method mà cung cấp 2 kiểu dữ liệu đó: `provideGsonConverterFactory` và `provideOkHttpClient` (nếu không Dagger sẽ chẳng biết tìm 2 tham số này ở đâu cả ~~).

Thêm một ý nữa, ta có thể thấy annotation *@Singleton*. Đây là một annotation dùng để #todo

### Component

```
@Singleton
@Component(modules = [ApplicationModule::class, ApiModule::class])
interface AppComponent {

}
```

Tiếp theo, chúng ta khai báo `AppComponent` - cầu nối giữa *@Module* và *@Inject*, bằng cách sử dụng annotation *@Component*. Khi khai báo như trên, component này bao gồm 2 module là `ApplicationModule` và `ApiModule`. Điều này có nghĩa là ở class nào mà inject component này vào, class đó có thể yêu cầu được component này cung cấp các dependency mà được khai báo trong 2 module kia.
