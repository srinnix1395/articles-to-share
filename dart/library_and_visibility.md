# Library and visibility

Từ khóa `import` và `library` trong *Dart* sẽ giúp chúng ta các tạo ra những module riêng biệt giúp cho việc chia sẻ codebase dễ dàng hơn. Ngoài mục đích cung cấp các API, library còn là một đơn vị của sự riêng tư khi chúng ta có thể show hoặc hide một API bằng cách sử dụng `_`. Những phần tử có tên bắt đầu bằng `_` sẽ chỉ có thể sử dụng bên trong library khai báo nó. Trong *Dart*, mỗi ứng dụng cũng được coi là một library. Các libary có thể được distribute thông qua package manager.

**Note**: Nếu bạn tò mò vì sao *Dart* sử dụng `_` thay vì các access modifier như `public` hay `private`, hãy xem [SDK issue 33383](https://github.com/dart-lang/sdk/issues/33383)

### Sử dụng library

Sử dụng từ khóa `import` để sử dụng một library khác
```
import 'dart:html';
```

Giá trị phía sau từ khoán `import` là URI của library muốn sử dụng. Với những built-in library, URI sẽ có dạng `dart:`. Với những thư viện khác, URI là đường dẫn đến library hoặc `package:` khi library được cung cấp thông qua package manager.
```
import 'package:test/test.dart';
```

##### Library prefix

Nếu import hai library mà một phần tử trong library này trùng tên với 1 phần tử trong library kia, chúng ta gán cho library một prefix để phân biệt
```
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

// Uses Element from lib1.
Element element1 = Element();

// Uses Element from lib2.
lib2.Element element2 = lib2.Element();
```

##### Import một phần của library

Nếu bạn muốn import chỉ một phần của library, bạn có thể sử dụng từ khóa `show` hoặc `hide`
```
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```

##### Lazy loading một library

Chỉ áp dụng cho *dart2js* nên bạn có thể tìm hiểu thêm [ở đây](https://dart.dev/guides/language/language-tour#using-libraries)
