[Android] Dagger 2 - Phần III: The time of our dependencies

Bài viết là phần thứ III của series bài học vỡ lòng về *Dagger 2*. Nếu bạn chưa đọc các phần trước, bạn có thể ghi danh vào lớp học [tại đây](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)

# Các bài học để lên lớp

1. [[Android] Dagger 2 - Phần I: Basic principles](https://kipalog.com/posts/Android--Dagger-2---Phan-I--Basic-principles)
2. [[Android] Dagger 2 - Phần II: Into the Dagger 2](https://kipalog.com/posts/Android--Dagger-2---Phan-II--Into-the-Dagger-2)
3. [Android] Dagger 2 - Phần III: The time of our dependencies

# Trong bài học trước...

Chúng ta đã tìm hiểu cách "mô hình hóa" các mối quan hệ giữa class và các dependency thông qua *dependency graph*. Tiếp đó, chúng ta xây dựng một ứng dụng đơn giản và từng bước áp dụng những annotation cơ bản trong *Dagger 2* để khởi tạo các dependency.

# Đi vào bài học hôm nay...
Chương trình nhỏ ở phần II sẽ tiếp tục scale up lên với thêm nhiều màn hình và chức năng. Khi đó, chúng ta cũng sẽ đối mặt với vấn đề quản lý vòng đời của các dependency.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/lb2kry1ck4_john-towner-3Kv48NS4WUU-unsplash.jpg">
  Photo by <a href="https://unsplash.com/@heytowner?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">JOHN TOWNER</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

</p>
