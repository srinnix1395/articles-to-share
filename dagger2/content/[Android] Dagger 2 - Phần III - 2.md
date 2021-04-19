[Android] Dagger 2 - Phần III - 2: The time of our dependencies

Bài viết là phần thứ III của series bài học vỡ lòng về *Dagger 2*. Nếu bạn chưa đọc các phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Basic principles](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)
2. [[Android] Dagger 2 - Phần II: Into the Dagger 2](https://kipalog.com/posts/Android--Dagger-2---Phan-II--Into-the-Dagger-2)
3. [[Android] Dagger 2 - Phần III - 1: The time of our dependencies](https://kipalog.com/posts/Android--Dagger-2---Phan-III---1--The-time-of-our-dependencies)
4. [Android] Dagger 2 - Phần III - 2: The time of our dependencies
5. [[Android] Dagger 2 - Phần IV: A new horizon]()

# Trong bài học trước...

Chúng ta đã tìm hiểu cách xây dựng thêm các component, từ đó chia *dependency graph* của chương trình thành những phần nhỏ hơn để dễ bề cai trị. Cùng với đó, chúng ta đã biết tại sao phải sử dụng các *scope annotation* và cách sử dụng chúng như thế nào.

# Đi vào bài học hôm nay...

Tuy những *dependency graph* nhỏ hơn đã được định nghĩa, những phần này vẫn chưa được sắp xếp sao cho khớp với nhau bởi chúng ta chưa chỉ cho *Dagger* biết mối liên hệ giữa các miếng ghép này là gì. Bài viết này sẽ giải quyết nốt vấn đề còn dơ dảng ấy.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/lb2kry1ck4_john-towner-3Kv48NS4WUU-unsplash.jpg">
  Photo by <a href="https://unsplash.com/@heytowner?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">JOHN TOWNER</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</p>

Trước khi đi vào phần cầm tay chỉ việc, chúng ta sẽ nhìn lại từng component xem các dependency mà component đang provide là gì

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/lvpprzypmn_Separated-component.jpg">
</p>

Như hình trên, chúng ta thấy có 3 component tương ứng với 3 scope khác nhau. Mỗi scope thể hiện một vòng đời của dependency mà component đó provide. Chính vì sự khác nhau về vòng đời này mà không phải class nào cũng chỉ sử dụng các dependency được provide từ 1 component duy nhất, VD: `MainRepository` được provide bởi `ActivityComponent` nhưng lại cần dependency là `ApiHelper` được provide bởi `ApplicationComponent` và `LoggedUserInfo` được provide bởi `UserComponent`.

Để giải quyết vấn đề về sự phụ thuộc lẫn nhau giữa các component như trên, chúng ta cần link các component lại với nhau. *Dagger* cung cấp cho chúng ta 2 lựa chọn để làm việc đó:
- Component dependency
- Subcomponent

### Component dependency

Lựa chọn này được *Google* mang từ *Dagger 1* lên nhưng khá lạ là official document của *Dagger 2* hiện tại gần như không đề cập đến cách implement này. Việc này theo cá nhân mình là bởi cách implement này phức tạp và khó nắm bắt hơn so với *Subcomponent*. Tuy nhiên, với các ứng dụng multi-module mà mỗi  module lại có các component thì đây là cách bắt buộc chúng ta phải sử dụng. Xem thêm [Issue Subcomponent vs Component dependency](https://github.com/google/dagger/issues/1364)

Những đặc điểm của kiểu implement này là:
- Mối quan hệ giữa 2 component là mối quan hệ phụ thuộc (hay có thể coi là *has-a*) nên 2 component thực chất là độc lập với nhau.
- Component phụ thuộc (component có scope hẹp hơn) và component cần phụ thuộc (component có scope rộng hơn) không được dùng chung một *scope annotation*. Xem thêm [ở đây](https://github.com/google/dagger/issues/107#issuecomment-71073298)
- Component cần phụ thuộc bắt buộc phải khai báo các dependency muốn chia sẻ cho component phụ thuộc
- Một component có thể phụ thuộc vào nhiều component

Với chương trình đang xét, chúng ta có một số phân tích như sau:
- Chúng ta thấy `UserComponent` sẽ provide `MainRepository` và `MainRepository` lại cần `ApiHelper` được provide bởi `ApplicationComponent`. Bởi vậy, `UserComponent` cần phụ thuộc vào `ApplicationComponent`
- Tương tự với phân tích ở trên, `ActivityComponent` cũng cần phụ thuộc vào `ApplicationComponent`.

Sau khi đã phân tích, chúng ta sẽ bắt tay vào implement *component dependency*. Các bước để implement là:
1. Chúng ta cần khai báo ở các dependency phụ thuộc rằng dependency cần phụ thuộc là gì bằng cách khai báo ở phía sau annotation `@Component`
```
@Component(dependencies = [ApplicationComponent::class], modules = [UserModule::class, PresenterUserModule::class, RepositoryUserModule::class])
@LoggedUserScope
interface UserComponent { ... }
```
```
@Component(dependencies = [ApplicationComponent::class],modules = [PresenterModule::class, RepositoryModule::class])
@ActivityScope
interface ActivityComponent { ... }
```

2. Khai báo những dependency cần được lấy ra từ component phụ thuộc. Chúng ta làm điều này bằng cách khai báo những function không có tham số và có kiểu trả về là kiểu của dependency cần lấy ra.
```
@Singleton
@Component(modules = [UtilsModule::class,  ApiModule::class])
interface ApplicationComponent {

    fun userManager(): UserManager

    fun apiHelper(): ApiHelper

    fun dbHelper(): DbHelper

    fun preferenceHelper(): PreferenceHelper
}
```

3. Khởi tạo component: Vì `UserComponent` phụ thuộc vào `ApplicationComponent` nên khi khởi tạo `UserComponent`, chúng ta cần truyền 1 instance của `ApplicationComponent` từ bên ngoài vào (có thể nói đây là *manual depedency injection* khi chúng ta phải tự truyền vào). Mặc định, *Dagger* sẽ gen cho chúng ta một class `Builder` của component và bên trong `Builder` đó 1 function để truyền component phụ thuộc vào:
```
val userComponent = DaggerUserComponent.builder()
            .applicationComponent((context as MyApplication).applicationComponent)
            .build()
```
```
val activityComponent = DaggerActivityComponent.builder()
            .applicationComponent((application as MyApplication).applicationComponent)
            .build()
```

### Subcomponent

Lựa chọn này mới được *Google* giới thiệu từ *Dagger 2*. Một số đặc điểm của cách implement này là:
- Mối quan hệ giữa 2 component là mối quan hệ cha con (hay có thể coi là kế thừa - *is-a*) nên 2 component có phần gắn với nhau chặt hơn.
- Tương tự như *component dependency*, 2 component không được dùng chung một *scope annotation* bởi nó sẽ gây ra *a circular dependency* làm *Dagger* không biết khởi tạo thế nào.
- Vì là mối quan hệ cha con nên subcomponent sẽ kế thừa tất cả những dependency mà parent-component đã provide. Bởi vậy, chúng ta không cần khai báo cụ thể những dependency nào muốn chia sẻ như với *component dependency* nữa.
- Một subcomponent chỉ có thể có một parent-component duy nhất.

Tương tự như bước phân tích với *component dependency*, chúng ta sẽ đi đến kết luật:
- `UserComponent` sẽ là subcomponent của `ApplicationComponent`
- `ActivityComponent` sẽ là subcomponent của `ApplicationComponent`

Các bước để implement subcomponent là:
1. Sử dụng annotation `@Subcomponent` để khai báo các component con thay vì `@Component`. Ngoài ra, chúng ta cần khai báo thêm một interface Factory để khởi tạo subcomponent
```
@Subcomponent(modules = [UserModule::class, PresenterUserModule::class, RepositoryUserModule::class])
@LoggedUserScope
interface UserComponent {

      @Subcomponent.Factory
      interface Factory {

          fun create() : UserComponent
      }
}
```
```
@Subcomponent(modules = [PresenterModule::class, RepositoryModule::class])
@ActivityScope
interface ActivityComponent {

      @Subcomponent.Factory
      interface Factory {

          fun create() : ActivityComponent
      }
}
```

    **Note:** Subcomponent Factory phải có một function trả về component và chúng ta sẽ sử dụng function đó để khởi tạo component. Ngoài ra, để truyền một dependency mà không thể được khởi tạo bên trong *dependency graph*, chúng ta có thể sử dụng `@BindInstance`. Xem thêm ở [Dagger 2 - Phần IV]()

2. Khai báo thêm một module cho component cha và list ra tất cả những subcomponent mà nó có bên trong module vừa định nghĩa.
```
@Module(subcomponents = [UserComponent::class, ActivityComponent::class])
class ApplicationSubComponent
```
```
@Singleton
@Component(modules = [ApplicationSubComponent::class, ...])
interface ApplicationComponent { ... }
```

3. Khai báo một function không tham số ở component cha và trả về kiểu Factory của subcomponent. Chúng ta sẽ sử dụng function này để có thể truy cập và khởi tạo subcomponent từ bên ngoài.
```
@Singleton
@Component(modules = [ApplicationSubComponent::class, UtilsModule::class, ApiModule::class])
interface ApplicationComponent {

      fun userComponentFactory(): UserComponent.Factory

      fun activityComponentFactory(): ActivityComponent.Factory
}
```

4. Khởi tạo subcomponent
```
val userComponent = (context as MyApplication).applicationComponent
            .userComponentFactory()
            .create()
```
```
val activityComponent = (application as MyApplication).applicationComponent
            .activityComponentFactory()
            .create()
```

    **Note:** Chúng ta có thể nhận ra một sự khác nhau ở đây giữa *subcomponent* và *component dependency* khi *Dagger* không gen cho chúng ta các class implement component như *DaggerUserComponent* hay *DaggerActivityComponent* trong trường hợp sử dụng kiểu implement *subcomponent*. Đó là bởi vì thực chất các subcomponent vẫn là một phần của component cha thay vì tách ra hẳn thành một component riêng biệt như đối với *component dependency*
    
# Cùng nhìn lại

Vậy là với *scope*, *component dependency* và *subcomponent*, chúng ta đã có đủ các công cụ để fulfill được các yêu cầu về sự khác nhau trong vòng đời của các dependency. Từ đó, chương trình của chúng ta cũng sẽ được module hóa rõ ràng hơn và clean hơn. Và đến được đây, khi nhìn vào những đoạn code được *Dagger* gen, những sự dễ dàng trong việc thay đổi dependency khi việc thay đổi ở phía trên là cực kỳ ít, chúng ta cũng đã có thể mường tược được về những gì *Dagger* đã làm và sẽ làm cho chúng ta khi những chương trình trong thực tế lớn hơn nhiều so với ví dụ đã xét.
