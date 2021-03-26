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

Một chút kiến thức lịch sử, *Dagger* là một library được Square tạo ra để implement *dependency injection* trong *Java* (Android là một trường hợp cụ thể hơn). *Dagger 1* là một *dynamic, run-time DI framework* và đã deprecated. *Dagger 1* khởi tạo các dependency "động", tức là việc tạo ra dependency được thực hiện lúc run-time thông qua java reflection. Bởi vậy, nó có nhược điểm là chậm và ứng dụng có thể bị crash khi chạy.

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


**Note**: Những bạn đã sử dụng mô hình MVP rồi chắc sẽ thắc mắc sao chẳng có interface



















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
