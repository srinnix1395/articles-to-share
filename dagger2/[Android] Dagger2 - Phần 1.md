[Android] Dagger 2 - Phần 1: Các khái niệm cơ bản

Mình biết đến Dagger(chính xác là Dagger 2) khi còn đi thực tập ở một công ty. Vì chỉ là một android intern nên việc chính là fix một vài cái issue phụ trong một ngày làm việc 4h ở đó. Bởi vậy, sau khi hoàn thành một cách khá nhanh chóng các issue đó, mình dành thời gian để đọc thêm về công nghệ ([AndroidWeekly](https://androidweekly.net/) là một nguồn mình recommend cho các bạn). Và mình bắt đầu biết về các khái niệm: *Dependency inversion*, *Inversion of control*, *Dependency injection* (DI) và một library thường được sử dụng để implement DI trong Android: [Dagger2](https://google.github.io/dagger/). Thực sự trong một thời gian sau đó, mình có và cố đọc thêm nhiều article, đọc thêm code example về chủ đề này. Tuy nhiên, do không thực sự cần, không thực sự hiểu sử dụng DI có tác dụng gì đối với project sẽ làm (và công nhận đây cũng là một chủ đề khó nhằn với một người chưa có nhiều kinh nghiệm về lập trình), mình đơn giản chỉ copy code và thay giá trị tương ứng để chạy được ứng dụng - vài tháng sau đó trong đồ án của mình. Tuy nhiên, trẻ con rồi cũng phải đến lúc cắp sách đến trường, để hiểu câu nói ngày xưa mình bắt chước bố mẹ nghĩa là gì. Bởi vậy, mình muốn hệ thống lại những kiến thức của mình về DI mà mình đã đọc và đã nghiệm ra trong quá trình bắt chước, hy vọng series có thể trở thành bài học vỡ lòng cho những bạn bắt đầu làm quen với DI trong Android.

### Các bài học để lên lớp

1. [Android] Dagger 2 - Phần 1: Các khái niệm cơ bản
2. [Android] Dagger 2 - Phần 2: Dependency component và sub-component
3. ...

### Kiến thức đầu vào

* Không bị ở lại lớp vì môn *Lập trình hướng đối tượng*, *Lập trình Android cơ bản*
* Đã đi học thêm về *Kotlin* (Vì các ví dụ mình sẽ sử dụng Kotlin)

Ảnh trên unsplash

### Lý thuyết

> Dagger 2 is a library which helps the developer to implement a pattern of Dependency Injection (one specific form of Inversion of control).

Trước hết, chúng ta đi qua một số khái niệm để hiểu rõ hơn vấn đề và background của vấn đề:

##### Dependency

Dependency là từ dùng để mô tả việc những module cấp cao gọi một module cấp thấp. Ta có ví dụ sau: khi chúng ta đi học một môn nào đấy, chúng ta cần có một quyển sách giáo khoa. Từ đó, ta có thể mô tả vấn đề trong thực tế đó thành các đối tượng: *Student* và *MathBook*.

```
class MathBook {

    fun getSubjectName(): String {
        return "Math"
    }
}
```

```
class Student {

    var mathBook: MathBook? = null

    fun learn() {
        mathBook?.let {
            println("Learning ${it.getSubjectName()}")
        }
    }
}
```

Và ở hàm `main`, ta sẽ gọi như sau:

```
fun main(args: Array<String>) {
    val student = Student()
    student.mathBook = MathBook()

    student.learn()
}
```

Ở đây, học sinh không thể học mà không có sách giáo khoa. Bởi vậy, ta nói *Student* phụ thuộc vào *MathBook* và *MathBook* được gọi là dependency của *Student*.

##### Dependency Inversion

Đây là nguyên lý cuối cùng trong những nguyên lý thiết kế trong lập trình hướng đối tượng **SOLID**. Nội dung của nguyên lý này như sau:

* Các module (có thể hiểu ở phạm vi nhỏ hơn là class) cấp cao không nên phụ thuộc vào các module cấp thấp hơn mà nên phụ thuộc vào abstractions (một interface chẳng hạn)
* Interface (abstraction) không nên phụ thuộc vào chi tiết, mà ngược lại. (Các class giao tiếp với nhau thông qua interface, không phải thông qua implementation.)

Lần đầu tiên đọc nội dung này, mình thấy đúng là abstract vãi nồi~~ Có lẽ chúng ta nên "phụ thuộc" vào chi tiết trước (lấy ví dụ), rồi mới nên "phụ thuộc" vào trừu tượng (đọc lại nguyên lý để ngẫm tiếp) :D. Mình sẽ tiếp tục với ví dụ phía trên:

```
class Student {

    var mathBook: MathBook? = null

    fun learn() {
        mathBook?.let {
            println("Learning ${it.getSubjectName()}")
        }
    }
}
```

Ta thấy nếu *Student* muốn học thêm một môn mới, ta sẽ phải khai báo thêm một thuộc tính là một quyển sách khác (*EnglishBook* chẳng hạn) và phải thêm một phương thức để học môn học đấy (`learnEnglish()`). Ngẫm một chút, nếu càng ngày *Student* càng học lên cao, số môn học cần phải học sẽ càng nhiều, *Student* sẽ càng ngày càng phình to ra~~ Ở đây, *Student* đang là module cấp cao, phụ thuộc vào một module cấp thấp hơn là *MathBook*(tức là đang phụ thuộc vào chi tiết, thay vì trừu tượng). Cách code này dẫn đến một vấn đề nữa là sửa đổi ở module cấp thấp sẽ kéo theo một loạt các sửa đổi ở module cấp cao, việc maintain code trở nên phức tạp hơn. Để giải quyết vấn đề trên, chúng ta đến với khái niệm tiếp theo: *Inversion of control*

##### Inversion of control

IoC là một design pattern để implement nguyên lý *Dependency inversion* ở trên. IoC tuân thủ các nội dung của *Dependency inversion* thông qua việc nó không quan tâm đến việc khởi tạo các module cấp thấp như thế nào, detail implementation của các module cấp thấp ra sao mà chỉ quan tâm đến những gì mà các module này cung cấp một cách abstraction (sử dụng interface).

Ngoài ra, IoC sử dụng 1 container để chứa các detail implementation của các abstractions


Ví dụ ở phía trên nên được sửa lại theo IoC như sau:

* Interface *TextBook* sẽ là cầu nối giữa module cấp cao (*Student*) và các module cấp thấp (*MathBook*, *EnglishBook*).
```
interface TextBook {

    fun getSubjectName(): String
}
```
```
class MathBook : TextBook {

    override fun getSubjectName(): String {
        return "Math"
    }
}
```
```
class EnglishBook : TextBook {

    override fun getSubjectName(): String {
        return "English"
    }
}
```

* Class *Student* thay vì bao gồm các thuộc tính cụ thể thì sẽ bao gồm một thuộc tính trừu tượng.
```
class Student {

    var textBook: TextBook? = null

    fun learn() {
        textBook?.let {
            println("Learning ${it.getSubjectName()}")
        }
    }
}
```

* Và bây giờ, *Student* có học bao nhiêu môn đi chăng nữa hoặc các môn học được "cải cách" thế nào đi chăng nữa, *Student* cũng sẽ không bị ảnh hưởng vì không còn quan tâm đến implementation detail của *MathBook* hay *EnglishBook* nữa. Học "lại" nào:
```
fun main(args: Array<String>) {
    val student = Student()

    val mathBook = MathBook()
    student.textBook = mathBook
    student.learn()

    val englishBook = EnglishBook()
    student.textBook = englishBook
    student.learn()
}
```

Có nhiều cách để implement IoC như *Service Locator*, *Event* hay *Dependency injection*... Mỗi cách có ưu và nhược điểm riêng mà tùy trường hợp áp dụng sao cho phù hợp. Tuy nhiên, trong khuôn khổ bài viết này
##### Dependency injection

Trong trường hợp này, nếu ta khởi tạo 2 thuộc tính `mathBook` và `englishBook` bên trong constructor của lớp *Student* như trong đoạn code ở trên, ta gọi kiểu này là hard dependency. Tuy nhiên, hard dependency có những hạn chế sau:

* Làm giảm tính tái sử dụng vì nếu càng phụ thuộc nhiều, ta càng khó tái sử dụng lại một lớp nào đó.

* Khó viết unit test hơn: Khi viết unit test, ta cần cô lập module cần test với phần còn lại của ứng dụng bằng cách mock các thành phần bên trong của đối tượng. Khi hard dependency xảy ra, ta không thể cô lập *MathBook* hoặc *EnglishBook* với *Student* để chạy được unit test.
