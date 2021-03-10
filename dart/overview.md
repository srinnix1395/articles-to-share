# Tổng quan

Bài viết đầu tiên này sẽ cho bạn một cái nhìn tổng quan về *Dart*, một ngôn ngữ được phát triển bởi *Google* và là nền tảng cho *Flutter*.

### Overview

Trên trang chủ, *Dart* được giới thiệu như sau:

> *Dart* là một ngôn ngữ *tối ưu hóa phía người dùng* ([client-optimized language](https://stackoverflow.com/a/59517340)) để phát triển các ứng dụng trên nhiều nền tảng. Mục tiêu của Dart là đưa ra một ngôn ngữ lập trình hiệu quả nhất cho việc phát triển đa nền tảng và đi kèm với cách chạy chương trình linh hoạt đối với từng  loại nền tảng.

Ở đây, cần nhấn mạnh yếu tố *client-optimized language* khi *Dart* tập trung vào các ứng dụng phía client:
* Hỗ trợ *hot reload* giúp việc phát triển nhanh hơn
* Hỗ trợ việc thực thi trên nhiều platform: web, mobile và desktop.

Note: Để *chơi* với *Dart* mà không cần cài đặt, bạn có thể  sử dụng [DartPad](https://dartpad.dev/?null_safety=true)

### Dart: Ngôn ngữ

### Dart: Thư viện

*Dart* có sẵn rất nhiều thư viện core, cung cấp cho bạn những công cụ cần thiết cho những vấn đề thường gặp:

* *dart:core*: Built-in types, collections và các core functions cho mọi chương trình của *Dart*
*

Để tìm hiểu kỹ hơn các thư viện đó, bạn có thể check [Library tour](https://dart.dev/guides/libraries/library-tour)

### Dart: Đa nền tảng

Compiler của *Dart* giúp code của bạn có thể chạy trên nhiều platform khác nhau:
* Native platform (Mobile and desktop): *Dart* sử dụng cả máy ảo *DVM (Dart Virtual Machine)* với cơ chế *JIT (just-in-time)* trong quá trình phát triển sản phầm và trình biên dịch *AOT (ahead-of-time)* khi ứng dụng ready-to-be-deployed, để chuyển từ code *Dart* sang mã máy.
* Web platform: *Dart* sử dụng *dartdevc* trong quá trình phát triển sản phầm và *dart2js* khi ứng dụng ready-to-be-deployed, để chuyển từ code *Dart* thành code *JavaScript*

![](https://dart.dev/assets/Dart-platforms-29d94ccaaea4bdd0b2bcf6e37084bae890eb8bae482baf3b0894e0b99a17b8d7.svg)
