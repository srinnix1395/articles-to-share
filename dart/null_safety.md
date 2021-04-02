# Null safety

Với null safety, mọi kiểu dữ liệu mặc định trong *Dart* đều là kiểu non-nullable. VD: mọi biến trong đoạn code sau đều có kiểu non-nullable
```
var i = 42; // Inferred to be an int.
String name = getFileName();
final b = Foo();
```

Nếu muốn một biến có thể có giá trị `null`, chúng ta thêm dấu `?` vào sau kiểu dữ liệu của biến đó
```
int? aNullableInt = null;
```

### Null safety principles

Null safety trong *Dart* hoặc động dựa trên 3 design principles chính:
- *Non-nullable by default*: Trừ khi bạn chỉ rõ cho *Dart* một biến có thể `null` bằng cách thêm `?`, kiểu dữ liệu mặc định luôn là non-nullable. Sự lựa chọn này được đưa ra sau khi tham khảo các API
- *Incrementally adaoptable* (Thích nghi dần đần): Khi migrate từ các version *Dart* không hỗ trợ null safety, chính bạn là người lựa chọn *phần nào* và khi nào hỗ trợ null safety. Bạn hoàn toàn có thể migrate một cách từ từ, kết hợp cả null-safe code và non-null-safe code trong cùng một project. *Dart* sẽ cung cấp tool cho quá trình migrate này.
- *Fully sound*:

### Enabling null safety

Null safety được hỗ trợ từ Dart 2.12 và Flutter 2

##### Migrate một package/app

Tham khảo tại [migration guide](https://dart.dev/null-safety/migration-guide)
