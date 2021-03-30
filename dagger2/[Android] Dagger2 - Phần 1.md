[Android] Dagger 2 - Phần I: Các khái niệm cơ bản

Mình biết đến *Dagger* (chính xác là *Dagger 2*) khi còn đi thực tập ở một công ty. Vì chỉ là một android intern làm việc 4 tiếng một ngày nên công việc chính của mình chỉ là fix một vài cái issue bé bé hay implement vài tính năng dùng đầu thì ít mà dùng tay thì nhiều. Bởi vậy, sau khi hoàn thành một cách khá nhanh chóng các công việc đó, mình dành thời gian để đọc thêm về công nghệ ([AndroidWeekly](https://androidweekly.net/) là một nguồn mình recommend cho các bạn). Và mình bắt đầu biết về các khái niệm: *Inversion of control*, *Dependency inversion*, *Dependency injection* (DI) và một library thường được sử dụng để implement *DI* trong Android: [Dagger2](https://google.github.io/dagger/). Thực sự trong một thời gian sau đó, mình có và cố đọc thêm nhiều article, đọc thêm code example về chủ đề này. Tuy nhiên, do không thực sự cần, không thực sự hiểu sử dụng *DI* có tác dụng gì đối với project sẽ làm (và công nhận đây cũng là một chủ đề khó nhằn với một người chưa có nhiều kinh nghiệm về lập trình), mình đơn giản chỉ copy & paste và thay đổi giá trị tương ứng để chạy được ứng dụng. Tuy nhiên, trẻ con rồi cũng phải đến lúc cắp sách đến trường, để hiểu câu nói ngày xưa mình bắt chước bố mẹ nghĩa là gì. Bởi vậy, mình muốn hệ thống lại những kiến thức của mình về *DI* mà mình đã đọc và đã nghiệm ra trong quá trình bắt chước, hy vọng series có thể trở thành bài học vỡ lòng cho những bạn bắt đầu làm quen với *DI* trong Android.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/nol8cqcf0x_riccardo-mion-IutqINJUAts-unsplash.jpg">
  An awaiting adventure...<br>Photo by <a href="https://unsplash.com/@riccardomion?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Riccardo Mion</a> on <a href="/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</p>

# Các bài học để lên lớp

1. [Android] Dagger 2 - Phần I: Các khái niệm cơ bản (Bạn đang ở đây)
2. [[Android] Dagger 2 - Phần II: Into the Dagger 2]()
3. [[Android] Dagger 2 - Phần III: Custom scope trong dagger 2]()

# Kiến thức đầu vào

* Không bị ở lại lớp vì môn *Lập trình hướng đối tượng*, *Lập trình Android cơ bản*
* Đã đi học thêm về *Kotlin* (Vì các ví dụ mình sẽ sử dụng Kotlin)

# Lý thuyết

Trước hết, chúng ta đi qua một số khái niệm để hiểu rõ hơn background của vấn đề:

### Dependency

*Dependency* là từ dùng để mô tả việc một module cấp cao phụ thuộc vào một module cấp thấp. VD: khi chúng ta học toán, chúng ta cần có một quyển sách toán. Ta có thể trừu tượng hóa vấn đề đó thành các đối tượng: `Student` và `MathBook`
```
class Student {

    private var book: MathBook

    constructor() {
        this.book = MathBook()
    }

    fun learn() {
        println("Learning ${book.getSubjectName()}")
    }
}

class MathBook {

    fun getSubjectName(): String {
        return "Math"
    }
}
```

Và ở hàm `main()`, ta sẽ gọi như sau:
```
fun main(args: Array<String>) {
    val student = Student()
    student.learn()
}
```

Ở đây, học sinh cần quyển sách toán để học và nếu không có sách toán, học sinh không thể hoàn thành việc học được. Bởi vậy, ta nói "`Student` phụ thuộc vào `MathBook`" hay "`MathBook` được gọi là dependency của `Student`".

Theo cách tiếp cận OOP, các class sẽ tương tác qua lại với nhau để có thể hoàn thành những chứng năng của app. Như trong ví dụ trên, `Student` tương tác (khởi tạo và quản lý vòng đời) với `MathBook`. Điều này sẽ dẫn chúng ta đến một khái niệm mới: *hard dependency*.

*Hard dependency* là khái niệm để mô tả những trường hợp các dependency được khởi tạo trực tiếp thay vì được truyền vào từ bên ngoài vào. Tại sao *hard dependency* lại không tốt:
* *Hard dependency* làm giảm tính tái sử dụng của các module
* *Hard dependency* làm việc kiểm thử khó khăn hơn
* *Hard dependency* làm cho việc maintain code khó hơn khi project được scale up.

##### #Tính tái sử dụng

Khi tồn tại nhiều *hard dependency*, mỗi khi khởi tạo module cấp cao sẽ bắt buộc khởi tạo các module cấp thấp. Các module sẽ bị ràng buộc với nhau hơn. Từ đó, tính tái sử dụng của các module sẽ bị giảm đi

##### #Kiểm thử

Khi thực hiện việc kiểm thử, ta cần cô lập module với các phần còn lại của app bằng cách mock các module cấp thấp. Tuy nhiên, nếu ta *hard dependency* như trường hợp ở trên, ta không thể mock `book` để test được module `Student`.

##### #Khả năng maintain code

Cuối cùng, nếu các module không được tái sử dụng, nếu quá trình kiểm thử không được thực hiện đầy đủ mà project lại ngày càng phình to ra, việc maintain code đều sẽ gây khó khăn đối với cả các "ma cũ" lẫn "ma mới" của project.

**Tóm lại vấn đề**: việc *hard dependency* sẽ làm cho các module *tight coupling* (dính chặt) vào nhau hơn, từ đó sẽ làm giảm tính tính tái sử dụng và tính mở rộng của module (Ngược lại với *tight coupling*, chúng ta có *loose coupling* và sẽ là mục tiêu mà chúng ta hướng đến khi sử dụng DI). Để giải quyết vấn đề này, chúng ta sẽ áp dụng nguyên lý *Inversion of Control* (IoC)

### Inversion of control - IoC

*IoC* là một design principle (không phải là design pattern), được sử dụng để đảo ngược các loại điều khiển khác nhau trong OOP design. Ở đây, việc điều khiển có thể là:
* Điều khiển flow của một ứng dụng
* Điều khiển flow của việc khởi tạo các dependency của một module.

Và ý thứ hai+ viết hôm nay của chúng ta. Chúng ta tiếp tục với ví dụ ở phần trước: `MathBook` được khởi tạo trực tiếp bên trong `Student`. Từ đó ta có *hard dependency*. *IoC* gợi ý chúng ta giải quyết vấn đề này bằng cách đảo ngược việc điều khiển: thay vì tự khởi tạo, đơn giản ta chỉ cần giao việc khởi tạo này cho một class khác. Ví dụ ở trên có thể được sửa lại như sau:
```
class Student {

    private var book: MathBook

    constructor() {
        this.book = BookFactory.getBook()
    }

    fun learn() {
        println("Learning ${book.getSubjectName()}")
    }
}

object BookFactory {

    fun getBook(): MathBook {
        return MathBook()
    }
}
```

Thay vì quan tâm đến việc khởi tạo, ta lấy `MathBook` từ `BookFactory` và không quan tâm xem `MathBook` được tạo ra thế nào nữa. Cách giải quyết này sử dụng design pattern *Factory*, một trong những design pattern impelement *IoC*.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/6k99fg3w4p_ioc-patterns.png">
  Các design pattern implement <i>Inversion of control</i>. Nguồn: <a href="https://www.tutorialsteacher.com/ioc/inversion-of-control">Inversion of Control</a>
</p>

Tuy nhiên, khi project được scale up lên, chúng ta vẫn chưa hoàn toàn giải quyết được vấn đề để đạt được *loose coupling*. Một số vấn đề phát sinh là:
* `Student` đang là module cấp cao, phụ thuộc vào một module cấp thấp hơn là `MathBook`. Vì vậy, việc sửa đổi module cấp thấp sẽ kéo theo một loạt các sửa đổi ở module cấp cao, điều đó làm việc maintain code trở nên phức tạp hơn.
* Nếu càng ngày `Student` càng học lên cao, môn học sẽ thay đổi theo thời gian và sách cũng cần thay đổi theo.

Để giải quyết vấn đề này, ta cần đến một design principle khác: *Dependency inversion*

### Dependency inversion - DIP

Trong những nguyên lý thiết kế trong lập trình hướng đối tượng *SOLID*, *DIP* là nguyên lý cuối cùng. Nội dung của nguyên lý này như sau:

* Các module cấp cao không nên phụ thuộc vào các module cấp thấp hơn. Cả hai nên phụ thuộc vào trừ tượng
* Trừu tượng không nên phụ thuộc vào chi tiết mà chi tiết nên phụ thuộc vào trừu tượng.

Lần đầu tiên đọc nội dung này, mình thấy abstract vãi nồi~~ Có lẽ chúng ta nên "phụ thuộc" vào chi tiết trước (lấy ví dụ), rồi mới nên "phụ thuộc" vào trừu tượng sau (đọc lại nguyên lý để ngẫm tiếp) :D.

Mình sẽ tiếp tục phân tích ví dụ phía trên: Ta thấy module cấp cao là `Student` đang phụ thuộc vào `MathBook` tức là phụ thuộc vào chi tiết thay vì trừu tượng. Để code tuân thủ đúng theo *DIP*, ta cần trừu tượng hóa module cấp thấp bằng cách tạo một interface:
```
interface TextBook {

    fun getSubjectName(): String
}
```

Từ đó, bất kỳ quyển sách mới nào cũng cần implement `TextBook` và triển khai các function bên trong:
```
class MathBook : TextBook {

    override fun getSubjectName(): String {
        return "Math"
    }
}
```

Đối với `BookFactory`, ta cũng sẽ trả về một kiểu trừu tượng `TextBook` thay vì trả về một kiểu cụ thể:
```
object BookFactory {

    fun getBook(): TextBook {
        return MathBook()
    }
}
```

Cuối cùng, ta sửa thuộc tính `book` của `Student` thành kiểu trừu tượng để nếu có học sang quyển sách khác, ta không cần phải sửa đổi `Student`:
```
class Student {

    private var book: TextBook

    constructor() {
        this.book = BookFactory.getBook()
    }

    fun learn() {
        println("Learning ${book.getSubjectName()}")
    }
}
```

Sau khi sửa đổi, việc thiết kế đã tuân theo các rule của *DIP*:
* Cả module cấp cao `Student` và module cấp thấp `MathBook` đều phụ thuộc vào trừu tượng (`TextBook`).
* Chi tiết đang phụ thuộc vào trừu tượng: `Student` phụ thuộc vào `TextBook` thay vì `MathBook`.

Việc này giúp chúng ta lại tiến thêm một bước nữa trong việc đạt được *loose coupling*: Chúng ta có thể sửa đổi module cấp thấp mà không làm ảnh hưởng đến module cấp cao hoặc chúng ta có thể thay thế hẳn một module cấp thấp khác, miễn là nó tuân theo cái khung `TextBook`.

Như vậy, chúng ta đã tìm hiểu về khái niệm *IoC* và *Dependency inversion* nhằm mục đích đạt được *loose coupling*. Với ví dụ đã xét, chúng ta sử dụng design pattern *Factory* để đạt được *IoC*. Tuy nhiên, với *Factory*, module cấp cao vẫn có một chút liên quan đến việc khởi tạo module cấp thấp khi chúng ta get ra module cấp thấp từ *Factory* khi muốn khởi tạo. Để hoàn toàn tách rời việc khởi tạo module cấp thấp ra khỏi module cấp cao, chúng ta sẽ áp dụng một design pattern khác và là main của series này: *Dependency injection*.

<p align="center">
  <img src="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/45dk2f9666_diana-polekhina-meEnpiwpzTA-unsplash.jpg">
  Photo by <a href="https://unsplash.com/@diana_pole?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Diana Polekhina</a> on <a href="/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
</p>

### Dependency injection

Vậy *Dependency injection* là gì? *DI* là một design pattern tuân theo *IoC*. Nó giúp chúng ta khởi tạo các dependency bên ngoài của module và cung cấp các dependency đó cho chúng ta thông qua nhiều cách. Với việc sử dụng *DI*, chúng ta đã loại bỏ hoàn toàn việc liên quan của một module đến việc khởi tạo các dependency của module đó.

Chúng ta có thể mô tả *DI* bằng quan hệ của ba class:
1. Client class: Module cấp cao và sẽ phụ thuộc vào service class.
2. Service class: Module cấp thấp và là phụ thuộc của client class.
3. Injector class: Người trung gian kết nối client class với service class bằng cách inject service class vào client class.

Và mô hình sau sẽ giúp chúng ta mường tượng ra dễ hơn sự kết nối giữa ba class:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/p0yv1x6gay_DI.png)

Như chúng ta thấy, *injector class* khởi tạo *service class* và inject nó vào *client class*. Theo cách này, *DI* làm cho *client class* sẽ hoàn toàn không liên quan đến việc khởi tạo *service class* như thế nào hay bao giờ nữa.

#### Các kiểu inject

*Injector class* có 3 cách để inject một *service class* vào *client class*. Đó là constructor injection, property injection và method injection.
* Constructor injection: *injector class* inject *service class* thông qua constructor của *client class*
* Property injection: *injector class* inject *service class* trực tiếp vào một public property của *client class*
* Method injection: *client class* sẽ implement một interface chứa một method để cung cấp dependency và *injector class* sẽ sử dụng interface này để cung cấp *service class* cho *client class*.

Chúng ta sẽ tiếp tục nâng cấp ví dụ ban đầu để làm ví dụ cho từng kiểu inject

##### Constructor injection

Chúng ta sẽ thêm một constructor cho `Student` để *injector class* có thể inject `TextBook` vào như sau:
```
class Student {

    private var book: TextBook

    constructor() {
        this.book = MathBook()
    }

    // TextBook sẽ được inject vào thông qua constructor này
    constructor(book: TextBook) {
       this.book = book
    }

    fun learn() {
        println("Learning ${book.getSubjectName()}")
    }
}
```

Đối với kiểu inject này, khi có chỗ nào cần một đối tượng `Student`, một đối tượng mới sẽ được khởi tạo và một đối tượng `TextBook` cũng được khởi tạo và truyền vào `Student`.

##### Property injection

Đối với *property injection*, ta cần để access modifier của property muốn được inject thành *public* bởi cách inject này thực chất là *injector class* sẽ inject *service class* trực tiếp bằng cách gán giá trị.
```
class Student {

    lateinit var book: TextBook

    constructor() {
      Injector.inject(this)
    }

    fun learn() {
        println("Learning ${book.getSubjectName()}")
    }
}

object Injector {

    fun inject(student: Student) {
        student.book = createBook();
    }

    private fun createBook(): TextBook {
        return MathBook()
    }
}
```

Ở đây, khi khởi tạo `Student` (hoặc một một lúc nào khác mà chúng ta muốn), chúng ta sẽ yêu cầu `Injector` cung cấp dependency cho `Student`, và *Injector* sẽ thỏa mãn chúng ta.

##### Method injection

Để inject bằng *method injection*, *client class* cần có một setter method để *injector class* truyền *service class* vào *client class*
```
class Student {

    private lateinit var book: TextBook

    fun learn() {
        println("Learning ${book.getSubjectName()}")
    }

    fun setTextBook(book: TextBook) {
        this.book = book
    }
}
```

Với kiểu inject này, sẽ được inject thông qua function `setTextBook()` tại mộ thời điểm nào đó.

Vậy là, trong bài học đầu tiên với một mả lý thuyết này, chúng ta đã bắt đầu vào đời với một chương trình nhỏ, ngây thơ và trong sáng. Nhưng rồi, khi bài toán được mở rộng ra: mối quan hệ giữa các class nhiều hơn, số lượng dependency phải khởi tạo và quản lý nhiều hơn. Chúng ta sẽ cần thêm những design principle như *Inversion of Control*, *Dependency inversion* hay những design pattern như *Dependency injection* để giữ cho code của chúng ta mãi mãi tuổi 18.

Cuối cùng, mình hy vọng các bạn sẽ phân biệt được các khái niệm nghe có vẻ na ná nhau này, sau đó là nắm rõ, trước khi đi vào tìm hiểu một công cụ không mới nhưng không phải ai cũng chắc tay khi sử dụng; một công cụ nhằm giải quyết những nhàm chán lặp lại khi viết code nếu chúng ta tự implement các khái niệm đã được đề cập ở phần I này. Đó là **Dagger 2**.
