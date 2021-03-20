# Control flow statements

Các câu lệnh điều khiển trong *Dart* bao gồm:
* `if` và `else`
* Vòng lặp `for`
* Vòng lặp `while` và `do-while`
* `break` và `continue`
* `switch` và `case`
* `assert`
* `try-catch` và `throw`

### `if` và `else`

Với câu lệnh `if`, điều kiện bất buộc phải có kiểu `bool`
```
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

### Vòng lặp `for`

*Closure* bên trong vòng lặp `for` sẽ capture được giá trị
```
var callbacks = [];
for (var i = 0; i < 2; i++) {
  callbacks.add(() => print(i));
}
callbacks.forEach((c) => c());
```

Kết quả được in ra là 0 và 1, thay vì 2 và 2 với *JavaScript*

Với đối tượng có kiểu *Iterable*, bạn có thể sử dụng function *forEach* hoặc `for-in`
```
candidates.forEach((candidate) => candidate.interview());

var collection = [1, 2, 3];
for (var x in collection) {
  print(x); // 1 2 3
}
```

### While và do-while

Tương tự như *Java*

### Break và continue

Tương tự như *Java*

### Switch và case

Câu lệnh `switch` trong *Dart* chấp nhận dữ liệu kiểu `int`, `String` hoặc một *compile-time constant* và sử dụng toáng tử `==` để so sánh. Mọi mệnh đề `case` mà không trống bắt buộc phải kết thúc bằng: `break`, `continue`, `throw` hay `return`, nếu không sẽ có lỗi
```
var command = 'OPEN';
switch (command) {
  case 'OPEN':
    executeOpen();
    // ERROR: Missing break

  case 'CLOSED':
    executeClosed();
    break;
}
```

Tuy nhiên, với mệnh đều `case` mà rỗng, thì case tiếp theo sẽ được thực hiện
```
var command = 'CLOSED';
switch (command) {
  case 'CLOSED': // Empty case falls through.
  case 'NOW_CLOSED':
    // Runs for both CLOSED and NOW_CLOSED.
    executeNowClosed();
    break;
}
```

Ngoài ra, khi sử dụng `continue`, case tiếp theo sẽ được thực hiện
```
var command = 'CLOSED';
switch (command) {
  case 'CLOSED':
    executeClosed();
    continue nowClosed;
  // Continues executing at the nowClosed label.

  nowClosed:
  case 'NOW_CLOSED':
    // Runs for both CLOSED and NOW_CLOSED.
    executeNowClosed();
    break;
}
```

### Assert

Để kiểm tra những điều kiện trong quá trình debug project, bạn có thể dùng `assert`. *Dart* chỉ enable việc assert trong quá trình debug và sẽ bỏ qua với khi project được release
```
// Make sure the variable has a non-null value.
assert(text != null);
```
