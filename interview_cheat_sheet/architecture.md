1. MVC - Model View Controller
  * Mô hình chia thành 3 lớp nhưng thực chất logic được xử lý chỉ trong 1 class: Activity/Fragment
  * Cons
    - Không thể viết unit test vì có chứa Android API
    - Controller có khả năng trở thành God class vì tất cả chỉ được xử lý tại 1 class
2. MVP - Model View Presenter
  * Mô hình 3 lớp
    - Model:
    - View: "dumb" class, nhận action từ người dùng, truyền cho presenter và thay đổi UI theo ý của presenter
    - Presenter: Chịu trách nhiệm xử lý logic, không chứa Android API để có thể viết unit test
  * Quan hệ 1 - 1 giữa View và Presenter
  * Presenter biết mình đang giao tiếp với View nào
  * Pros
    - Các lớp được phân chia nhiệm vụ rõ ràng hơn,
    - Có thể viết unit test
    - Dễ mở rộng hơn
  * Cons
    - Phải viết nhiều code thừa
3. MVVM - Model View ViewModel  
  * Mô hình 3 lớp
    - Model:
    - View: Nhận action từ người dùng, truyền action cho ViewModel để xử lý, lắng nghe ViewModel và phản ứng với những sự thay đổi data
    - ViewModel: xử lý logic ở đây
  * Quan hệ 1 - n giữa View và ViewModel
  * ViewModel không biết mình giao tiếp với View nào, chỉ chìa ra những observable để view có thể lắng nghe và phản ứng lại.
