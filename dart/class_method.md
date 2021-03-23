# Class's method

Method là function của class và thể hiện các hành vi của một đối tượng.

### Instance method

*Instance method* có thể truy cập *instance variable* thông qua từ khóa `this`
```
import 'dart:math';

const double xOrigin = 0;
const double yOrigin = 0;

class Point {
  double x = 0;
  double y = 0;

  Point(this.x, this.y);

  double distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }
}
```

### Operator

Operator là các method có tên đặc biệt là các toán tử như `+`, `-`,... mà khi được định nghĩa bên trong class, chúng ta có thể sử dụng các đối tượng trong các phép toán với các toán tử đó. *Dart* cho phép ta định nghĩa các toán tử sau

| `<`  |  `+` | `\|` | `[]`  |
|------|------|------|-------|
|  `>` |  `/` | `^`  | `[]=` |
| `<=` | `~/` | `&`  |  `~`  |
| `>=` | `*`  | `<<` | `==`  |
| `–`  | `%`  | `>>` |       |

**Note**: Có thể bạn sẽ thắc mắc khi không tìm thấy một số toán tử `!=`. Đó là vì `!=` thực chấn là một syntactic sugar - một toán tử rút gọn của `==`: `!(e1 == e2)`

Để định nghĩa một operator method, ta sử dụng từ khóa `operator` theo cú pháp sau
```
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  // Operator == and hashCode not shown.
  // ···
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```

**Note**: chúng ta cần tham khảo signature có sẵn của các operator method trên mạng trước khi định nghĩa lại.


### Getter và setter

Getter và setter là các method đặc biệt để các đối tượng bên ngoài truy cập đến thuộc tính của object. Trong *Dart*, mỗi instance variable đều có một getter method được tự động gen và một setter method nếu đủ điều kiện. Ngoài ra, chúng ta có thể có tạo ra thêm các property bằng cách định nghĩa các getter và setter method bằng cách sử dụng `get` và `set`
```
class Rectangle {
  double left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // Define two calculated properties: right and bottom.
  double get right => left + width;
  set right(double value) => left = value - width;

  double get bottom => top + height;
  set bottom(double value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

Trong ví dụ trên, để override lại getter và setter được gen tự động của instance variable `left`. Ta làm như sau
```
  double get leftValue {
    return left;
  }

  set leftValue(double value) {
    left = value;
  }
```

Cá nhân mình thấy các khai báo các hàm getter setter như trên không được tường minh cho lắm khi chúng ta có thể đặt tên cho getter và setter như thế nào cũng được, dẫn đến việc chúng ta có cảm giác không phải là đang override lại getter setter method có sẵn mà là tạo ra hẳn một function mới. Tuy nhiên, khi ta gọi hàm getter mà ta vừa override lại thì có vẻ nó lại trông giống như đang gọi đến một variable khác hơn
```
var leftValue = rect.leftValue;
```

### Abstract method

Instance method, getter method và setter method có thể được định nghĩa là các abstract method - các method có thân hàm trống và nằm trong các *abstract class*, bằng cách để trống method's body
```
abstract class Doer {
  // Define instance variables and methods...

  void doSomething(); // Define an abstract method.
}

class EffectiveDoer extends Doer {
  void doSomething() {
    // Provide an implementation, so the method is not abstract here...
  }
}
```
