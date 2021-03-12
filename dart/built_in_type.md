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

*Literal number* (các ký tự đại diện cho các số) là một *compile-time constant*. Các biểu thức số học cũng là *compile-time constant* nếu các toán tử của biểu thức cũng là *compile-time constant*
```
const msPerSecond = 1000;
const secondsUntilRetry = 5;
const msUntilRetry = secondsUntilRetry * msPerSecond;
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

Ngoài ra, trong *Dart* còn có khái niệm một "raw" string, tức là một string mà không có ký tự nào được treat như là một ký tự đặc biệt. Ta thêm `r` vào phía trước String:
```
var s = r"\n \t \b";                   // \n \t \b
```

*Literal string* trong Dart là một *compile-time constant*. Nếu String đó là một biểu thức nội suy (interpolated expression), các thành phần được chèn vào phải là một số, một string, một giá trị boolean hoặc null thì nó mới là một *compile-time constant*
```
// These work in a const string.
const aConstNum = 0;
const aConstBool = true;
const aConstString = 'a constant string';

// These do NOT work in a const string.
var aNum = 0;
var aBool = true;
var aString = 'a string';
const aConstList = [1, 2, 3];

const validConstString = '$aConstNum $aConstBool $aConstString';
// const invalidConstString = '$aNum $aBool $aString $aConstList';
```


Bạn có thể tham khảo thêm một số function xử lý `String` trong thư viện `dart:core` [ở đây](https://dart.dev/guides/libraries/library-tour#strings-and-regular-expressions)

### Boolean

Để biểu diễn các giá trị boolean, *Dart* có kiểu `bool`. Chỉ có 2 object có kiểu `bool` trong *Dart*: `true` và `false`. 2 *literal boolean* này cũng là *compile-time constant*
```
bool a = true;
bool b = false;
```

### List

Đối với các ngôn ngữ khác, cấu trúc dữ liệu dạng collection có thứ tự quen thuộc nhất có lẽ là *array*. Với *Dart*, kiểu dữ liệu này là `List`.
```
var list = [1, 2, 3];
```
**Note**: Trong trường hợp ở trên, *Dart* tự suy ra kiểu của biến `list` là `List<int>`. Nếu có một phần tử không phải kiểu *int* được add vào, sẽ có compile error.

*Dart* sẽ không báo lỗi nếu tồn tại một dấu `,` ở sau phần tử cuối cùng khi khai báo một list. Dấu phẩy này sẽ giúp tránh lỗi khi bạn copy-paste phần tử cuối cùng.
```
var list = [
  'Car',
  'Boat',
  'Plane',
];
```

Tương tự như CTDL array trong các ngôn ngữ khác, `List` sử dụng zero-based indexing - tức là phần tử đầu tiên có chỉ số là 0 và phần tử cuối có chỉ số là `list.length -1 `.
```
var list = [1, 2, 3];
list[0];                  // 1
list[list.length - 1];    // 3
```

Để tạo một list không thể sửa đổi, ta thêm từ khóa `const` khi khai báo. Bạn có thể xem kỹ hơn ở phần [Final & const]()
```
var constantList = const [1, 2, 3];
constantList[1] = 1;                  // ERROR: Không thể thay đổi giá trị của một constant list.
```

Từ *Dart* 2.3, ta có thể sử dụng toán tử `...`([Spread operator](https://github.com/dart-lang/language/blob/master/accepted/2.3/spread-collections/feature-specification.md)) để insert tất cả giá trị của một list vào một list khác. Ngoài ra, nếu list được add vào có khả năng bị null, bạn có thể sử dụng thêm `?`
```
var list = [1, 2, 3];
var list2 = [0, ...list];             // list2 = [0, 1, 2, 3]
list = null;
var list3 = [0, ...?list];            // list3 = [0]
```

*Dart* cũng hỗ trợ việc tạo list với câu lệnh `if` hoặc `for`. Bạn có thể tham khảo thêm về [Control flow collections proposal](https://github.com/dart-lang/language/blob/master/accepted/2.3/control-flow-collections/feature-specification.md)
```
var nav = [
  'Home',
  'Furniture',
  'Plants',
  if (promoActive) 'Outlet'
];

var listOfInts = [1, 2, 3];
var listOfStrings = [
  '#0',
  for (var i in listOfInts) if (i & 1 == 0) '#$i'      // listOfStrings = ['#0', '#2']
];
```

Bạn có thể tham khảo thêm về Generic trong *Dart* [ở đây](https://dart.dev/guides/language/language-tour#generics) và những thao tác với List [ở đây](https://dart.dev/guides/libraries/library-tour#collections)

### Set

`Set` là một collection không có thứ tự mà các phần tử bên trong là duy nhất.
```
var halogens = {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};
```
**Note**: Tương tự như `List`, *Dart* cũng tự suy ra kiểu cho set trong trường hợp trên là `Set<String>`

Để tạo một set rỗng, ta sử dụng `{}` và chỉ rõ kiểu của set
```
var names = <String>{};
// Set<String> names = {}; // This works, too.
// var names = {};        // Creates a map, not a set.
```
**Note**: Nếu bạn không chỉ rõ kiểu của set, *Dart* sẽ suy ra biến đó có kiểu `Map<dynamic, dynamic>`

Tương tự như `List`, `Set` cũng có các phương thức `add` hay `addAll`
```
var elements = <String>{};
elements.add('fluorine');
elements.addAll(halogens);    // elements.length = 5 vì 'fluorine' đã tồn tại trong set
```

Ngoài ra, ta cũng có thể tạo một constant set
```
final constantSet = const {
  'fluorine',
  'chlorine',
  'bromine',
  'iodine',
  'astatine',
};
constantSet.add('helium');    // ERROR: không thể thay đổi giá trị của constant values
```

Tương tự như `List`, `Set` cũng hỗ trợ `...` (spread operator); toán tử `if` và `for` khi tạo một set; và [generics]()

### Map

`Map` là CTDL dạng keys-values. Cả key và value có thể là bất kỳ kiểu dữ liệu nào. Trong một map, key là duy nhất còn value thì có thể trùng nhau.
```
var gifts = {                 // Map kiểu Map<String, String>
  // Key:    Value
  'first': 'partridge',
  'second': 'turtledoves',
  'fifth': 'golden rings'
};

var nobleGases = {            // Map kiểu Map<int, String>
  2: 'helium',
  10: 'neon',
  18: 'argon',
};

var gifts = Map();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';
gifts['fifth'] = 'golden rings';
```

Một số thao tác trên map như sau:
```
var gifts = {'first': 'partridge'};
gifts['fourth'] = 'calling birds';      // Add a key-value pair
gifts['first'];                         // 'partridge'
gifts['second'];                        // null
gifts.length;                           // 2
```

Tương tự như `List` và `Set`, ta cũng có thể tạo compile-time constant; ta cũng có thể sử dụng `...`, toán tử `if` và `for` để tạo map.

### Runes và grapheme clusters

TODO
Bạn có thể tham khảo [ở đây](https://dart.dev/guides/language/language-tour#runes-and-grapheme-clusters)

### Symbol

TODO
Bạn có thể tham khảo [ở đây](https://dart.dev/guides/language/language-tour#symbols)
