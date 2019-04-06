[Android] Dagger 2 - Phần 1: Các khái niệm cơ bản

Mình biết đến Dagger(chính xác là Dagger 2) khi còn đi thực tập ở một công ty. Vì chỉ là một android intern nên việc chính là fix một vài cái issue phụ trong một ngày làm việc 4h ở đó. Bởi vậy, sau khi hoàn thành một cách khá nhanh chóng các issue đó, mình dành thời gian để đọc thêm về công nghệ ([AndroidWeekly](https://androidweekly.net/) là một nguồn mình recommend cho các bạn). Và mình bắt đầu biết về các khái niệm: *Dependency inversion*, *Inversion of control*, *Dependency injection* (DI) và một library thường được sử dụng để implement DI trong Android: [Dagger2](https://google.github.io/dagger/). Thực sự trong một thời gian sau đó, mình có và cố đọc thêm nhiều article, đọc thêm code example về chủ đề này. Tuy nhiên, do không thực sự cần, không thực sự hiểu sử dụng DI có tác dụng gì đối với project sẽ làm (và công nhận đây cũng là một chủ đề khó nhằn với một người chưa có nhiều kinh nghiệm về lập trình), mình đơn giản chỉ copy code và thay giá trị tương ứng để chạy được ứng dụng - vài tháng sau đó trong đồ án của mình. Tuy nhiên, trẻ con rồi cũng phải đến lúc cắp sách đến trường, để hiểu câu nói ngày xưa mình bắt chước bố mẹ nghĩa là gì. Bởi vậy, mình muốn hệ thống lại những kiến thức của mình về DI mà mình đã đọc và đã nghiệm ra trong quá trình bắt chước, hy vọng series có thể trở thành bài học vỡ lòng cho những bạn bắt đầu làm quen với DI trong Android.

### Các bài học để lên lớp

1. [Android] Dagger 2 - Phần 1: Các khái niệm cơ bản
2. [Android] Dagger 2 - Phần 2: Dependency component và sub-component
3. [Android] Dagger 2 - Phần 3: Custom scope trong dagger 2

### Kiến thức đầu vào

* Không bị ở lại lớp vì môn *Lập trình hướng đối tượng*, *Lập trình Android cơ bản*
* Đã đi học thêm về *Kotlin* (Vì các ví dụ mình sẽ sử dụng Kotlin)

Ảnh trên unsplash

### Lý thuyết

> Dagger 2 is a library which helps the developer to implement a pattern of Dependency Injection (one specific form of Inversion of control).

Trước hết, chúng ta đi qua một số khái niệm để hiểu rõ hơn background của vấn đề:

##### Dependency

Dependency là từ dùng để mô tả việc module cấp cao gọi một module cấp thấp. Ta có ví dụ sau: khi chúng ta học một môn nào đấy, chúng ta cần có một quyển sách giáo khoa. Từ đó, ta có thể mô tả vấn đề trong thực tế đó thành các đối tượng: *Student* và *MathBook*.

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

Ở đây, học sinh không thể học mà không có sách giáo khoa. Bởi vậy, ta nói *Student* phụ thuộc vào *MathBook* và *MathBook* được gọi là dependency của *Student*. Vậy có điều gì cần lưu ý khi khai báo và khởi tạo các dependency? Xin giới thiệu: *Dependency inversion*!

##### Dependency Inversion

Trong những nguyên lý thiết kế trong lập trình hướng đối tượng **SOLID**, *Dependency inversion* là nguyên lý cuối cùng. Nội dung của nguyên lý này như sau:

* Các module (có thể hiểu là class) cấp cao không nên phụ thuộc vào các module cấp thấp hơn. Cả 2 nên phụ thuộc vào abstractions (một interface chẳng hạn)
* Interface (abstraction) không nên phụ thuộc vào chi tiết, mà ngược lại. (Các class giao tiếp với nhau thông qua interface, không phải thông qua implementation.)

Lần đầu tiên đọc nội dung này, mình thấy thật sự abstract vãi nồi~~ Có lẽ chúng ta nên "phụ thuộc" vào chi tiết trước (lấy ví dụ), rồi mới nên "phụ thuộc" vào trừu tượng sau (đọc lại nguyên lý để ngẫm tiếp) :D. Mình sẽ tiếp tục với ví dụ phía trên:

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

Ta thấy nếu *Student* muốn học thêm một môn mới, ta sẽ phải khai báo thêm một thuộc tính là một quyển sách khác (*EnglishBook* chẳng hạn) và phải thêm một phương thức để học môn học đấy (`learnEnglish()`). Ở đây sẽ xảy ra một số vấn đề:
* Nếu càng ngày *Student* càng học lên cao, số môn học cần phải học sẽ càng nhiều, class *Student* sẽ càng ngày càng phình to ra~~
* *Student* đang là module cấp cao, phụ thuộc vào một module cấp thấp hơn là *MathBook*(tức là đang phụ thuộc vào chi tiết, thay vì trừu tượng). Vì vậy, việc sửa đổi module cấp thấp sẽ kéo theo một loạt các sửa đổi ở module cấp cao, điều đó làm việc maintain code trở nên phức tạp hơn.

Để giải quyết vấn đề trên, chúng ta đến với khái niệm tiếp theo: *Inversion of control*

##### Inversion of control

IoC là một design pattern để implement nguyên lý *Dependency inversion* ở trên. IoC tuân thủ các nội dung của *Dependency inversion* thông qua việc nó không quan tâm đến việc khởi tạo các module cấp thấp như thế nào, detail implementation của các module cấp thấp ra sao mà chỉ quan tâm đến những gì mà các module này cung cấp một cách abstraction (sử dụng interface).

IoC sử dụng 1 container để chứa các detail implementation của các abstractions. Khi nào một module cấp cao cần dùng một module cấp thấp, module cấp cao cần tìm instance của module cấp thấp trong container và inject nó vào module cấp cao. todo: confirm

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

Có nhiều cách để implement IoC như *Service Locator*, *Event* hay *Dependency injection*... Mỗi cách có ưu và nhược điểm riêng mà tùy trường hợp áp dụng sao cho phù hợp. Tuy nhiên, trong khuôn khổ series này, chúng ta sẽ tìm hiểu về *Dependency injection* - một specific form của IoC.

##### Dependency injection

Vậy *Dependency inject* là gì? DI là quá trình trong đó dependency của một module sẽ được cung cấp (inject) từ bên ngoài thay vì được khởi tạo bên trong của module. Ta có ví dụ sau về việc không sử dụng DI và có sử dụng DI:

```
class Student {

    private var textBook: TextBook? = null

    constructor() {
        textBook = MathBook()
    }

    fun learn() {
        textBook?.let {
            println("Learning ${it.getSubjectName()}")
        }
    }
}
```

Ta gọi trường hợp này là hard dependency khi `textBook` được khởi tạo cứng trong constructor chứ không phải được cung cấp từ bên ngoài. Với cách này, ta có những nhược điểm sau:

* Tính tái sử dụng của module sẽ bị giảm đi: vì mỗi khi khởi tạo module cấp cao sẽ bắt buộc khởi tạo các module cấp thấp
* Không thể unit test: khi viết unit test, ta cần cô lập module với các phần còn lại của app bằng cách mock các module cấp thấp. Tuy nhiên, nếu ta hard dependency như trường hợp ở trên, ta không thể mock `textBook` để test được module `Student`.

Ta sẽ sử dụng DI để giải quyết vấn đề hard dependency ở trên:

```
class Student {

    private var textBook: TextBook? = null

    constructor(textBook: TextBook?) {
        this.textBook = textBook
    }

    fun learn() {
        textBook?.let {
            println("Learning ${it.getSubjectName()}")
        }
    }
}
```

Và ở main:

```
fun main(args: Array<String>) {

    val textBook = MathBook()
    val student = Student(textBook)

    student.learn()
}
```

Ở đây, chúng ta đã sử dụng DI một cách manual bằng cách khởi tạo `textBook` ở bên ngoài và inject nó vào module sử dụng là `student`. Tuy nhiên, đây chỉ là một ví dụ bé xíu, với 1 dependency được khởi tạo và inject vào, so với những vấn đề ở một project lớn hơn mà chúng ta sẽ gặp phải: khởi tạo và quản lý các dependency tùy theo scope của mỗi dependency. And Dagger 2 comes to the rescue!!!

##### Dagger 2

Dagger được Square tạo ra để implement DI trong Android. Tuy nhiên, một nhược điểm của Dagger 1 là Dagger 1 sử dụng reflection (reflection thì chậm) và việc tạo ra dependency được thực hiện lúc run-time nên có thể làm cho app bị crash. Khắc phục những nhược điểm từ Dagger 1, Dagger 2 không sử dụng reflection nữa mà sử dụng *annotation processor* (a code generator using annotation) để "viết" code cho chúng ta. Những đoạn code được sinh ra khi compile sẽ không làm app crash bất thình lình nữa. Cùng với đó, các đoạn code này hoàn toàn không cao siêu mà rất dễ đọc bởi chúng bắt chước những đoạn code như của một lập trình viên thật sự viết ra nên việc debug là hoàn toàn có thể.

##### Các annotation trong Dagger 2

Dagger 2 dựa vào các thông tin có được từ các annotation để "viết" ra code khi compile. Bởi vậy, ta sẽ tìm hiểu các annotation cơ bản trong Dagger 2:

* *@Inject* - đánh dấu "đâu" là nơi "cần một dependency".
* *@Module* - đánh dấu một class, nơi "cung cấp các dependency"
* *@Provides* - đánh dấu các method nằm bên trong *@Module* và thể hiện "cách khởi tạo các dependency".
* *@Component* - đánh dấu một interface (dependency graph) là cầu nối giữa cung - *@Module* và cầu - *@Inject*.
* *@Scope* - thể hiện vòng đời (scope) của các dependency, từ đó giúp ta tạo ra các global singleton hoặc local singleton.
* *@Qualifier* - annotation này giúp phân biệt các dependency có cùng kiểu dữ liệu với nhau.

Khá nhiều thứ phải nhớ đấy nhỉ~~. Nhưng mà đừng quá lo lắng, chúng ta sẽ thử implement ngay Dagger 2 để hiểu hơn từng annotation một nhớ.

### Thực hành
