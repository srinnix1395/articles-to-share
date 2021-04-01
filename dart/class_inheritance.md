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
