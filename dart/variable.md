# Variable

### Khai báo

Để khai báo một biến, ta có thể sử dụng từ khóa `var` và *Dart* sẽ tự suy ra kiểu dữ liệu dựa vào giá trị được gán.
```
var name = "bob";     // name có kiểu dữ liệu String
```

Nếu bạn muốn chỉ định rõ kiểu dữ liệu cho biến, thay `var` bằng kiểu dữ liệu
```
String? name = null;  // name có kiểu dữ liệu String?
```

Ngoài ra, nếu bạn muốn khai báo một biến có kiểu dữ liệu không cố định (tức là có thể thay đổi được lúc run-time), hãy sử dụng `dynamic`. Tuy nhiên, `dynamic` [không được khuyên dùng](https://dart.dev/guides/language/effective-dart/design#avoid-using-dynamic-unless-you-want-to-disable-static-checking).
```
dynamic name = "a";
name = 4;
```

### Giá trị mặc định

Khi một biến được khởi tạo, nếu biến đó có kiểu dữ liệu nullable, giá trị mặc định của biến đó khi chưa được gán là `null` dù biến đó có kiểu nào đi chăng nữa.
```
String? a;            // default value: null
int? b;               // default value: null
```

Nếu biến đó có kiểu dữ liệu non-nullable, bạn phải gán cho nó khi khai báo. Nếu không, sẽ có compile-time error xảy ra.
```
int a;                // ERROR: chưa gán giá trị cho a mà đã sử dụng
int b = 4;
```

Tuy nhiên, nếu biến đó là một local variable, ta không cần khởi tạo ngay lúc khai báo mà chỉ cần gán cho n một giá trị trước khi sử dụng lần đầu tiên.
```
int a;
a = 3;
print(a);
```

### Late variable

Từ *Dart* version 2.12, từ khóa `late` được thêm vào nhằm xử lý 2 vấn đề:
* Khai báo một biến non-nullable mà không cần gán cho ngay cho nó một giá trị cụ thể.
* Khởi tạo chậm một biến: biến sẽ được khởi tạo khi biến được sử dụng lần đầu tiên.

##### Khai báo mà không cần gán giá trị

Thông thường, *control flow analysis* của *Dart* có thể xác định xem một biến non-nullable đã được gán giá trị trước khi sử dụng chưa và báo lỗi nếu biến đó chưa được gán giá trị. Tuy nhiên, bạn có thể thêm `late` khi khai báo biến đó báo cho compiler rằng:"Biến này sẽ được gán giá trị, nhưng không phải ngay bây giờ" và compiler sẽ không báo lỗi nữa.
```
late String description;

void main() {
  description = 'Feijoada!';
  print(description);
}
```
Note: Tuy nhiên, nếu việc gán giá trị cho biến không xảy ra, chương trình của bạn có thể bị crash khi chạy.

##### Khởi tạo chậm

Khi bạn khai báo một biến với từ khóa `late`, bạn có thể cung cấp cho n một *initializer*(một hàm dùng để khởi tạo) để đánh dấu rằng:"Biến này sẽ được gán giá trị thông qua *initializer* khi biến đó được sử dụng lần đầu tiên, nhưng không phải bây giờ".
```
late String temperature = _readThermometer();
```

Việc khởi tạo chậm này khá là tiện trong một số trường hợp:
- Việc khởi tạo biến đó không thật sự cần thiết ở thời điểm khai báo và việc khởi tạo biến đó cũng là một việc không nhẹ nhàng gì. Trong trường hợp trên, nếu biến `temperature` không bao giờ được sử dụng, function không nhẹ nhàng `_readThermometer` cũng không cần phải chạy.
- Bạn cần khởi tạo một biến thông qua *initializer* nhưng *initializer* lại cần truy cập đến `this`

### Final và const

Nếu bạn muốn khai báo một biến là hằng số, bạn có thể sự dụng từ khóa `final` hoặc `const`. Ý nghĩa của 2 từ khóa này là thông báo rằng:"Các biến này chỉ được gán giá trị một lần trong đời và không thể thay đổi được".

##### final

```
final name = "Bob";               // Dart tự suy ra kiểu của biến là String
final String nickName = "Bobby";

name = "Alice";                   // ERROR: Không thể thay đổi giá trị của một biến final
```

Một biến `final` nếu nằm ở top-level hoặc nằm trong class sẽ được khởi tạo ở lần đầu tiên biến đó được sử dụng. Và chúng ta cần lưu ý rằng: một instance variable có thể là `final`, nhưng không thể là `const`.

##### const

Thực chất, một biến `const` cũng là một biến `final`. Nhưng biến `const` có thêm một điều kiện là biến đó phải là *compile-time constants*. Để thỏa mãn điều kiện đó:
* Khi ta khai báo một biến `const`, ta phải gán ngay một giá trị cho nó thay vì có thể để trống và gán giá trị sau như `final`
* Giá trị của một biến `const` có thể là một số , một `String`, một biến `const` khác hoặc một biểu thức số học của các biến `const` khác.
```
const foo = 1000;                 // Dart tự suy ra kiểu của foo là int
const String bar = "A thousand"
const double = baz = 3.14 * foo;
```

Từ khóa `const` ngoài việc dùng để tạo ra các *constant variables* còn có thể tạo ra các *constant values*. Tức là các giá trị không thể thay đổi được thay vì các biến không thể thay đổi được.
```
var foo = const [1, 2, 3];          // Biến foo không phải là constant variables nhưng giá trị được gán cho biến foo là constant values.
final bar = const [1, 2, 3];        // Biến bar là final variables và giá trị được gán cho bar là constant values.
const baz = [1, 2, 3];              // Biến baz là constant variables và giá trị được gán cho baz không phải constant values.
```

Ở đây, ta cần phân biệt giữa *constant variables* và *constant values*. Với một biến là *constant variables*, ta có 2 trường hợp, `const` variable và `final` variable
```
final bar = [1, 2, 3];
const baz = [1, 2, 3];

bar[0] = 4;                         // Ta có thể thay đổi được giá trị của values
bar = [4, 5, 6];                    // ERROR: Ta không thể thay đổi được giá trị của biến bar vì bar là final variable
baz[0] = 4;                         // ERROR: Ta không thể thay đổi được giá trị của values vì biến baz là constant variable
baz = [4, 5, 6];                    // ERROR: Ta không thể thay đổi được giá trị của biến baz vì biến baz là constant variable
```

Với một biến có giá trị là *constant values*, ta có thể thay đổi giá trị của biến tùy thuộc vào biến đó có là *final variables* hay *constant variables* hay không
```
var foo = const [1, 2, 3];
final bar = const [1, 2, 3];
const baz = [1, 2, 3];  

foo[0] = 4;                         // ERROR: Ta không thể thay đổi giá trị của values vì đó là một constant values
foo = [4, 5, 6];                    // Ta có thể thay đổi được giá trị của foo vì foo chỉ là một biến bình thường
bar = [4, 5, 6];                    // ERROR: Ta không thể thay đổi được giá trị của biến bar vì bar là final variable
bar[0] = 4;                         // ERROR: Ta không thể thay đổi được giá trị của values của biến bar vì đó là một constant values
baz[0] = 4;                         // ERROR: Ta không thể thay đổi được giá trị của values vì biến baz là constant variable
baz = [4, 5, 6];                    // ERROR: Ta không thể thay đổi được giá trị của biến baz vì biến baz là constant variable           
```

**Note**: Một `final` object thì không thể thay đổi được, nhưng các trường bên trong (fields) thì có thể. Một `const` object thì không thể thay đổi được lẫn không thể thay đổi được các trường bên trong.
