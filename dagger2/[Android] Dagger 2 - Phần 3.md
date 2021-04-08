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

Chúng ta sẽ xây dựng thêm một màn hình để xác thực thông tin user (`LoginActivity`) trước khi đến với màn hình `UserActivity`. Tương tự như `UserActivity`, chúng ta cũng sẽ có các class ăn theo: `LoginPresenter`, `LoginRepository` và cả các implementation class của chúng.
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

Chúng ta thấy rằng `ApiHelper`, `PreferenceHelper` và `DbHelper` đang được sử dụng ở `LoginRepositoryImpl` và có thể là ở những repository khác trong tương lai nữa. Bởi vậy, chúng ta muốn chỉ có một instance duy nhất của các class đó được tạo ra thay vì thêm repository thì thêm instance được khởi tạo. Mở rộng ra trong thực tế, sẽ có những trường hợp chúng ta muốn chỉ một và duy nhất một instance được tạo ra vì:
1. Chi phí khởi tạo class có thể là không nhỏ và còn tốn nhiều thời gian nữa. VD: Memory cache
2. Một class là dependency của nhiều class khác nhưng cần là duy nhất để đảm bảo tính nhất quán dữ liệu. VD: `UserData` sau khi login có thể được sử dụng bởi nhiều `Presenter`

Trong trường hợp implement *DI* thủ công, chúng ta có thể đạt được yêu cầu trên bằng cách truyền cùng một instance cho nhiều class khác nhau. Nhưng với *Dagger*, vì không thể can thiệp vào quá trình khởi tạo này nên chúng ta cần báo cho *Dagger* biết bằng cách sử dụng các *scope annotation*.

### Scope

Giải thích một cách đơn giản, scope trong *Dagger* cho phép chúng ta quản lý vòng đời của dependency bằng cách gắn vòng đời của dependency vào vòng đời của component, tức là khi component "sống" thì dependency "sống" còn khi component "chết" thì dependency cũng được tiễn về miền cực lạc theo. Cơ chế này cũng giúp ta giải quyết vấn đề ở trên khi chỉ một instance duy nhất được provide dù nó được request nhiều lần tại nhiều class khác nhau - tóm gọn lại là một component, một instance được provide.

Trong  
