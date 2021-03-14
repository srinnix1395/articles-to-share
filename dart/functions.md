# Functions

*Dart* là một ngôn ngữ hướng đối tượng, kể cả functions cũng là đối tượng trong *Dart* và cũng có kiểu dữ liệu riêng: `Function`. Thế có nghĩa là, functions có thể được gán cho một biến hoặc truyền vào các function khác như một tham số.
```
bool isEvenNumber(int number) {
  return (number & 1) == 0;
}
```

*Dart* có thể tự suy ra kiểu dữ liệu của các tham số và kiểu dữ liệu trả về của function nên bạn có thể lược bỏ việc khai báo kiểu dữ liệu. Tuy nhiên, việc này [không được khuyên khích với các public API](https://dart.dev/guides/language/effective-dart/design#prefer-type-annotating-public-fields-and-top-level-variables-if-the-type-isnt-obvious)
```
isEvenNumber(number) {          // Kiểu giá trị trả về của function này là `bool`
  return (number & 1) == 0;
}
```

Ngoài ra, đối với các function mà chỉ có một expression duy nhất, bạn cũng có thể lược bỏ luôn `{ return expression; }` và thay thế bằng `=>` (cú pháp *arrow*)
```
bool isEvenNumber(int number) => (number & 1) == 0;
```
Ở ví dụ trên, phía sau `return` là *expression*. Nếu thay bằng *statement*, kiểu giá trị trả về của function là `void`

### Parameters

Function trong *Dart* có thể gốm các loại tham số như sau:
* Tham số bắt buộc (*required positional parameters*)
* Tham số đặt tên (*named parameters*)
* Tham số vị trí tùy chọn (*optional positional parameters*)

**Note**: Chỉ một trong 2 loại tham số tùy chọn được phép có mặt

##### Required positional parameters

Tham số bắt buộc phải truyền vào và phải được khai báo trước các tham số tùy chọn khác
```
void printUserInfo(int id, String name) {
  var idString = "ID: $id\n";
  var nameString = "Name: $name\n";

  print(idString + nameString);
}
```

##### Named parameters

Để định nghĩa một tham số hoặc một nhóm tham số là *named parameter*, ta khai báo các tham số đó bên trong `{}`
```
void printUserInfo(int id, String name, {String? dateOfBirth}) {
  var idString = "ID: $id\n";
  var nameString = "Name: $name\n";
  var dateOfBirthString = "${dateOfBirth != null ? "Date of birth: $dateOfBirth" : ""}\n";

  print(idString + nameString + dateOfBirthString);
}
```

Khi truyền các tham số vào lời gọi function, ta bắt buộc phải chỉ rõ truyền giá trị nào cho tham số nào theo cú pháp `paramName: value`. Nếu không chỉ định rõ, compiler sẽ báo lỗi.
```
printUserInfo(1, "James", dateOfBirth: "30/02/1999");
printUserInfo(1, "James", "30/02/1999");          // ERROR: giá trị string cuối không biết là truyền vào cho tham số nào
```

Với trường hợp trên, *named parameter* đóng vai trò là một tham số tùy chọn. Tuy nhiên, ta cũng có thể chuyển nó thành một tham số bắt buộc bằng cách thêm `required` vào trước tham số
```
void printUserInfo(int id, String name, {required bool male, String? dateOfBirth}) {
  var idString = "ID: $id\n";
  var nameString = "Name: $name\n";
  var genderString = "Gender: ${male ? "Male" : "Female"}\n";
  var dateOfBirthString = "${dateOfBirth != null ? "Date of birth: $dateOfBirth" : ""}\n";

  print(idString + nameString + genderString + dateOfBirthString);
}
```

Và khi gọi function, ta bắt buộc phải giá trị cho tham số bắt buộc đó, đồng thời vẫn phải chỉ rõ truyền giá trị nào cho tham số nào.
```
printUserInfo(1, "James", male: true);
printUserInfo(1, "James");                         // ERROR: tham số `male` là một tham số bắt buộc
```

##### Optional positional parameters

Để khai báo một tham số hoặc một nhóm tham số là `optional positional parameters`, ta sẽ khai báo chúng bên trong `[]`
```
void printUserInfo(int id, String name, [String? dateOfBirth]) {
  var idString = "ID: $id\n";
  var nameString = "Name: $name\n";
  var dateOfBirthString = "${dateOfBirth != null ? "Date of birth: $dateOfBirth" : ""}\n";

  print(idString + nameString + dateOfBirthString);
}
```

Và khi ta gọi function, ta có thể bỏ qua các tham số đó. Các tham số tùy chọn sẽ có giá trị mặc định là `null`.
```
printUserInfo(1, "James");
```

##### Default parameter values

Để chỉ định giá trị mặc định cho 1 tham số tùy chọn (*named parameters* hoặc *optional positional parameters*), ta sử dụng `=`. Giá trị mặc định cho tham số bắt buộc phải là một *compile-time constant*. Giá trị mặc định khi không được chỉ định là `null`
```
void printUserInfo(int id, String name = "James", [bool gender = true, String dateOfBirth = "31/12/1999"]) {
  var idString = "ID: $id\n";
  var nameString = "Name: $name\n";
  var genderString = "Gender: ${male ? "Male" : "Female"}\n";
  var dateOfBirthString = "${dateOfBirth != null ? "Date of birth: $dateOfBirth" : ""}\n";

  print(idString + nameString + genderString + dateOfBirthString);
}
```

### The main() function

Mọi thứ đều phải có một điểm bắt đầu của mình. Đối với một app trong *Dart*, đó là một top-level function `main()` với một tham số tùy chọn là một `List<String>`
```
void main(List<String> arguments) {
  print('Hello, World!');
}
```

### Functions as first-class objects

Bạn có thể truyền một function vào một function khác như là một tham số.
```
void printElement(int element) {
  print(element);
}

var list = [1, 2, 3];

// Pass printElement as a parameter.
list.forEach(printElement);
```

Ngoài ra, bạn cũng có thể gán một function cho một biến
```
var printFunction = (element) => printElement(element);
list.forEach(printFunction);
```

### Anonymous functions

Trong trường hợp gán một function cho một biến như ở trên, thay vì khai báo `printElement()`, ta có thể chuyển function đó thành một *anonymous functions*
```
var printFunction = (element) => print(element);
list.forEach(printFunction);
```

*Anonymous functions* (hay thỉnh thoảng được gọi là *lambda* hay *closure*) là các function mà không có tên. Tuy nhiên, các đặc điể khác thì vẫn giống một function bình thường: có tham số hoặc không có tham số, có tham số bắt buộc và tham số tùy chọn...
```
([[Type] param1[, …]]) {
  // function content
};
```

Một trong những ví dụ chúng ta thường hay gặp đối với *anonymous functions* là
```
var list = [1, 2, 3];
list.forEach((item) {
  print('${list.indexOf(item)}: $item');
});
```

Nếu *anonymous functions* chỉ chứa một biểu thức hoặc một câu lệnh duy nhất, bạn có thể sử dụng cú pháp rút gọn `=>` (*arrow syntax*)
```
list.forEach(item => print('${list.indexOf(item)}: $item'));
```

### Lexical scope

*Dart* là một ngôn ngữ *lexical scoping*, tức là scope của các biến được xác định tĩnh, thông qua layout của code (cụ thể là thông qua "{}")
```
bool topLevel = true;

void main() {
  var insideMain = true;

  void myFunction() {
    var insideFunction = true;

    void nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

### Lexical closures

Một *closure* là một function object mà có khả năng truy cập đến các biến nằm trong lexical scope của nó, dù cho function đó được sử dụng bên ngoài scope ban đầu
```
/// Returns a function that adds [addBy] to the
/// function's argument.
Function makeAdder(int addBy) {
  return (int i) => addBy + i;
}

void main() {
  // Create a function that adds 2.
  var add2 = makeAdder(2);

  // Create a function that adds 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```
Trong trường hợp ở trên, `makeAdder()` đã giữ biến `addBy` nên khi được gọi sau đó, ta vẫn có giá trị của biến `addBy`.

### Return values

Tất cả các function đều trả về một giá trị nào đó. Nếu một function không được chỉ định kiểu trả về và không có giá trị trả về, câu lệnh `return null;` sẽ được thêm vào function
```
foo() {}
assert(foo() == null);
```

### Callable classes

Với bất kỳ class nào trong *Dart* bạn có thể implement function `call()` để instance của class đó có thể được gọi như là một function. Và bạn có thể implement function `call()` theo bất kỳ cách nào mà bạn muốn.

```
class CallableClass {
  void call(int id, String name){
    print("ID: $id Name: $name");
  }
}

var callableClass = CallableClass();
callableClass(1, "James");
```
