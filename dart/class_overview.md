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

### Gettting an object's type

Để get kiểu của object lúc run-time, chúng ta có thể sử dụng property `runtimeType` có sẵn của `Object` và trả về object kiểu `Type`.
```
print('The type of a is ${a.runtimeType}');
```

Vậy là với 2 phần cơ bản trên đây, chúng ta đã biết một số thao tác cơ bản như truy cập đến các member của class và lấy kiểu của class đó. Các phần tiếp theo sẽ nói chi tiết hơn về các thành phần tạo nên class:
- [Instance variable]()
- [Constructor]()
- [Method]()
- [Inheritance]()
