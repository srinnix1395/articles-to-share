# Operators

Bảng sau liệt kê những toán tử mà *Dart* hỗ trợ

| Mô tả   	               | Toán tử  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| Một ngôi hậu tố  	       | `expr++`   ` expr--`    `()`    `[]`    `.`   ` ?.`                    |
| Một ngôi tiền tố  	     | `-expr`    `!expr`    `~expr`    `++expr`    `--expr`      `await expr`|
| Nhân chia  	             | `*`    `/`    `%`  `~/`                                                |
| Cộng trừ  	             | `+`    `-`  	                                                          |
| Dịch bit  	             | `<<`    `>>`    `>>>`                                                 	|
| Thao tác bit AND  	     | `&`  	                                                                |
| Thao tác bit XOR  	     | `^`  	                                                                |
| Thao tác bit OR  	       | `\|`  	                                                                |
| So sánh và kiểm tra kiểu | `>=`    `>`    `<=`    `<`    `as`    `is`    `is!`  	                |
| So sánh bằng nhau  	     | `==`    `!=`                                                         	|
| Luận lý AND  	           | `&&`  	                                                                |
| Luận lý OR  	           | `\|\|`  	                                                              |
| if null  	               | `??`  	                                                                |
| Điều kiện  	             | `expr1 ? expr2 : expr3`  	                                            |
| Cascade  	               | `..`    `?..`  	                                                      |
| Gán  	                   | `=`    `*=`    `/=`   `+=`   `-=`   `&=`   `^=`                        |

Trong bảng trên, các toán tử nào càng ở trên cao thì càng có độ ưu tiên cao và ngược lại. Ví dụ: toán tử `%` có độ ưu tiên cao hơn (nên cũng sẽ được chạy trước) toán tử `==` và đến lượt mình, toán tử `==` lại có độ ưu tiên cao hơn toán tử logic AND là `&&`. Bởi vậy, 2 đoạn code sau sẽ có kết quả giống nhau:
```
// Parentheses improve readability.
if ((n % i == 0) && (d % i == 0)) ...

// Harder to read, but equivalent.
if (n % i == 0 && d % i == 0) ...
```

**Note**: Với những toán tử có 2 ngôi, toán tử bên trái sẽ xác định method nào được sử dụng. VD: bạn có một `Vector` object và 1 `Point` object thì biểu thức `aVector + aPoint` sẽ sử dụng `+` của `Vector` object.

### Arithmetic operators - Toán tử số học

*Dart* hỗ trợ các toán tử số học thông thường

| Toán tử 	               | Ý nghĩa  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| `+`              	       | Cộng                                                                   |
| `-              ` 	     | Trừ                                                                    |
| `-expr`   	             | Số âm                                                                  |
| `*`       	             | Nhân                                                                   |
| `/`  	                   | Chia                                                                   |
| `~/`              	     | Chia lấy nguyên                                                        |
| `%`               	     | Chia lấy dư                                                            |

Ví dụ:
```
assert(2 + 3 == 5);
assert(2 - 3 == -1);
assert(2 * 3 == 6);
assert(5 / 2 == 2.5); // Result is a double
assert(5 ~/ 2 == 2); // Result is an int
assert(5 % 2 == 1); // Remainder

assert('5/2 = ${5 ~/ 2} r ${5 % 2}' == '5/2 = 2 r 1');
```

*Dart* cũng hỗ trợ các toán tử tiền tố và hậu tố  để tăng giá trị hoăc giảm giá trị

| Toán tử 	               | Ý nghĩa  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| `++var`                  | var = var + 1 (Giá trị của biểu thức là var + 1)                       |
| `var++`           	     | var = var + 1 (Giá trị của biểu thức là var)                           |
| `--var `   	             | var = var – 1 (Giá trị của biểu thức là var – 1)                       |
| `var--`       	         | var = var – 1 (Giá trị của biểu thức là var)                           |

Ví dụ:
```
var a, b;

a = 0;
b = ++a; // Tăng a TRƯỚC khi gán cho b giá trị.
assert(a == b); // 1 == 1

a = 0;
b = a++; // Tăng a SAU khi gán cho b giá trị.
assert(a != b); // 1 != 0

a = 0;
b = --a; // Giảm a TRƯỚC khi gán cho b giá trị.
assert(a == b); // -1 == -1

a = 0;
b = a--; // Giảm a SAU khi gán cho b giá trị.
assert(a != b); // -1 != 0
```

### Equality and relational operators - Toán tử quan hệ

Các toán tử so sánh trong *Dart* bao gồm

| Toán tử 	               | Ý nghĩa  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| `==`                     | So sánh bằng                                                           |
| `!=`           	         | So sánh khác                                                           |
| `>`   	                 | Lớn hơn                                                                |
| `<`       	             | Nhỏ hơn                                                                |
| `>=`       	             | Lớn hơn hoặc bằng                                                      |
| `<=`       	             | Nhỏ hơn hoặc bằng                                                      |

Để kiểm tra 2 đối tượng x và y có bằng nhau không về mặt giá trị không (tương đương với function `equals()` trong *Java*), ta sử dụng toán tử  `==`. Ta có thể override lại operator `==` để xác định các tiêu chuẩn so sánh. Cách toán tử `==` hoạt động như sau:
1. Nếu x và y đều null, kết quả trả về là `true`. Nếu một trong 2 null, kết quả trả về là `false`
2. Function operator `==` được gọi và kết quả được trả về: `x.==(y)`

Ngoài ra, nếu muốn kiểm tra 2 đối tượng có phải trỏ tới cùng 1 object hay không, ta sử dụng fucntion [`identical()`](https://api.dart.dev/stable/2.12.1/dart-core/identical.html) trong thư viện *dart:core*

### Type test operators - Toán tử kiểm tra kiểu dữ liệu

Các toán tử kiểm tra kiểu dữ liệu trong *Dart* bao gồm

| Toán tử 	               | Ý nghĩa  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| `as`                     | Toán tử ép kiểu                                                        |
| `is`           	         | So sánh kiểu dữ liệu, trả về `true` nếu giống kiểu muốn so sánh        |
| `is!`           	       | So sánh kiểu dữ liệu, trả về `true` nếu không giống kiểu muốn so sánh  |

Bạn có thể ép kiểu trực tiếp nếu tự tin
```
(employee as Person).firstName = 'Bob';
```

Tuy nhiên, chúng ta nên kiểm tra kiểu dữ liệu trước khi gọi để tránh crash
```
if (employee is Person) {
  // Type check
  employee.firstName = 'Bob';
}
```

### Assignment operators - Toán tử gán

Để gán một giá trị, chúng ta sử dụng toán tử `=`. Để gán một giá trị cho một biến chỉ trong trường hợp biến đó `null`, chúng ta sử dụng toán tử `??=`
```
// Gán giá trị cho a
a = value;
// Gán giá trị cho b nếu b null; nếu không, b vẫn giữ nguyên giá trị
b ??= value;
```

Ngoài ra, chúng ta có các toán tử gán kết hợp

|       |        |       |       |       |      |
|-------|--------|-------|-------|-------|------|
|  `=`  |  `-=`  | `/=`  | `%=`  | `>>=` | `^=` |
|  `+=` |  `*=`  | `~/=` | `<<=` | `&=`  | `|=` |

Các toán tử này là kết hợp của một phép toán và theo sau là một phép gán. 2 biểu thức sau có kết quả tương đương nhau:
```
a = a + b;
a += b;
```

### Logical operators - Toán tử luận lý

Các toán tử luận lý trong *Dart* bao gồm:

| Toán tử 	               | Ý nghĩa  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| `!expr`                  | Toán tử phủ định                                                       |
| `||`           	         | Toán tử logic OR                                                       |
| `&&`             	       | Toán tử logic AND                                                      |

### Bitwise and shift operators - Toán tử thao tác trên bit

| Toán tử 	               | Ý nghĩa  	                                                            |
|--------------------------|------------------------------------------------------------------------|
| `&`                      | AND                                                                    |
| `|`           	         | OR                                                                     |
| `^`             	       | XOR                                                                    |
| `~expr`             	   | Toán tử lấy phần bù một ngôi                                           |
| `<<`             	       | Dịch bit số học trái                                                   |
| `>>`             	       | Dịch bit số học phải                                                   |

Ví dụ
```
final value = 0x22;
final bitmask = 0x0f;

assert((value & bitmask) == 0x02); // AND
assert((value & ~bitmask) == 0x20); // AND NOT
assert((value | bitmask) == 0x2f); // OR
assert((value ^ bitmask) == 0x2d); // XOR
assert((value << 4) == 0x220); // Shift left
assert((value >> 4) == 0x02); // Shift right
```

### Conditional expressions - Biểu thức điều kiện

*Dart* có 2 toán tử hỗ trợ biểu thức điều kiện
* `condition ? expr1 : expr2`: Toán tử trả về  `expr1` nếu `condition` đúng và `expr2` nếu `condition` sai
```
var visibility = isPublic ? 'public' : 'private';
```

* `expr1 ?? expr2`: Trả về toán tử `expr1` nếu nó không null và trả về `expr2` nếu `expr1` null.
```
String playerName(String? name) => name ?? 'Guest';
```

### Cascade notation

Các ký pháp `..` hay `?..` cho phép bạn thực hiện một chuỗi các thao tác trên cùng một đối tượng. Các thao tác đó có thể là truy cập các thuộc tính hoặc gọi các function. Việc sử dụng các ký pháp này sẽ giúp code của bạn trông có vẻ "mượt mà" hơn
```
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;
```

Đoạn code trên sẽ tương đương với đoạn code sau:
```
var paint = Paint();
paint.color = Colors.black;
paint.strokeCap = StrokeCap.round;
paint.strokeWidth = 5.0;
```

Trong trường hợp đối tượng gọi có thể null, bạn có thể sử dụng `?..` cho thao tác đầu tiên. Việc này sẽ đảm bảo rằng nếu đối tượng gọi bằng null, không một thao tác nào được thực hiện
```
querySelector('#confirm') // Get an object.
  ?..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

Ngoài ra, bạn cũng có thể sử dụng `..` bên trong một thao tác
```
final addressBook = (AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

### Other operators - Các toán tử khác

Cuối cùng, ta có một số toán tử sau

| Toán tử 	    | Tên                         | Ý nghĩa  	                                                             |
|---------------|-----------------------------|------------------------------------------------------------------------|
| `()`          | Function ứng dụng           | Lời gọi function                                                       |
| `[]`          | Truy cập list	              | Truy cập đến một phần tử trong list                                    |
| `.`           | Truy cập class 	            | Truy cập đến một thuộc tính của đối tượng                              |
| `?.`          | Truy cập class có điều kiện | Truy cập đến một thuộc tính của đối tượng nếu đối tượng đó không null  |
