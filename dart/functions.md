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

Function trong *Dart* có thể gốm các loại tham số như
* Tham số bắt buộc (*required positional parameters*)
* Tham số đặt tên (*named parameters*)
* Tham số vị trí tùy chọn (*optional positional parameters*)

**Note**: Chỉ một trong 2 loại tham số tùy chọn được phép có mặt

##### Required positional parameters

Tham số bắt buộc phải truyền vào và phải được khai báo trước các tham số tùy chọn khác
```
void printUserInfo(int id, String name) {
  print("ID: $id\nName: $name");
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

Và khi gọi function, ta vẫn phải chỉ rõ truyền giá trị nào cho parameter nào.
```
printUserInfo(1, "James", male: true);
printUserInfo(1, "James";                         // ERROR: tham số `male` là một tham số bắt buộc
```

##### Optional positional parameters

Để khai báo một tham số hoặc một nhóm tham số là `optional positional parameters`, ta sẽ khai báo chúng bên trong `[]`
### Callable classes
