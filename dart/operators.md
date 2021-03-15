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
