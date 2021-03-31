[Android] Dagger 2 - Phần II: Into the Dagger 2

Bài viết là phần thứ 2 của series bài học vỡ lòng về *Dagger 2*. Nếu bạn chưa đọc phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Cac-khai-niem-co-ban)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Các khái niệm cơ bản](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Cac-khai-niem-co-ban)
2. [Android] Dagger 2 - Phần II: Into the Dagger 2 (Bạn đang ở đây)
3. [[Android] Dagger 2 - Phần III: Custom scope trong dagger 2]()

# Trong bài học trước...

Chúng ta đã nói một chút về việc khởi tạo và quản lý dependency. Tiếp đó, chúng ta ngờ ngợ ra những vấn đề khi ứng dụng được scale up lên. Cuối cùng, chúng ta được giác ngộ với những design principle và design pattern có thể giải quyết giả thiết của bài toán ban đầu.

# Đi vào bài học hôm nay...
Chúng ta sẽ tìm hiểu sâu hơn về *Dagger 2*: trước là lý thuyết và sau là từng bước implement một chương trình đơn giản.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/29vjyokjrl_joshua-woroniecki-6YHlHIVROzg-unsplash.jpg">
  Into the woods but it won't be long<br>Photo by <a href="https://unsplash.com/@joshua_j_woroniecki?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Joshua Woroniecki</a> on <a href="https://unsplash.com/s/photos/sunshine-pine-forest?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
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

**Note**: Những bạn đã sử dụng mô hình MVP (hoặc đã thấm nhuần tư tưởng của... *Dependency inversion*) chắc sẽ thắc mắc tại sao việc giao tiếp giữa các layer chẳng có interface gì cả!?! Tuy nhiên, mình xin phép bắt đầu với một ứng dụng "cộc lốc" này trước. Sau đó, chúng ta sẽ dần dần trả món "nợ kỹ thuật" này bằng cách implement đầy đủ để nó thỏa mãn *DIP* để ứng dụng gần với thực tế nhất để các bạn có thể tham khảo.

Nhìn vào mối quan hệ giữa các class, ta thấy cần phải build một *dependency graph* với các mối quan hệ sau:
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

Component trong *Dagger 2* là một interface được annotate với `@Component`. *Dagger* sẽ sử dụng component và các thông tin chúng ta khai báo thông qua `@Inject` và build lên *dependency graph* thỏa mãn các mối quan hệ mà chúng ta đã khai báo. Bên trong component này, chúng ta có thể khai báo các function trả về các dependency mà chúng ta cần(`UserPresenter`).
```
@Component
interface UserComponent {
    fun userPresenter(): UserPresenter
}
```

**Note**: tên của function này không quan trọng mà quan trọng là kiểu mà function này trả về.

Tiếp đó, chúng ta cần phải build project để *Dagger* gen code cho chúng ta. Sau khi build xong, ta sẽ thấy code được *Dagger* gen ra trong thư mục `app/build/generated/source`, các bạn có thể đọc để thấy code cũng tương đối dễ hiểu ;). Và class mà chúng ta cần quan tâm là `DaggerUserComponent`. Class này được gen ra từ interface component ở trên với format tên là *Dagger* + *Component name* . Class này sẽ implement interface component và override lại các function mà chúng ta khai báo bên trong interface. Thông qua những function này, chúng ta có thể lấy ra dependency cần thiết mà không cần quan tâm các dependency này được khởi tạo ở đâu và quản lý như thế nào.
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

Tuy nhiên, dependency cũng có dependency this, dependency that, không phải class nào cũng do chúng ta tạo ra hay có kiểu là một class có thể khởi tạo được(interface/abstract class). Bởi vậy, chúng ta cần thêm một "cái kho", nơi chúng ta chỉ cho *Dagger* biết cách khởi tạo các dependency that này. Cái kho đó trong *Dagger 2* gọi là các module.

### @Module

Module trong *Dagger 2* có thể là một class hoặc một abstract class, nơi chúng ta cung cấp những dependency ta muốn thêm vào *dependency graph*. Khi build *dependency graph*, Dagger component ngoài tìm kiếm ở những constructor có annotation `@Inject` như chúng ta đã làm ở trên, nó sẽ tìm thêm trong các module được gắn với nó.

**Note**: Có một misconception rằng không có module thì *Dagger 2* không gáy được :|. Tuy nhiên, chương trình đang xét cho ta thấy rằng không nhất thiết cần có các module trong trường hợp dependency đều là những class có thể khởi tạo được thông qua constructor. Chỉ cho *Dagger* biết cách khởi tạo một dependency thông qua module là 1 cách nhưng không phải là duy nhất.

Tiếp tục ví dụ ở trển với một requirement mới, chúng ta cần thêm library *Retrofit* để call API và sử dụng *Gson* để parse object. Bởi vậy, chúng ta sẽ tạo một API service là `UserServices` chứa các API liên quan đến user. Các bước để config *Retrofit* và tạo `UserServices` là:
```
val baseUrl = "https://api.github.com/"
val gson = GsonBuilder().setDateFormat("ddMMyyyy").create()
val retrofit = Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create(gson))
            .baseUrl(baseUrl)
            .build()

val userServices: UserServices = retrofit.create(UserServices::class.java)
```

Ta thấy `Retrofit`, `Gson` và `UserServices` đều là những "dependency that" đã được nói tới. Bởi vậy, tạo ra một module thôi chứ còn gì!?! Để một class được coi là một module, ta chỉ cần thêm annotation `@Module` vào trước phần khai báo class đó.
```
@Module
class ApiModule { ... }
```

##### @Provides

Bên trong module, chúng ta cần chỉ cho *Dagger* biết cách khởi tạo dependency bằng cách khai báo các function được annotate với `@Provides` và trả về kiểu của dependency mà chúng ta cần. Với đoạn code config *Retrofit* và khởi tạo `UserServices` ở trên, chúng ta có thể tách ra thành 4 function riêng rẽ trả về 4 dependency chúng ta mong muốn:`baseUrl`, `gson`, `retrofit` và `userServices` để sau này nếu có chỗ khác cần, code sẽ không bị lặp.
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
```

**Note**: Tên của các provide function và thứ tự của các function đó trong module không quan trọng mà quan trọng là kiểu trả về của các function đó, *Dagger* sẽ dựa vào đó mà thêm các class vào *dependency graph*. Trong trường hợp trên: để provide `UserServices`, chúng ta cần một object `Retrofit`. Bởi vậy, ta sẽ provide cho *Dagger* `Retrofit`. Để khởi tạo `Retrofit`, chúng ta lại cần có một `String` và một object `Gson`. Vì thế, chúng ta tiếp tục provide cho *Dagger* cả `Gson` và `String`. Bởi vậy, miễn là ta satisfy các dependency đầy đủ là được.

Ngoài ra, *Dagger* cho phép chúng ta gắn nhiều module vào một component giúp cho các module đó lại được thông với nhau nên dependency cung cấp ở module này có thể provide cho dependency ở module kia. Bởi vậy, các bạn nên nhóm các dependency liên quan vào một module để code không bị lặp. VD: `UtilsModule`
```
@Module
class UtilsModule {

    private lateinit var mContext: Context

    constructor(context: Context) {
        this.mContext = context
    }

    @Provides
    fun provideContext(): Context {
        return mContext
    }
}
```

Khác với `ApiModule` không có một dependency nào, `UtilsModule` cần một dependency có kiểu `Context` nên chúng ta cần truyền vào từ bên ngoài khi khởi tạo module và gán nó cho component. Chúng ta cần làm điều này bởi vì `Context` không thể được provide ở đâu khác ngoài lấy ra từ `Application`, `Activity`, etc. Với những module mà không cần một dependency từ bên ngoài, chúng ta không nhất thiết phải tự khởi tạo và truyền vào cho component bởi component sẽ tự khởi tạo ở bên dưới.
```
val userComponent = DaggerUserComponent.builder()
            .utilsModule(UtilsModule(this))
            .build()
mUserPresenter = userComponent.userPresenter()
```

**Note**: trong trường hợp module không cần một dependency nào từ bên ngoài, ta có thể khai báo nó là môt `object` class để module chỉ cần khởi tạo một lần duy nhất.
```
@Module
object ApiModule { ... }
```

##### @Bind

Quay lại với quả [bát họ kỹ thuật](https://buihuycuong.medium.com/technical-debt-n%E1%BB%A3-k%E1%BB%B9-thu%E1%BA%ADt-6a312eb5eb42) đã bốc từ đầu bài viết khi chúng ta chỉ sử dụng concrete type và từ chối abstract type nhằm giảm độ phức tạp của chương trình. Và vui mừng là chúng ta đã có "the right tool" để giải quyết vấn đề rồi. It's payback time!

Chúng ta sẽ tạo thêm các interface và sử dụng các interface đấy thay vì các concrete class:
```
interface UserPresenter { ... }
class UserPresenterImpl @Inject constructor(var repository: UserRepository) : UserPresenter { ... }

interface UserRepository { ... }
class UserRepositoryImpl @Inject constructor(var apiHelper: ApiHelper,
                                             var preferenceHelper: PreferenceHelper,
                                             var dbHelper: DbHelper) : UserRepository { ... }
```

Vậy là giờ đây, các dependency là các interface thay vì các class có thể khởi tạo được nên bắt buộc chúng ta phải provide chúng thông qua các module
```
@Module
object PresenterModule {

    @Provides
    @JvmStatic
    fun provideUserPresenter(userPresenterImpl: UserPresenterImpl): UserPresenter {
        return userPresenterImpl
    }
}

@Module
object RepositoryModule {

    @Provides
    @JvmStatic
    fun provideUserRepository(userRepositoryImpl: UserRepositoryImpl): UserRepository {
        return userRepositoryImpl
    }
}
```

**Note**: Các dependency còn lại như `ApiHelper`, `PreferenceHelper` và `DbHelper` thì các bạn làm tương tự nhé: `ctrl-c`, `ctrl-vvvvvvvv` :D

Các bạn có thể thấy cách khai báo các dependency này là hoàn toàn giống nhau khi chúng ta provide một interface và trả về implementation của interface đó. Đây có thể coi là một đoạn code lặp mà chúng ta thì ngày càng lười :D. Bởi vậy, *Dagger*  lại phải cung cấp thêm một annotation nữa cho chúng ta: `@Bind`
```
@Module
abstract class PresenterModule {

    @Binds
    abstract fun provideUserPresenter(userPresenterImpl: UserPresenterImpl): UserPresenter
}

@Module
abstract class RepositoryModule {

    @Binds
    abstract fun provideUserRepository(userRepositoryImpl: UserRepositoryImpl): UserRepository
}
```

**Note**: Các binding function cần phải nằm trong một abstract class module và module này không được chứa lẫn lộn cả binding function và provides function. Đó là vì *Dagger* sử dụng thông tin có được từ 2 annotation này là khác nhau để khởi tạo dependency. Để hiểu kỹ hơn, các bạn có thể tham khảo [ở đây](https://dagger.dev/dev-guide/faq.html#why-is-binds-different-from-provides).

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
fun provideOkHttp(): OkHttpClient {
    return OkHttpClient.Builder()
        .authenticator(TokenAuthenticator())
        .build()
}
```

Cùng với đó, ở tất cả những chỗ sử dụng `Retrofit` cũng cần sử dụng `@Named` để chỉ rõ dependency cần dùng là loại nào
```
@Provides
fun provideUserServices(@Named("No-Authentication") retrofit: Retrofit): UserServices {
    return retrofit.create(UserServices::class.java)
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

Cách sử dụng thì sẽ tương tự như `@Named`
```
@Provides
@AuthenticationRetrofit
fun provideRetrofitNoAuthentication(baseUrl: String, gson: Gson): Retrofit { ... }

@Provides
@NoAuthenticationRetrofit
fun provideRetrofitAuthentication(baseUrl: String, okHttpClient: OkHttpClient, gson: Gson): Retrofit { ... }

@Provides
fun provideUserServices(@NoAuthenticationRetrofit retrofit: Retrofit): UserServices { ... }
```

Cuối cùng, sau khi đã hoàn thành việc khai báo các dependency không mấy khó khăn, phần việc nhàm chán còn lại là khởi tạo và quản lý các dependency, *Dagger* sẽ lo hết cho chúng ta. Diagram dưới đây sẽ thể hiển mối quan hệ giữa các thành phần trong *Dagger*:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/4w1wm4uy48_Dagger2_component.jpg">
</p>

Gòi xong, hy vọng với những kiến thức nhiêu đây, bạn đã có thể bắt đầu và không thấy nản với *Dagger 2* nữa. Tuy chương trình trên đây chỉ là một chương trình nhỏ, chúng ta có thể sẽ chưa thấy hết được sức mạnh của *Dagger 2* khi các dependency là chưa nhiều. Tuy nhiên, với một ứng dụng phức tạp hơn với nhiều màn hình hơn, mỗi màn hình sẽ sử dụng một loạt các dependency kèm theo thì việc khởi tạo và quản lý sẽ rất mất thời gian khi phải viết rất nhiều những đoạn code lặp và còn dễ gây ra lỗi nữa. *Dagger 2* chính là "the right tool" giúp chúng ta loại bỏ mốí quan tâm đấy và tập trung vào các phần quan trọng hơn của chương trình.
