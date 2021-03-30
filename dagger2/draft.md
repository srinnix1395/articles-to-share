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
