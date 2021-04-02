# Asynchrony support

Library trong *Dart* có rất nhiều những function trả về đối tượng `Future` và `Stream`. Những function này là những function bất đồng bộ: chúng trả về kết quả sau khi một công việc tốn nhiều thời gian hoàn thành (VD: I/O), giúp chúng ta không phải đợi công việc đó xong xuôi rồi mới được chạy tiếp.

Ngoài ra, *Dart* có các từ khóa `async` và `await` hỗ trợ việc lập trình bất đồng bộ. Các từ khóa này làm cho code của chúng ta trông tương tự như code đồng bộ.

### Futures

Khi bạn muốn lấy kết quả của một `Future`, có 2 cách:
- Sử dụng `async` và `await`
- Sử dụng các API của `Future`, được mô tả ở [library tour](https://dart.dev/guides/libraries/library-tour#future)

Code sử dụng `async` và `await` là bất đồng bộ. Tuy nhiên, chúng lại trông giống code đồng bộ hơn. VD:
```
Future checkVersion() async {
  var version = await lookUpVersion();
  // Do something with version
}
```

Trong trường hợp trên, `lookUpVersion()` là một function thực hiện một tác vụ time-consuming. Bởi vậy, chúng ta sử dụng `await` để đợi function thực hiện xong và trả về `version` mới tiếp tục thực hiện các câu lệnh bên dưới. Chúng ta chỉ có thể sử dụng `await` bên trong một `async` function.

Chúng ta cũng có thể sử dụng `try`, `catch` và `finally` để xử lý lỗi khi sử dụng `await`:
```
try {
  version = await lookUpVersion();
} catch (e) {
  // React to inability to look up the version
}
```

Bên trong một `async` function, chúng ta có thể sử dụng `await` nhiều lần:
```
var entrypoint = await findEntrypoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
```

Với `await expression`, giá trị của `expression` thường có kiểu là một `Future`. Nếu không, giá trị đó sẽ được tự động wrap trong một `Future`. Object `Future` là một dạng promise như trong JavaScrip, promise sẽ trả về một kết quả ở tương lai. Giá trị của `await expression` là giá trị tương lai mà `expression` trả về đó. Mỗi lần gặp `await`, quá trình chạy sẽ dừng lại và đợi cho đến khi giá trị tương lai kia khả dụng.

### Khai báo async function

`async` function là một function bình thường với phần body được đánh dấu với từ khóa `async`. Việc thêm `async` vào function sẽ làm cho function đó trả về một `Future`.VD: function sau trả về một `String`
```
String lookUpVersion() => '1.0.0';
```

Nếu bạn thêm `async` vào function, kiểu dữ liệu trả về bây giờ là `Future<String>`
```
Future<String> lookUpVersion() async => '1.0.0';
```

**Note**: Bạn không cần trả về một `Future`, *Dart* sẽ tự động tạo `Future` object nếu cần thiết. Với các function không trả về giá trị gì, kiếu dữ liệu trả về là `Future<void>`

### Streams

Khác với `Future` chỉ trả về một giá trị, `Stream` trả về nhiều giá trị, để lấy các giá trị đó, có 2 cách:
- Sử dụng `async` và một vòng lặp for bất đồng bộ - `await for`
- Sử dụng Stream API, được mô tả ở [library tour](https://dart.dev/guides/libraries/library-tour#stream)

**Note**: Trước khi sử dụng `await for`, hãy chắc chắn rằng code của bạn rõ ràng, sạch sẽ và bạn thực sự muốn đợi tất cả kết quả của stream. Ví dụ, không bao giờ nên sử dụng `await for` với các event listener liên quan đến UI bởi vì UI frameworks sẽ emit một stream không bao giờ đóng.

Một vòng lặp for bất đồng bộ có dạng như sau:
```
await for (varOrType identifier in expression) {
  // Executes each time the stream emits a value.
}
```

Giá trị của `expression` phải có kiểu `Stream`. Thứ tự thực hiện đoạn code trên như sau:
1. Đợi đến khi một giá trị được stream emit
2. Chạy đoạn lệnh bên trong vòng lặp for với `identifier` là giá trị được emit
3. Lặp lại 1 và 2 cho đến khi stream đóng.

Để dừng việc lắng nghe stream, bạn có thể sử dụng `break` hoặc `return`.

Tương tự như `await`, `await for` phải được sử dụng bên trong một `async` function
```
Future main() async {
  // ...
  await for (var request in requestServer) {
    handleRequest(request);
  }
  // ...
}
```
