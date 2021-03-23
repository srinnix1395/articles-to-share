# Kế thừa đối với class

### Abstract class

*Abstract class* là các class mà không thể khởi tạo và thường được sử dụng để định nghĩa các class tổng quát với những property và method chung nhưng không được implement và để các class con định nghĩa tùy theo đặc điểm của class con. Để định nghĩa một *abstract class*, ta sử dụng từ khóa `abstract`
```
// This class is declared abstract and thus
// can't be instantiated.
abstract class AbstractContainer {
  // Define constructors, fields, methods...

  void updateChildren(); // Abstract method.
}
```
