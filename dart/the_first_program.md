# Chương trình đầu tiên

Đoạn code sau sử dụng hầu hết các feature cơ bản của *Dart*

```
// Định nghĩa một function
void printInteger(int aNumber) {
  print('The number is $aNumber.'); // In ra console.
}

// Function main(), nơi chương trình bắt đầu chạy.
void main() {
  var number = 42; // Khai báo và khởi tạo một biến
  printInteger(number); // Gọi một function.
}
```

Ở đây, ta sẽ thấy một số feature quen thuộc:


`// This is a comment.`
Comment một dòng. Ngoài ra, *Dart*  cũng support comment nhiều dòng hoặc document comment. Chúng ta sẽ xem xét kỹ hơn ở [Comment]()

void
    A special type that indicates a value that’s never used. Functions like printInteger() and main() that don’t explicitly return a value have the void return type.

int
    Another type, indicating an integer. Some additional built-in types are String, List, and bool.

42
    A number literal. Number literals are a kind of compile-time constant.

print()
    A handy way to display output.
'...' (or "...")
    A string literal.

$variableName (or ${expression})
    String interpolation: including a variable or expression’s string equivalent inside of a string literal. For more information, see Strings.

main()
    The special, required, top-level function where app execution starts. For more information, see The main() function.

var
    A way to declare a variable without specifying its type. The type of this variable (int) is determined by its initial value (42).

### Các khái niệm quan trọng

* Mọi giá trị bạn có thể gán cho một biến đều là một *object* (và mọi *object* đều là một instance của một *class*) dù giá trị đó là kiểu số, là một function hay `null`. Mọi *object* đều được kế thừa từ kiểu `Object`.
```
int a = 3;
var b = () =>{
  //do nothing
};
var c = null;
```

* *Dart* là một ngôn ngữ *strongly type* (tức là với mỗi biến, kiểu của biến đó được xác định lúc compile). Tuy nhiên, chúng ta cũng có thể không cần chỉ rõ kiểu của biến lúc khai báo mà có thể để *Dart* tự suy ra dựa vào giá trị gán cho biến đó.
```
var a = 3;        // a có kiểu dữ liệu int
a = "3";          // ERROR: không thể gán cho một giá trị kiểu String
```

* Với mỗi biến, bạn có thể xác định xem nó có thể bằng `null` một lúc nào đó không bằng cách thêm dấu `?` vào sau kiểu dữ liệu - *nullable type*.
```
String? a;        // a = null
String b = null;  // Error: b không phải kiểu dữ liệu nullable
```
Với một biến kiểu nullable, giá trị mặc định của biến đó sẽ bằng null nếu không được gán giá trị. Ngược lại, bạn buộc phải gán một giá trị cho một biến non-nullable trước khi sử dụng biến đó.
```
String? a;        
String b;
print(a);         // null
print(b);         // Error: Biến b chưa  được khởi tạo
```
Khi bạn chắc chắn một biến nullable nào đó sẽ không thể bằng null được, bạn có thể thêm dâu `!` vào sau biến để chuyển biến đó thành kiểu non-nullable
```
String? a = "3";
String b = a;     // Error: Không thể gán một giá trị nullable cho biế n có kiểu non-nullable
String c = a!;
```

* Khi bạn muốn một biến có thể gán bất kỳ kiểu dữ liệu nào, bạn có thể  sử dụng kiểu `Object?` hoặc `Object` nếu muốn biến đó chấp nhận tất cả các đối tượng:
```
Object? a = "3";
a = null;
Object b = "3";
b = 3;
```
Hoặc sử dụng kiểu `dynamic` nếu bạn muốn thay đổi kiểu của biến lúc run-time
```
var a = 1;        // a có kiểu int
a = 3;            
a = "abc";        // ERROR: Kiểu của a đã là int nên ta không thể gán một String cho a
dynamic b = 3;    // b có kiểu int
b = 1;            
b = "abc";        // Lúc này, b có kiểu dữ liệu là String thay vì int
```
Tuy nhiên, việc sử dụng `dynamic` là [không được khuyến khích](https://dart.dev/guides/language/effective-dart/design#avoid-using-dynamic-unless-you-want-to-disable-static-checking).

* *Dart* hỗ trợ kiểu *generic*: `List<int>` hoặc `List<Object?>`

* Ngoài các function gắn với class (*static method*) và function gắn với object (*instance method*), *Dart* hỗ trợ thêm *top-level function*. Ngoài ra, bạn có thể khai báo một function bên trong một function (*nested function* hay *local function*). Chúng ta sẽ xem xét kỹ hơn ở phần [Function]()

* Tương tự như function, *Dart* hỗ trợ *static variable*, *instance variable* và *top-level variable*. Chúng ta sẽ xem xét kỹ hơn ở phần [Variable]()

* Không giống với *Java*, Dart không có các *visibility modifier*. Thay vào đó, nếu một biến có tên bắt đầu bằng ký hiệu `_`, biến đó sẽ private với library của nó. Chúng ta sẽ xem xét kỹ hơn ở phần [Library và visibility]()

* Tên của một biến trong *Dart* có thể bắt đầu bằng một chữ hoặc ký hiệu `_` và theo sau bởi số hoặc chữ.

* Dart hỗ  trợ cả biểu thức (expression - có trả về giá trị): `condition ? expr1 : expr2` và câu lệnh (statement - không trả về giá trị): câu lệnh if else.

* Trong quá trình phát triển, bạn có thể gặp 2 vấn đề: *Warning* hoặc *Error*. *Warning* là chỉ dẫn rằng code của bạn có thể có lỗi gì đó nhưng không ngăn việc thực thi code. *Error* thì có thể là *compile-time error* (Ngăn không cho bạn thực thi code) hoặc *run-time error* (Throw ra exception và có thể làm app của bạn crash).

### Từ khóa

Trong *Dart*, ta có các từ khóa mà ta nên tránh việc sử dụng để làm định danh. Tuy nhiên, vẫn có những ngoại lệ. Bạn có thể tham khảo [ở đây](https://dart.dev/guides/language/language-tour#keywords)
