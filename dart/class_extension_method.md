# Extension methods

Extension method được thêm vào từ *Dart 2.7*, là một cách để thêm một method vào một library (có thể là một class) mà không cần kế thừa nó.

### Overview

Khi bạn đang sử dụng một API của một ai đó hoặc khi bạn đang implement một library được sử dụng rộng rãi, thật khó để có thể sửa đổi API đó. Tuy nhiên, bạn vẫn có cách để thêm các chức năng.

VD: để parse một string thành một số nguyên:
```
int.parse('42');
```

Tuy nhiên, sẽ là ngắn gọn, đẹp đẽ và dễ hiểu hơn nếu ta có một method làm công việc tương tự đối với một object String:
```
'42'.parseInt();
```

Để sử dụng function `parseInt()` cho String, bạn có thể import một thư viện có chứa các extension method của class `String`
```
import 'string_apis.dart';
// ···
print('42'.parseInt()); // Use an extension method.
```

Chúng ta có thể định nghĩa ngoài *extension method* là *extension getter*, *setter* hay *operator*. *Extension method* `parseInt()` ở trên được implement như sau:
```
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }
  // ···
}
```

### Sử dụng extension method

*Extension method* được khai báo trong các library. Vì vậy, khi muốn sử dụng, bạn chỉ cần import library vào và gọi như một method bình thường
```
// Import a library that contains an extension on String.
import 'string_apis.dart';
// ···
print('42'.padLeft(5)); // Use a String method.
print('42'.parseInt()); // Use an extension method.
```

##### Static type và dynamic type

Bạn không thể gọi một *extension method* đối với một biến có kiểu `dynamic`. Ví dụ: đoạn code sau sẽ có lỗi
```
dynamic d = '2';
print(d.parseInt()); // Runtime exception: NoSuchMethodError
```

The reason that dynamic doesn’t work is that extension methods are resolved against the static type of the receiver. Because extension methods are resolved statically, they’re as fast as calling a static function.

##### API conflicts

Khi xảy ra api conflict với một interface hoặc một *extension member* lúc gọi *extension method*, bạn có vài lựa chọn để giải quyết như sau:

Lựa chọn đầu tiên là thay đổi cách bạn import các extension mà gây ra conflict, sử dụng `show` và `hide` để giới hạn việc import
```
// Định nghĩa một extension method parseInt().
import 'string_apis.dart';

// Cúng định nghĩa một extension method parseInt(), nhưng ta sẽ ẩn NumberParsing2
// vào để tránh xảy ra conflict.
import 'string_apis_2.dart' hide NumberParsing2;

// ···
// Sử dụng parseInt() được định nghĩa ở 'string_apis.dart'.
print('42'.parseInt());
```

Lựa chọn thứ 2 là chỉ định rõ xem *extension method* được sử dụng chính xác được khai báo ở đâu
```
// Cả 2 libary đều định nghĩa extension method cho String là parseInt(),
import 'string_apis.dart'; // Contains NumberParsing extension.
import 'string_apis_2.dart'; // Contains NumberParsing2 extension.

// ···
// print('42'.parseInt()); // Không rõ parseInt() này là parseInt() nào
print(NumberParsing('42').parseInt());
print(NumberParsing2('42').parseInt());
```

Lựa chọn cuối cùng là nếu cả 2 extension đều có cùng tên, ta có thể sử dụng một prefix để phân biệt 2 library khác nhau khi import.
```
import 'string_apis.dart';
import 'string_apis_3.dart' as rad;

// ···
// print('42'.parseInt()); // Không rõ parseInt() này là parseInt() nào

// Use the ParseNumbers extension from string_apis.dart.
print(NumberParsing('42').parseInt());

// Use the ParseNumbers extension from string_apis_3.dart.
print(rad.NumberParsing('42').parseInt());

// Only string_apis_3.dart has parseNum().
print('42'.parseNum());
```

Như ở lựa chọn cuối cùng, bạn có thể gọi *extension method* mà không cần chỉ rõ xem method đó ở đâu.

### Implement extension method

Cú pháp để khai báo extension method là
```
extension <extension name> on <type> {
  (<member definition>)*
}
```

Ví dụ:
```
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }

  double parseDouble() {
    return double.parse(this);
  }
}
```

Để tạo ra các local extension, tức là các extension chỉ có thể được sử dụng bên trong library, bạn có thể lược bỏ tên của extension hoặc thêm `_` vào tên của extension.
```
extension on String {
  // code
}

extension _NumberParsing on String {
  // code
}
```

Bên trong một extension, chúng ta có thể khai báo các method, getter, setter hoặc operator. Ngoài ra, chúng ta còn có thể khai báo static field và static method.

### Implement generic extension

Extension có thể được sử dụng với generic
```
extension MyFancyList<T> on List<T> {
  int get doubleLength => length * 2;
  List<T> operator -() => reversed.toList();
  List<List<T>> split(int at) => <List<T>>[sublist(0, at), sublist(at)];
}
```

Kiểu `T` là static type.
