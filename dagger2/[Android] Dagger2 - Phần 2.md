[Android] Dagger 2 - Phần 2: Into the Dagger 2

Bài viết là phần thứ 2 của series bài học vỡ lòng về Dagger 2. Nếu bạn chưa đọc phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-1--Cac-khai-niem-co-ban)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần 1: Các khái niệm cơ bản](https://kipalog.com/posts/Android--Dagger-2---Phan-1--Cac-khai-niem-co-ban)
2. [Android] Dagger 2 - Phần 2: Into the Dagger 2 (Bạn đang ở đây)
3. [[Android] Dagger 2 - Phần 3: Custom scope trong dagger 2]()

# Trong bài học trước...

Chúng ta đã đi qua





# Dagger












> Dagger 2 is a library which helps the developer to implement a pattern of Dependency Injection (one specific form of Inversion of control).

Dagger là một library được Square tạo ra để implement DI trong Android. Hiện tại, Dagger có 2 version:
* Dagger 1 là một *dynamic, run-time DI framework* được Square viết và đã deprecated. Dagger 1 khởi tạo các dependency "động", tức là việc tạo ra dependency được thực hiện lúc run-time bằng cách sử dụng reflection. Bởi vậy, nó có nhược điểm là reflection thì chậm và app có thể bị crash khi chạy.
* Dagger 2 là một *fully static, compile-time DI framework* được maintain bởi Google. Để khắc phục những nhược điểm của Dagger 1, Dagger 2 không sử dụng reflection để gen code lúc run-time nữa mà sử dụng *annotation processor* (a code generator using annotation) để "viết" code cho chúng ta khi compile. Bởi vậy, nếu có lỗi gì, app sẽ không thể run được. Cùng với đó, nguyên tắc để gen ra các đoạn code này là cố gắng bắt chước những đoạn code mà người dùng thực sự sẽ viết. Từ đó, code cũng sẽ đơn giản và dễ trace.

#### Annotation trong Dagger 2

*Annotation* là một class chứa các metadata của các class, các method, các field hoặc thậm chí là các annotation khác. Từ đó, *Dagger 2* dựa vào các thông tin có được từ các annotation để "viết" code khi compile. Các annotation cơ bản trong *Dagger 2* là:
* *@Component* - đánh dấu một interface (dependency graph) là cầu nối giữa cung - *@Module* và cầu - *@Inject*.
* *@Inject* - đánh dấu "đâu" là nơi "cần một dependency".
* *@Module* - đánh dấu một class, nơi "cung cấp các dependency"
* *@Provides* - đánh dấu các method nằm bên trong *@Module* và thể hiện "cách khởi tạo các dependency".
* *@Scope* - thể hiện vòng đời (scope) của các dependency, từ đó giúp ta tạo ra các global singleton hoặc local singleton.
* *@Qualifier* - annotation này giúp phân biệt các dependency có cùng kiểu dữ liệu với nhau.

Trong đó, 3 annotation đầu tiên là 3 annotation quan trọng nhất mà chúng ta cần phải nhớ để implement Dagger 2. Các annotation còn lại sẽ không phải là vấn đề nếu ta hiểu rõ cách hoạt động của *Dagger 2* từ 3 annotation đầu tiên.

# Thực hành

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

Chúng ta đã hoàn thành
