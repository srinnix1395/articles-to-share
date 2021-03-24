# Class's constructor

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

Khi sử dụng *constant constructor*, chúng ta vẫn phải thêm `const` trước constructor hoặc constructor đó phải ở trong một *constant context* thì object được tạo ra mới là một *compile-time constant*
```
var a = const ImmutablePoint(1, 1);        // compile-time constant
var b = ImmutablePoint(1, 1);              // not a compile-time constant
const c = ImmutablePoint(1, 1);            // compile-time constant
```

### Khai báo constructor

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

Trong *Dart*, mỗi class chỉ có thể có 1 default constructor duy nhất. Đó là constructor không có tên được khai báo đầu tiên trong class. Nếu bạn muốn có thêm constructor cho class đó, bạn bắt buộc phải đặt tên cho constructor mới này:
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

Mặc định, constructor của lớp con sẽ gọi constructor không tên, không tham số của lớp cha trước khi chạy đoạn code trong constructor's body của nó. Nếu có *initializer list* (chúng ta sẽ nói tới ở phía sau), nó lại sẽ được chạy trước constructor của lớp cha. Thứ tự chạy của từng đoạn code như sau:
1. initializer list
2. constructor không tham số của lớp cha
3. constructor không tham số của lớp hiện tại

Trong trường hợp lớp cha không có constructor không tên, không tham số, chúng ta phải gọi thay thế một constructor khác của lớp cha bằng cách sử dụng `:`
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

Initializer list là một cú pháp cho phép việc khởi tạo instance variable trước khi constructor's body được thực hiện
```
// Initializer list sets instance variables before
// the constructor body runs.
Point.fromJson(Map<String, double> json)
    : x = json['x']!,
      y = json['y']! {
  print('In Point.fromJson(): ($x, $y)');
}
```

Trong quá trình phát triển, chúng ta có thể sử dụng `assert` để  validate tham số của constructor
```
Point.withAssert(this.x, this.y) : assert(x >= 0) {
  print('In Point.withAssert(): ($x, $y)');
}
```

##### Redirecting constructor

Thỉnh thoảng, mục đích duy nhất của một constructor là gọi đến một constructor khác trong cùng class đó. Cúp pháp để gọi đến một constructor khác là
```
class Point {
  double x, y;

  // The main constructor for this class.
  Point(this.x, this.y);

  // Delegates to the main constructor.
  Point.alongXAxis(double x) : this(x, 0);
}
```

##### Constant constructor

Để khởi tạo các object là *compile-time constant*, chúng ta có thể khai báo class đó với `const` constructor và các `final` instance variable
```
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

Tuy nhiên, khi sử dụng *constant constructor*, ta vẫn phải thêm từ khóa `const` thì object được tạo mới là `compile-time constant`. Bạn có thể xem lại phần [Sử dụng constructor]()

##### Factory constructor

Sử dụng từ khóa `factory` trong trường hợp constructor không phải lúc nào cũng tạo ra một instance mới. Ví dụ: một *factory constructor* có thể trả về một instance được lấy ra từ cache hoặc trả về một instace có kiểu dữ liệu subtype. Ngoài ra, *factory constructor* còn được sử dụng để khởi tạo final variable mà đoạn code khởi tạo này không thể thực hiện trong *initializer list*
```
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to
  // the _ in front of its name.
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(
        name, () => Logger._internal(name));
  }

  factory Logger.fromJson(Map<String, Object> json) {
    return Logger(json['name'].toString());
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```
Ở đoạn code phía trên, ta thấy
- Một factory constructor của `Logger` trả về một instance lấy ra từ cache hoặc khởi tạo mới nếu instance không có trong cache.
- Constructor `Logger.fromJson` khởi tạo biến final `name` từ một JSON object

**Note**: Với *factory constructor*, ta có thể liên tưởng nó với các hàm khởi tạo static `newInstance` trong *java* hơn là các constructor thông thường. Bởi vậy, các *factory constructor* cũng cần trả về một object thay vì không trả về gì như constructor thông thường
