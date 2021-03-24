# Kế thừa đối với class

### Abstract class

*Abstract class* là các class mà không thể khởi tạo và thường được sử dụng để định nghĩa các class tổng quát với những property và method chung nhưng không được implement và để các class con định nghĩa tùy theo đặc điểm của class con. Để định nghĩa một *abstract class*, ta sử dụng từ khóa `abstract`
```
// This class is declared abstract and thus
// can't be instantiated.
abstract class AbstractContainer {
  // Define constructors, fields, methods...

  void updateChildren(); // Abstract method.

  void setChildren() {
    //do something
  }
}
```

Tương tự như *Java*, *abstract class* bao gồm cả *abstract method* và những method đã được implement.

Nếu bạn muốn *abstract class* trông có vẻ có thể khởi tạo được, bạn có thể định nghĩa cho nó một *factory constructor*
```
void main() {
  var container = AbstractContainer.factory(3);
}

abstract class AbstractContainer {
  // Define constructors, fields, methods...

  factory AbstractContainer.factory(int number) {
    return Container(5);
  }

  AbstractContainer();

  void updateChildren(); // Abstract method.
}

class Container extends AbstractContainer {

  Container.empty();

  Container(int number);

  @override
  void updateChildren() {

  }
}
```

### Implicit interfaces

Mọi class trong *Dart* đều là một interface ẩn mà interface này bao gồm các *instance member* của class đó và của các interface mà class đó implement. Nếu bạn muốn tạo một lớp A mới mà cũng hỗ trợ các API của một class B cũ thì bạn có thể khai báo A implement interface B thông qua từ khóa `implements`.
```
// A person. The implicit interface contains greet().
class Person {
  // In the interface, but visible only in this library.
  final _name;

  // Not in the interface, since this is a constructor.
  Person(this._name);

  // In the interface.
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// An implementation of the Person interface.
class Impostor implements Person {
  get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}

String greetBob(Person person) => person.greet('Bob');

void main() {
  print(greetBob(Person('Kathy')));
  print(greetBob(Impostor()));
}
```

Class A có thể implement nhiều hơn 1 class B. Sau đây là một ví dụ
```
class Point implements Comparable, Location {...}
```

### Extending a class

Để khai báo một lớp A kế thừa lớp B, ta sử dụng từ khóa `extends`. Để refer đến lớp cha, ta sử dụng từ khóa `super`
```
class Television {
  void turnOn() {
    _illuminateDisplay();
    _activateIrSensor();
  }
  // ···
}

class SmartTelevision extends Television {

  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
}
```

##### Overriding members

Lớp con có thể override các *instance method* (cả *operator method*), *getter* và *setter*. Để thể hiện chúng ta chủ ý muốn override lại các method đó, hãy sử dụng annotation `@override`
```
class SmartTelevision extends Television {
  @override
  void turnOn() {...}
  // ···
}
```

Khi override một method, chúng ta có thể làm "hẹp lại" kiểu của tham số trong các method bằng cách sử dụng từ khóa `covariant`. Tham khảo [ở đây](https://dart.dev/guides/language/sound-problems#the-covariant-keyword)

**Warning**: Nếu bạn override toán tử `==`, bạn nên override cả method `hashCode`. Tham khảo [ở đây](https://dart.dev/guides/libraries/library-tour#implementing-map-keys)

### Extension method

*Extension method* là một cách để thêm các method vào một class mà không cần kế thừa.  Chi tiết sẽ nói kỹ [ở đây]()

### Kiểu enum

Enumerated type, thường được gọi là *enumerations* hoặc *enums*, là một class đặc biệt nhằm đại điện cho một số lượng cố định các giá trị constant.

##### Sử dụng enums

Để khai báo clas là enums, ta sử dụng từ khóa `enum`
```
enum Color { red, green, blue }
```

Mỗi giá trị trong enum có một index bắt đầu từ 0
```
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

Để lấy tất cả các giá trị của enum, ta sử dụng variable `values`
```
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);
```

Bạn có thể sử dụng enum với `switch-case`. Nếu bạn không handle đủ các giá trị của enum, sẽ có warning được phát ra
```
var aColor = Color.blue;

switch (aColor) {
  case Color.red:
    print('Red as roses!');
    break;
  case Color.green:
    print('Green as grass!');
    break;
  default: // Without this, you see a WARNING.
    print(aColor); // 'Color.blue'
}
```

Những giới hạn của enum là
- Bạn không thể kế thừa, mix in hoặc implement một enum
- Bạn không thể khởi tạo một enum

Chi tiết hơn, bạn có thể tham khảo [Dart language specification](https://dart.dev/guides/language/spec)


### Mixin

Mixin là cơ chế để tái sử dụng code của một class cho nhiều class khác.

Để sử dụng mixin, ta sử dụng từ khóa `with`
```
class Musician extends Performer with Musical {
  // ···
}

class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

Để implement một mixin, ta có thể tạo một class như bình thường, không khai báo constructor nào cho class đó và thay từ khóa `class` bằng từ khóa `mixin`
```
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

Thỉnh thoảng, nếu bạn muốn giới hạn các class có thể sử dụng mixin, bạn có thể làm như sau:
```
class Musician {
  // ...
}
mixin MusicalPerformer on Musician {
  // ...
}
class SingerDancer extends Musician with MusicalPerformer {
  // ...
}
```

Trong ví dụ trên, chỉ những class nào là `Musician` (có thể là lớp con của `Musician`) là có thể sử dụng mixin `MusicalPerformer`.

### Class variable và method

Khác với *instance variable* và *instance method* đã nói ở phần trên là những member của một class và chỉ được sử dụng khi object được khởi tạo, chúng ta có thêm *class variable* và *class method* hay chính là *static variable* và *static method*.

##### Static variable

*Static variable* (hay class variable) thường được sử dụng để khai báo các constant của một class
```
class Queue {
  static const initialCapacity = 16;
  // ···
}

void main() {
  assert(Queue.initialCapacity == 16);
}
```

*Static variable* chỉ được khởi tạo ở lần đầu tiên được sử dụng.

##### Static method

Vì *static method* (class method) không chạy trên một instance nào nên *static method* cũng không thể truy cập đến `this`. *Static method* chỉ có thể truy cập đến các *static variable* và gọi đến các *static method* khác.
```
import 'dart:math';

class Point {
  double x, y;
  Point(this.x, this.y);

  static double distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```
Bạn có thể sử dụng *static method* như là một *compile-time constant* bằng cách gán nó cho một constant variable hoặc truyền n vào một constant constructor.
