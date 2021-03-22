# Class

*Dart* là một ngôn ngữ hướng đối tượng và hỗ trợ kế thừa thông qua cơ chế mixin. Mọi object đều là một instance của một class và tất cả class (ngoại trừ `Null`) đều là lớp con của `Object`. *Dart* chỉ hỗ trợ đơn kế thừa, tức là mọi class (trừ `Object?`) đều chỉ một lớp cha duy nhát. Tuy nhiên, cơ chế mixin cho phép một class có thể tái sử dụng todo của nhiều class khác. Ngoài ra, *extension method* cũng là một cách để thêm các chức năng vào một class mà không cần sửa đổi class hoặc tạo các lớp con.

### Truy cập class members

Các member của một object bao gồm các *instance variable* và các *method*. Để truy cập đến các member này, ta sử dụng `.`
```
var p = Point(2, 2);

// Get the value of y.
assert(p.y == 2);

// Invoke distanceTo() on p.
double distance = p.distanceTo(Point(4, 4));
```

Và sử dụng `?.` đối với trường hợp đối tượng truy cập có kiểu dữ liệu nullable.
```
// If p is non-null, set a variable equal to its y value.
var a = p?.y;
```

### Sử dụng constructor

Để khởi tạo các object, ta sử dụng *constructor*. Tên của *constructor* trong *Dart* có thể là *ClassName* hoặc *ClassName.identifier*.
```
var p1 = new Point(2, 2);
var p2 = new Point.fromJson({'x': 1, 'y': 2});
```

Trong ví dụ trên, `Point.fromJson()` cũng là một *constructor* của class `Point`. Ngoài ra, từ khóa `new` khi khởi tạo object cũng có thể được lược bỏ
```
var p1 = Point(2, 2);
var p2 = Point.fromJson({'x': 1, 'y': 2});
```

Một số class còn có các *constant constructor*. Để khởi tạo một *compile-time constant* thông qua *constant constructor*, ta thêm `const` vào trước lời gọi
```
var p = const ImmutablePoint(2, 2);
```

Hai *compile-time constant* có cùng giá trị và được khởi tạo thông qua *constant constructor* thực chất là một object
```
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);

assert(identical(a, b)); // They are the same instance!
```

Trong trường hợp khởi sử dụng *constant constructor* trong một *constant context*, chúng ta có thể lược bỏ các từ khóa `const` không cần thiết để code dễ đọc hơn. VD: trước khi lược bỏ
```
// Lots of const keywords here.
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```

...và sau khi lược bỏ
```
// Only one const, which establishes the constant context.
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

### Gettting an object's type

Để get kiểu của object lúc run-time, chúng ta có thể sử dụng property `runtimeType` có sẵn của `Object` và trả về object kiểu `Type`.
```
print('The type of a is ${a.runtimeType}');
```

Vậy là với 3 phần đầu tiên của bài này, chúng ta đã biết cách sừ dụng class: cách truy cập đến các member, cách khởi tạo và cách lấy kiểu của một object. Các phần tiếp theo sẽ nói về cách implement một class.

### Instance variable

Ta có thể khai báo các *instance variable* theo cú pháp sau:
```
class Point {
  double? x; // Declare instance variable x, initially null.
  double? y; // Declare y, initially null.
  double z = 0; // Declare z, initially 0.
}
```

Giá trị mặc định của các biến nullable chưa được khởi tạo là `null`.

Tất cả các *instace-variable* đều có một *getter* method được tự động gen. Các *non-final instance variable* và `late final` *instance variable* (mà không có function khởi tạo `initializer()`) thì được gen tự động các *setter* method (Chi tiết ta sẽ nói đến ở phần sau)

Đối với một `final` instance variable, chúng ta có thể khởi tạo trực tiếp khi khai báo variable đó hoặc khởi tạo ở constructor. Nếu bạn không muốn khởi tạo tại một trong 2 vị trí đó, hãy sử dụng `late` để khởi tạo sau. Tuy nhiên, [hãy cẩn thân với điều này](https://dart.dev/guides/language/effective-dart/design#avoid-public-late-final-fields-without-initializers)

### Constructor

*Constructor* của một class là một function có tên giống như class đó và không có kiểu dữ liệu trả về
```
class Point {
  double x = 0;
  double y = 0;

  Point(double x, double y) {
    // There's a better way to do this, stay tuned.
    this.x = x;
    this.y = y;
  }
}
```

*Dart* hỗ trợ một syntactic sugar cho đoạn code trên
```
class Point {
  double x = 0;
  double y = 0;

  Point(this.x, this.y) {
    print("Initialized a Point($x,$y)!");
  }
}
```
Đối với constructor trên, đoạn code gán giá trị cho *instance variable* sẽ được chạy trước body của constructor.


##### Default constructor

Nếu class không khai báo một constructor nào, một constructor mặc định sẽ được tự động gen. Constructor mặc định này không có tham số nào và sẽ gọi đến constructor không tham số của lớp cha nếu có

##### Kế thừa constructor

Các lớp con sẽ không kế thừa các constructor từ lớp cha. Nếu một lớp con không khai báo một constructor nào, nó sẽ chỉ có một constructor mặc định không tham số  được tự động gen duy nhất.

##### Named constructor

*Dart* cũng hỗ trợ việc đặt tên cho constructor để việc khởi tạo có ý nghĩa dễ nhận biết hơn:
```
class Point {
  double x = 0;
  double y = 0;

  Point(this.x, this.y);

  // Named constructor
  Point.origin(double xOrigin, double yOrigin) {
    x = xOrigin;
    y = yOrigin;
  }
}
```

##### Gọi non-default superclass constructor

Mặc định, constructor của lớp con sẽ gọi constructor không tên, không tham số của lớp cha trước khi chạy đoạn code trong constructor's body của nó. Nếu có *initializer list* (chúng ta sẽ nói tới ở phía sau), nó sẽ được chạy trước constructor của lớp cha. Thứ tự chạy của từng đoạn code như sau:
1. initializer list
2. constructor không tham số của lớp cha
3. constructor không tham số của lớp chính

Trong trường hợp lớp cha không có constructor không tên, không tham số, chúng ta phải gọi thay thế một constructor khác của lớp cha bằng cách sử dụng `:` phía sau của constructor và trước thân của constructor
```
class Person {
  String? firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person does not have a default constructor;
  // you must call super.fromJson(data).
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}
```

Bởi vì tham số của constructor lớp cha sẽ được tính toán trước khi constructor được gọi, tham số được truyền constructor của lớp cha có thể là một expression như là một lời gọi function
```
class Employee extends Person {
  Employee() : super.fromJson(fetchDefaultData());
  // ···
}
```

##### Initializer list

Initializer list là một cú pháp cho phép việc khởi tạo instance variable trước khi constructor's body được chạy
```
// Initializer list sets instance variables before
// the constructor body runs.
Point.fromJson(Map<String, double> json)
    : x = json['x']!,
      y = json['y']! {
  print('In Point.fromJson(): ($x, $y)');
}
```

Trong quá trình phát triển
