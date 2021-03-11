# Built-in type

*Dart* hỗ trợ những kiểu dữ liệu sau:
* [Số](): `int` và `double`
* [String](): `String`
* [Boolean](): `bool`
* [List](): `List`
* [Set](): `Set`
* [Map](): `Map`
* [Rune](): `Runes`
* [Symbol](): `Symbol`
* Giá trị `null`

Mọi biến trong *Dart* đều refer đến một object - một instance của một *class*. Bởi vậy, bạn có thể khởi tạo một biến bằng *constructor* hoặc bằng literal (VD: String literal - `"This a a string"`, boolean literal - `true`)

Ngoài ra, một số  kiểu dữ liệu có sẵn có những vai trò đặc biệt trong *Dart* như:
* `Object`: Lớp cha của mọi class trong *Dart* trừ `Null`
* `Future` và `Stream`: Sử dụng trong [Hỗ trợ bất đồng bộ]()
* `Iterable`: Sử dụng ở [vòng lặp for-in]() và [các hàm sinh đồng bộ]()
* `Never`: Thể hiện một biểu thức sẽ không bao giờ được evaluate thành công. Thường sử dụng cho các function mà luôn luôn throw exception.
* `dynamic`: Thể hiện rằng biến có thể thay đổi kiểu dữ liệu lúc run-time
* `void`: Thường sử dụng đối với function để biểu thị rằng function đó không trả về kết quả gì

Tiếp theo, chúng ta sẽ đi vào chi tiết của từng kiểu dữ liệu cơ bản trong *Dart*

### Number

Kiểu dữ liệu đại diện cho số trong *Dart* là `num`. Tuy nhiên, 2 kiểu dữ liệu con được sử dụng nhiều hơn là:
* `int`: biểu diễn các số nguyên có độ dài lớn nhất là 64 bit, tùy thuộc vào platform. Với các nền tảng sử dụng máy ảo Dart (*Dart virtual machine*)), khoảng giá trị của `int` là từ -2^63 đến 2^63 - 1. Với web, code của *Dart* được compile ra code *JavaScript* và sẽ sử dụng kiểu giá trị số của *JavaScript*, khoảng giá trị là -2^53 đến 2653 -1.
* `double`: biểu diễn các số thực có độ dài 64bit.
```
var a = 1;                // *Dart* tự suy ra biến có kiểu int
var b = 1.1;              // *Dart* tự suy ra biến có kiểu double
```

Để chuyển một biến kiểu `String` thành kiểu `num` và ngược lại, ta làm như sau:
```
// String -> int
var one = int.parse('1');
assert(one == 1);

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');
```

Với dữ liệu kiểu `num`, ta có sẵn các toán tử +, -, *, / và một số function nh `abs()`, `ceil()` hay `floor()`. Với riêng `int`, ta có thêm các toán tử thao tác trên bit. Ngoài ra, bạn có thể tìm thêm các function thao tác về toán học ở thư viện [dart:math]()
```
(-1).abs();                 // 1
(1.3).floor();              // 1
(1.3).ceil();               // 2

(3 << 1);                   // 6
(3 >> 1);                   // 1
(3 | 4);                    // 7
(3 ^ 1);                    // 2
```

### String

`String` trong *Dart* là một chuỗi các ký tự UTF-16. Để thể hiện `String`, bạn có thể dùng ký hiệu `'` hoặc `""` đều được:
```
var s1 = 'Single quotes work well for string literals.';
var s2 = "Double quotes work just as well.";
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

Để chèn một biểu thức vào String, bạn có thể dùng `${expression}`:
```
var s = "world";
print("Hello $s");                    // Hello world
print("HELLO ${s.toUpperCase()}");    // HELLO WORLD
```

Một feature giống với *Kotlin* là để tạo một string gồm nhiều dòng, ta có thể sử dụng triple quote `'''`:
```
var s1 = '''
You can create
multi-line strings like this one.
''';
```

Ngoài ra, để có một "raw" string, tức là một string mà không có ký tự nào được treat như là một ký tự đặc biệt:
```
var s = r"\n \t \b";                   // \n \t \b
```

Bạn có thể tham khảo thêm một số xử lý với `String` trong thư viện `dart:core` [ở đây](https://dart.dev/guides/libraries/library-tour#strings-and-regular-expressions)
