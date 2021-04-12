[Android] Dagger 2 - Phần III: The time of our dependencies

Bài viết là phần thứ III của series bài học vỡ lòng về *Dagger 2*. Nếu bạn chưa đọc các phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Basic principles](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)
2. [[Android] Dagger 2 - Phần II: Into the Dagger 2](https://kipalog.com/posts/Android--Dagger-2---Phan-II--Into-the-Dagger-2)
3. [Android] Dagger 2 - Phần III: The time of our dependencies

# Trong bài học trước...

Chúng ta đã tìm hiểu cách "mô hình hóa" các mối quan hệ giữa class và các dependency thông qua *dependency graph*. Tiếp đó, chúng ta xây dựng một ứng dụng đơn giản và từng bước áp dụng những annotation cơ bản trong *Dagger 2* để khởi tạo các dependency.

# Đi vào bài học hôm nay...
Chương trình ở phần II sẽ tiếp tục scale up lên với thêm nhiều màn hình và chức năng. Khi đó, chúng ta cũng sẽ đối mặt với vấn đề quản lý vòng đời của các dependency.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/lb2kry1ck4_john-towner-3Kv48NS4WUU-unsplash.jpg">
  Photo by <a href="https://unsplash.com/@heytowner?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">JOHN TOWNER</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</p>

Chúng ta sẽ xây dựng thêm một màn hình để xác thực thông tin user (`LoginActivity`) trước khi đến với màn hình chính bên trong `MainActivity`. Tương tự như `MainActivity`, chúng ta cũng sẽ có các class ăn theo: `LoginPresenter`, `LoginRepository` và cả các implementation class của chúng.
```
class LoginActivity : FragmentActivity() {

    @Inject
    lateinit var mLoginPresenter: LoginPresenter

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        val userComponent = DaggerUserComponent.builder()
            .utilsModule(UtilsModule(this))
            .build()
        userComponent.inject(this)
    }
}

class LoginPresenterImpl @Inject constructor(var repository: LoginRepository) : LoginPresenter { ... }
class LoginRepositoryImpl @Inject constructor(var apiHelper: ApiHelper,
                                              var preferenceHelper: PreferenceHelper,
                                              var dbHelper: DbHelper) : LoginRepository { ... }
```

Các class này cũng sẽ được inject thông qua *Dagger* nên ta cũng phải thêm inject function vào component, thêm provide function vào `PresenterModule` và `RepositoryModule`
```
@Component(modules = [UtilsModule::class, PresenterModule::class, RepositoryModule::class,  ApiModule::class])
interface UserComponent {
    ...

    fun inject(loginActivity: LoginActivity)
}

@Module
abstract class PresenterModule {
    ...

    @Binds
    abstract fun provideLoginPresenter(loginPresenterImpl: LoginPresenterImpl): LoginPresenter
}

@Module
abstract class RepositoryModule {
    ...

    @Binds
    abstract fun provideLoginRepository(loginRepositoryImpl: LoginRepositoryImpl): LoginRepository
}
```

Chúng ta thấy rằng `ApiHelper`, `PreferenceHelper` và `DbHelper` đang được sử dụng ở `MainRepositoryImpl`, `LoginRepositoryImpl` và có thể là ở những repository khác trong tương lai nữa. Bởi vậy, chúng ta muốn chỉ có một instance duy nhất của các class đó được tạo ra thay vì thêm repository thì thêm 3 instance được khởi tạo. Mở rộng ra trong thực tế, sẽ có những trường hợp chúng ta muốn chỉ một và duy nhất một instance được tạo ra vì:
1. Chi phí khởi tạo class có thể là không nhỏ và còn tốn nhiều thời gian nữa. VD: Memory cache
2. Một class là dependency của nhiều class khác nhưng cần là duy nhất để đảm bảo tính nhất quán dữ liệu. VD: `UserData` sau khi login có thể được sử dụng bởi nhiều `Presenter`

Trong trường hợp implement *DI* thủ công, chúng ta có thể thỏa mãn yêu cầu trên bằng cách truyền cùng một instance cho nhiều class khác nhau. Nhưng với *Dagger*, vì không thể can thiệp vào quá trình khởi tạo này nên chúng ta cần báo cho *Dagger* biết bằng cách sử dụng các *scope annotation*.

### Scope

Giải thích một cách đơn giản, scope trong *Dagger* cho phép chúng ta quản lý vòng đời của dependency bằng cách gắn vòng đời của dependency vào vòng đời của component, tức là khi component "sống" thì dependency "sống" còn khi component "chết" thì dependency cũng được tiễn về miền cực lạc theo. Cơ chế này cũng giúp ta giải quyết vấn đề ở trên khi chỉ một instance duy nhất được provide dù nó được request nhiều lần tại nhiều class khác nhau. Tóm gọn lại là một component thì sẽ chỉ một instance được provide - *singleton* đối với component.

Trước khi sử dụng *scope annotation*, chúng ta sẽ thử request 2 dependency xem có bao nhiêu instance được tạo ra:
```
class MainRepositoryImpl @Inject constructor(var apiHelper: ApiHelper,
                                             var preferenceHelper: PreferenceHelper,
                                             var dbHelper: DbHelper,
                                             var dbHelper1: DbHelper
) : MainRepository{
    init {
        println(dbHelper.toString())
        println(dbHelper1.toString())
    }
}
```

Khi run project, chúng ta nhận được kết quả như sau:
```
2021-04-09 00:01:26.738 2478-2478/io.srinnix.playground I/System.out: io.srinnix.playground.dagger2.automatic.data.DbHelper@4e7f50e
2021-04-09 00:01:26.739 2478-2478/io.srinnix.playground I/System.out: io.srinnix.playground.dagger2.automatic.data.DbHelper@a41cb2f
```

Ngoài ra, code mà *Dagger* gen ra cũng cho ta câu trả lời vì sao có 2 instance được tạo ra:
```
public final class DaggerMainComponent implements MainComponent {
    private DaggerMainComponent() { }

    private DbHelper dbHelper() {
        return new DbHelper();
    }
}
```

Điều đó có nghĩa là, với mỗi lần request dependency, một instance mới tương ứng sẽ được tạo ra. Để giải quyết vấn đề này, chúng ta sẽ sử dụng *scope annotation* để "gắn" vòng đời của `DbHelper` vào `MainComponent`. Trong *Dagger 2*, *scope annotation* duy nhất được định nghĩa trước là `@Singleton`. Chúng ta cần thêm `@Singleton` vào các vị trí sau:
1. Component mà chúng ta muốn sử dụng scope:
```
@Singleton
@Component(modules = [UtilsModule::class, PresenterModule::class, RepositoryModule::class,  ApiModule::class])
interface MainComponent { ... }
```

2. Những class mà được thêm vào *dependency graph* bằng `@Inject`:
```
@Singleton
class ApiHelper @Inject constructor(var userService: MainService) { ... }
```
```
@Singleton
class PreferenceHelper @Inject constructor() { ... }
```
```
@Singleton
class DbHelper @Inject constructor() { ... }
```

3. Các provide function và bind function của các module gắn với component:
```
@Module
object RepositoryModule {

    @Provides
    @Singleton
    @JvmStatic
    fun provideMainRepository(mainRepositoryImpl: MainRepositoryImpl): MainRepository {
        return mainRepositoryImpl
    }

    @Provides
    @Singleton
    @JvmStatic
    fun provideLoginRepository(loginRepositoryImpl: LoginRepositoryImpl): LoginRepository {
        return loginRepositoryImpl
    }
}
```
```
@Module
abstract class PresenterModule {

    @Binds
    @Singleton
    abstract fun provideMainPresenter(mainPresenterImpl: MainPresenterImpl): MainPresenter

    @Binds
    @Singleton
    abstract fun provideLoginPresenter(loginPresenterImpl: LoginPresenterImpl): LoginPresenter
}
```

Run project và kiểm tra lại kết quả: Voila!, chỉ có một instance duy nhất được tạo ra.
```
2021-03-22 10:48:40.391 11937-11937/? I/System.out: io.srinnix.playground.dagger2.automatic.data.DbHelper@86bb88e
2021-03-22 10:48:40.391 11937-11937/? I/System.out: io.srinnix.playground.dagger2.automatic.data.DbHelper@86bb88e
```

Tiếp tục kiểm tra code mà *Dagger* gen ra:
```
public final class DaggerMainComponent implements MainComponent {
    private Provider<DbHelper> dbHelperProvider;

    private DaggerMainComponent() {
      initialize();
    }

    private void initialize() {
        this.dbHelperProvider = DoubleCheck.provider(DbHelper_Factory.create());
    }

    private MainRepositoryImpl mainRepositoryImpl() {
        return new MainRepositoryImpl(apiHelper(), new PreferenceHelper(), dbHelperProvider.get());
    }

    private LoginRepositoryImpl loginRepositoryImpl() {
        return new LoginRepositoryImpl(apiHelper(), new PreferenceHelper(), dbHelperProvider.get());
    }
}
```

Chúng ta thấy rằng `dbHelperProvider` sẽ được khởi tạo một lần duy nhất khi *DaggerMainComponent* được khởi tạo. Và từ sau đó, mỗi khi cần `DbHelper`, `dbHelperProvider.get()` sẽ trả về một `DbHelper` duy nhất.

**Note:** Khi sử dụng `@Singleton`, cái tên này dễ làm chúng ta nhớ đến design pattern *Singleton*. Điều này có thể gây hiểu nhầm rằng cứ sử dụng annotation này thì các dependency sẽ "sống" cho đến mãn kiếp của chương trình. Tuy nhiên, cần phải nhấn mạnh rằng, đây chỉ là một cái tên, ý nghĩa của tên không giúp *Dagger* hiểu được vòng đời của dependency là gì. *Dagger* chỉ đơn giản dùng tên scope để quyết định xem: có nên provide một instance duy nhất (component có scope **trùng** với dependency) hay nên tạo ra thêm instance (component có scope **khác** với dependency).

Tuy nhiên, việc chỉ khởi tạo duy nhất 1 instance chỉ có tác dụng khi ta sử dụng chung component. Tức là nếu chúng ta khởi tạo 2 component ở 2 nơi (VD: `LoginActivity` và `MainActivity`), chúng ta sẽ có 2 bộ dependency khác nhau. Bởi vậy, chúng ta cần khởi tạo và keep component ở một chỗ khác, nơi có vòng đời bao trùm lên các Activity, thì khi dù cho các Activity được tạo ra hoặc chết đi, các dependency mong muốn vẫn sẽ tồn tại độc lập. Với *Android*, một chỗ phù hợp để keep các *global singleton dependency* kiểu này là `Application`. Cùng với việc chuyển đoạn code khai báo và khởi tạo component sang `Application`, chúng ta sẽ đổi tên component thành `ApplicationComponent` để thông tin về component được mô tả rõ hơn.
```
class MyApplication : Application() {

    val applicationComponent: ApplicationComponent by lazy {
        return@lazy DaggerApplicationComponent.builder()
            .utilsModule(UtilsModule(applicationContext))
            .build()
    }
}
```

Và ở các activity, chúng ta sẽ lấy `DaggerApplicationComponent` từ `MyApplication` ra để inject thay vì khởi tạo mới:
```
class LoginActivity : FragmentActivity() {
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        (application as? MyApplication)?.applicationComponent?.inject(this)
        ...
    }
}

class MainActivity : FragmentActivity() {
    ...

    override fun onCreate(savedInstanceState: Bundle?) {
        (application as? MyApplication)?.applicationComponent?.inject(this)
        ...
    }
}
```

Tuy nhiên, nếu gắn vòng đời `ApplicationComponent` vào `MyApplication`, những class như `MainPresenter`, `MainRepository`, `LoginPresenter`, `LoginRepository` sẽ tiếp tục tồn tại trong memory ngay cả sau khi các màn hình `MainActivity` và `LoginActivity` đóng. Thay vào đó, chúng ta muốn các instance sẽ được tạo mới mỗi khi một màn hình mới được mở lên. Để giải quyết vấn đề này, *Dagger* cho phép định nghĩa nhiều hơn một component (hay *dependency graph*) duy nhất. Chúng ta có thể chia `ApplicationComponent` thành các component nhỏ hơn để đóng gói các dependency có những điểm chung thành một component riêng. Các component nhỏ hơn này cũng sẽ có những scope riêng, đáp ứng đủ mọi loại yêu cầu về vòng đời của chúng ta.

Trước khi bắt tay vào implement scope, chúng ta sẽ tăng độ khó cho game để thấy rõ hơn sức mạnh của *Dagger*. Chương trình bây giờ sẽ gồm 3 màn hình như sau:

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/bew7c60zs3_Screenshot%20from%202021-04-12%2017-11-01.png">
</p>

Flow của app sẽ như sau:
- Ở màn hình `LoginActivity`, user sẽ nhập username và password để login
- Sau khi login thành công, màn `MainActivity` sẽ mở lên và hiển thị số thông báo chưa đọc
- Nếu user click vào button Settings, màn `SettingsActivity` sẽ mở lên. Ở đây, chúng ta có:
  + Button Refresh để mô tả việc refresh và lấy về số notification chưa đọc và hiển thị lên.
  + Button Logout để logout user và trở về màn `LoginActivity`
- Khi quay về màn `MainActivity`, số notification chưa đọc ở đây được update tương ứng với màn `SettingsActivity`.

Ngoài ra, phần data của app sẽ có thêm một sự thay đổi là chúng ta sẽ có thêm một class `UserManager` là một dạng memory cache, giữ thông tin của user sau khi user đăng nhập thành công. Và chúng ta sẽ lưu số notification chưa đọc tại đây.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/p6w8tr7sjn_DaggerScope.jpg">
</p>

Với requrirement như trên, chúng ta có thể chia các dependency vào 3 nhóm như sau với vòng đời tương ứng như sau:
1. Nhóm application: những class chúng ta chỉ muốn tạo một lần và sẽ được tái sử dụng trong suốt ứng dụng. Đó là `Context`(Application context), `ApiHelper`, `PreferenceHelper` và `DbHelper`
2. Nhóm user info: những class sẽ tồn tại khi user login thành công và chỉ bị hủy khi user logout.
2. Nhóm activity: những class sẽ được khởi tạo khi màn hình được mở lên và hủy khi màn hình đóng: `MainPresenter`, `MainRepository`, `LoginPresenter`, `LoginRepository`...

Vòng đời của hai nhóm này được mô hình hóa rõ hơn biểu đó dưới đây:
