[Android] Dagger 2 - Phần IV: A new horizon

Bài viết là phần cuối của series bài học vỡ lòng về *Dagger 2*. Nếu bạn chưa đọc phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Basic principles](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)
2. [[Android] Dagger 2 - Phần II: Into the Dagger 2](https://kipalog.com/posts/Android--Dagger-2---Phan-II--Into-the-Dagger-2)
3. [[Android] Dagger 2 - Phần III - 1: The time of our dependencies](https://kipalog.com/posts/Android--Dagger-2---Phan-III---1--The-time-of-our-dependencies)
4. [[Android] Dagger 2 - Phần III - 2: The time of our dependencies](https://kipalog.com/posts/Android--Dagger-2---Phan-III---2--The-time-of-our-dependencies)
5. [Android] Dagger 2 - Phần IV: A new horizon

# Trong bài học trước...

Chúng ta đã thành công trong việc chia tách *dependency graph* ban đầu thành các *dependency graph* nhỏ hơn nhưng vẫn không làm mất đi tính kết nối giữa chúng. Ngoài ra, lời giải của bài toán chia để trị cũng giúp cho việc quản lý vòng đời của các dependency trở nên đơn giản hơn vài phần.

# Đi vào bài học hôm nay...

Chúng ta sẽ điểm qua nốt những phần nhỏ nhỏ còn lại mà sẽ giúp cho việc implement *Dagger* vốn đã bớt phức tạp sau khi bạn hiểu cách *Dagger* hoạt động, nay còn đơn giản hơn. Ngoài ra, một số phần mình thấy là sẽ hữu dụng sẽ được trình bày ở bài viết này, bạn có thể lướt qua và tự nhủ rằng: "Wow, hóa ra *Dagger* có cả chức năng này ah!" rồi một lúc nào đó, bạn gặp một usecase tương ứng thì bạn vẫn biết là *Dagger* vẫn có thể giải quyết nó dễ dàng.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/6mmrbymcd1_jonatan-pie-FOcMXBbe5rU-unsplash.jpg">
  Photo by <a href="https://unsplash.com/@r3dmax?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Jonatan Pie</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</p>

### Khởi tạo component

Nếu như đã lướt qua cả 4 phần trước, hẳn sẽ có lúc bạn thấy hơi rối một chút về việc khởi tạo component khi lúc thì chúng ta sử dụng `Builder`, lúc lại "bắt buộc" phải dùng `Factory`. Vậy thì rốt cuộc, sự khác nhau giữa hai cách khởi tạo này là gì? Phần này của bài viết sẽ giải đáp nỗi niềm đắn đo ấy.

Câu trả lời cho câu hỏi trên là chúng ta sẽ có 2 style để khởi tạo một component. Đó là sử dụng *Factory*  hoặc *Builder*. Và mình gọi là style bởi vì mình nghĩ việc chọn cái nào đơn giản là thói quen hoặc sở thích của mỗi người và vì thế nên không có cách nào là bắt buộc cả ;)

##### Factory

Với *Factory*, chúng ta cần tạo một interface (thường chúng ta sẽ đặt luôn tên là `Factory`) và annotate nó với annotation `@Component.Factory` (hoặc `@Subcomponent.Factory` nếu component đó là subcomponent). Bên trong interface đó, chúng ta cần khai báo một method có kiểu trả về là chính component đó. Việc khởi tạo component sau này sẽ thông qua interface và method đó.
```
@Singleton
@Component(modules = [ApplicationSubComponent::class, UtilsModule::class, ApiModule::class])
interface ApplicationComponent {

    @Component.Factory
    interface Factory {
        fun create() : ApplicationComponent
    }
}
```

Và đoạn code khởi tạo component sẽ như thế này:
```
val applicationComponent = DaggerApplicationComponent
            .factory()
            .create()
```

Tuy nhiên, khi chạy chương trình, chúng ta thấy *Dagger* báo lỗi sau:
```
ApplicationComponent.java:21: error: @Component.Factory method is missing parameters for required modules or components: [io.srinnix.playground.dagger2.automatic.module.UtilsModule]
```

Issue này xảy ra vì `UtilsModule` cần một dependency mà chỉ có thể được khởi tạo bên ngoài component, đó là `Context`. Bởi vậy, `UtilsModule` cũng cần được truyền vào từ bên ngoài. Tức là chúng ta cần thêm tham số cho method `create()` để truyền `UtilsModule` vào cho component.
```
@Component.Factory
interface Factory {
    fun create(utilsModule: UtilsModule) : ApplicationComponent
}
```

Việc khởi tạo component sẽ thành như thế này:
```
val applicationComponent = DaggerApplicationComponent
            .factory(UtilsModule(applicationContext)
            .create()
```

###### BindInstance

Có một cách khác để truyền các dependency được khởi tạo bên ngoài component thay vì truyền cả module vào. Đó là sử dụng annotation `@BindInstance`. Thay vì truyền cả `UtilsModule` vào, chúng ta chỉ cần truyền chính dependency được khởi tạo bên ngoài vào và thêm `@BindInstance` vào trước tham số của method `create()`:
```
@Component.Factory
interface Factory {
    fun create(@BindsInstance @ApplicationContext context: Context): ApplicationComponent
}
```

Khi sử dụng `@BindInstance`, *Dagger* sẽ biết cần thêm instance mà bạn truyền vào vào *dependency graph* để khi có chỗ nào request, instance này sẽ được provide. Với `UtilsModule`, chúng ta không cần truyền `Context` thông qua constructor nữa:
```
@Module
object UtilsModule { ... }
```

Và khi khởi tạo, chúng ta sẽ truyền `Context` vào là xong:
```
val applicationComponent = DaggerApplicationComponent
            .factory()
            .create(applicationContext)
```

##### Builder

Với *Builder*, *Dagger* đã mặc định gen cho các component một class `Builder` để khởi tạo component (nếu chúng ta không khai báo interface `@Component.Factory`). Mình đã sửa lại `UserComponent` thành phụ thuộc vào `ApplicationComponent` và đây là class `Builder` của `DaggerUserComponent` được gen ra:
```
public static final class Builder {
    private RepositoryUserModule repositoryUserModule;

    private ApplicationComponent applicationComponent;

    private Builder() {
    }

    @Deprecated
    public Builder userModule(UserModule userModule) {
      Preconditions.checkNotNull(userModule);
      return this;
    }

    public Builder repositoryUserModule(RepositoryUserModule repositoryUserModule) {
      this.repositoryUserModule = Preconditions.checkNotNull(repositoryUserModule);
      return this;
    }

    public Builder applicationComponent(ApplicationComponent applicationComponent) {
      this.applicationComponent = Preconditions.checkNotNull(applicationComponent);
      return this;
    }

    public UserComponent build() {
      if (repositoryUserModule == null) {
        this.repositoryUserModule = new RepositoryUserModule();
      }
      Preconditions.checkBuilderRequirement(applicationComponent, ApplicationComponent.class);
      return new DaggerUserComponent(repositoryUserModule, applicationComponent);
    }
  }
```

Chúng ta thấy rằng: với mỗi module mà component khai báo, *Dagger* cũng sẽ gen tương ứng một method để truyền module đó từ bên ngoài vào. Tuy nhiên, thực chất chúng ta chỉ phải phải truyền module mà chúng ta thực sự sử dụng hoặc module có chứa dependency được truyền vào từ bên ngoài (`UtilsModule` trong trường hợp `ApplicationComponent`). Với các module còn lại, một là với những module có sử dụng, *Dagger* sẽ tự khởi tạo cho chúng ta (`RepositoryUserModule`); hai là với những [unused module](https://dagger.dev/unused-modules), *Dagger* sẽ tự động bỏ qua.

Việc khởi tạo component sẽ như sau:
```
val userComponent = DaggerUserComponent.builder()
            .applicationComponent((context as? MyApplication)?.applicationComponent)
            .build()
```


### Dagger annotation cheat sheet

Tuy là chưa đề cập đến [Hilt](https://developer.android.com/training/dependency-injection/hilt-android)- một phiên bản rút gọn của *Dagger*. Mình xin phép re-up một cái ảnh liệt kê những mô tả lướt qua về các annotation trong *Dagger*.

<p align="center">
  <img src="https://developer.android.com/images/training/dependency-injection/hilt-cheatsheet.png">Nguồn:</a> <a href="https://developer.android.com/training/dependency-injection/hilt-cheatsheet">Google</a>
</p>

# Cùng nhìn lại

Ơn giàng, ơn Đảng, cuối cùng cũng đã đến phần này rồi ^^ Mình nghĩ chúng ta có thể tạm đặt một dấu chấm cho series này ở đây được rồi. Tuy vẫn còn một số phần mình chưa đề cập đến trong series này như [Lazy initialization](https://proandroiddev.com/deep-dive-into-dagger-lazy-7a5860cca7cc), [Multi bindings](https://dagger.dev/dev-guide/multibindings), [Testing](https://dagger.dev/dev-guide/testing) hay [Sử dụng Dagger với multi-module app](https://developer.android.com/training/dependency-injection/dagger-multi-module)... nhưng mình nghĩ: sẽ cần thời gian để bạn tiêu hóa được những phần cơ bản nhất đã, một khi bạn đã hiểu *Dagger* sẽ làm gì và làm như thế nào, những phần còn lại sẽ không còn quá khó khăn để nắm bắt nữa. Và mình cũng muốn dành đất cho sự "đói khát kiến thức" của bạn nữa ;)

Good luck and good night.
