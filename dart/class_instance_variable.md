# Class's instance variables

Đối với một class trong *Dart*, một *instance variable* là một thuộc tính chứa thông tin về đặc điểm của class đó. Ta có thể khai báo các *instance variable* theo cú pháp sau:
```
class Point {
  double? x; // Declare instance variable x, initially null.
  double? y; // Declare y, initially null.
  double z = 0; // Declare z, initially 0.
}
```

Giá trị mặc định của các biến nullable chưa được khởi tạo là `null`.

Tất cả các *instance-variable* đều có một *getter* method được tự động gen. Các *non-final instance variable* và `late final` *instance variable* (mà không có function khởi tạo `initializer()`) thì có các *setter* method được tự động gen. Chi tiết ta sẽ nói đến ở phần [Getter & setter]()

Nếu bạn khai báo một non-`late` instance variable và gán giá trị ngay cho nó, đoạn code gán gía trị này sẽ được thực hiện trước constructor và [initializer list]() của class đó
```
class Point {
  double? x; // Declare instance variable x, initially null.
  double? y; // Declare y, initially null.
}

void main() {
  var point = Point();
  point.x = 4; // Use the setter method for x.
  assert(point.x == 4); // Use the getter method for x.
  assert(point.y == null); // Values default to null.
}
```

Để khai báo một *instance-variable* không thể thay đổi, ta dùng thêm `final` khi khai báo. Việc khởi tạo `final`, non-`late` *instance variable* phải được thực hiện khi khai báo hoặc ở constructor's body hay [initializer list]()
```
class ProfileMark {
  final String name;
  final DateTime start = DateTime.now();

  ProfileMark(this.name);
  ProfileMark.unnamed() : name = '';
}
```

Nếu bạn muốn gán giá trị cho `final` *instance variable* sau cả khi constructor's body được thực hiện, hãy sử dụng `late final`. Tuy nhiên, [hãy cẩn thân với điều này](https://dart.dev/guides/language/effective-dart/design#avoid-public-late-final-fields-without-initializers)
