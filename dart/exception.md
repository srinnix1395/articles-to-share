# Exception

Exception là những lỗi xảy ra mà chúng ta không mong muốn. Nếu exception không được bắt, chương trình sẽ crash.

Ngược lại với *Java*, exception trong *Dart* đều là *unchecked exception* tức là các exception không được kiểm tra ở compile-time. Ta sẽ không phải chỉ rõ kiểu exception khi khai báo function hoặc không phải bắt buộc catch các exception

*Dart* cung cấp kiểu dữ liệu `Exception`, `Error` và các lớp con được định nghĩa trước các exception thường gặp. Và bạn cũng có thể tự định nghĩa các exception bạn muốn. Ngoài ra, *Dart* cũng hỗ trợ throw bất kỳ object non-null nào.

### Throw

```
//throw an exception
throw FormatException('Expected at least 1 section');

//throw a non-object
throw 'Out of llamas!';
```

Bởi vì throw một exception là một expression (biểu thức) nên bạn có thể sử dụng throw exception với arrow syntax (`=>`) hoặc bất kỳ chỗ nào cho phép sử dụng expression
```
void distanceTo(Point other) => throw UnimplementedError();
```

### Catch

Cú pháp của catch trong *Dart* là
```
try {
  // code can throw an exception
} on {Exception} catch (e) {
    print('Unknown exception: $e');
}
```

Nếu bạn không cần exception object, `catch (e)` có thể được lược bỏ
```
try {
  // code can throw an exception
} on {Exception} {
    print('Something wrong happened');
}
```

Hoặc nếu bạn xác định được loại exception muốn catch, lược bỏ `on {Exception}`
```
try {
  // code can throw an exception
} catch (e) {
    print('Something wrong that I don't know happened');
}
```

Ngoài ra, catch còn một function nữa 2 tham số với tham số thứ 2 là `StackTrace` object
```
try {
  // ···
} on Exception catch (e) {
  print('Exception details:\n $e');
} catch (e, s) {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```

Trong trường hợp bạn đã xử lý exception nhưng vẫn muốn aware cho một hàm gọi trước đó, bạn có thể sử dụng `rethrow()`
```
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++); // Runtime error
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // Allow callers to see the exception.
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

### Finally

Tương tự với *Java*, ta có `finally` để dùng trong những trường hợp ta muốn chạy những đoạn code dọn dẹp kể cả trong trường hợp exception được catch
```
try {
  breedMoreLlamas();
} finally {
  // Always clean up, even if an exception is thrown.
  cleanLlamaStalls();
}
```

Với trường hợp có exception được catch, `finally` sẽ được chạy sau khi exception được xử lý
```
try {
  breedMoreLlamas();
} catch (e) {
  print('Error: $e'); // Handle the exception first.
} finally {
  cleanLlamaStalls(); // Then clean up.
}
```
