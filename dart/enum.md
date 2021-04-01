# Enumerated types

Kiểu *enum* là một class đặc biệt được sinh ra nhằm mục đích biểu diễn các giá trị constant

Để khai báo kiểu enums, chúng ta sử dụng từ khóa `enum`
```
enum Color { red, green, blue }
```

Mỗi giá trị enum đại diện cho một giá trị constant và có một index tương ứng. Chúng ta có thể lấy index của một giá trị thông qua getter `index`:
```
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

Ngoài ra, chúng ta có thể lấy ra tất cả các giá trị của enums thông qua property `values`
```
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);
```

Vì enums đại diện cho các giá trị constant, chúng ta có thể sử dụng enums với câu lệnh `switch`. Trong trường hợp bạn không handle đầy đủ tất cả các giá trị của enums, sẽ có warning được compiler đưa ra
```
var aColor = Color.blue;

switch (aColor) {
  case Color.red:
    print('Red as roses!');
    break;
  case Color.green:
    print('Green as grass!');
    break;
  default: // Without this, you see a WARNING.
    print(aColor); // 'Color.blue'
}
```

Những giới hạn của enum là
- Bạn không thể kế thừa, mix in hoặc implement một enum
- Bạn không thể khởi tạo một enum

Chi tiết hơn, bạn có thể tham khảo [Dart language specification](https://dart.dev/guides/language/spec)
