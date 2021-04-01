# Class variable và method

Khác với *instance variable* và *instance method* đã nói ở phần trên là những member của một class và chỉ được sử dụng khi object được khởi tạo, chúng ta có thêm *class variable* và *class method* hay chính là *static variable* và *static method*.

### Static variable

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

### Static method

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
