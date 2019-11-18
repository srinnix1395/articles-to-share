## R8

Sau một quá trình code và debug vất vả hoặc cực kỳ vất vả, mọi developer đều có một mong muốn và khao khát cuối cùng: release thành công ứng dụng. Tuy vậy, quãng đường còn lại sẽ chưa hết chông gai nếu





## Outline

### Giới thiệu

* Nên enable shrinking khi build release để tối ưu hóa size của ứng dụng nhất có thể
* Khi build app với Android Gradle plugin 3.4.0 trở lên, R8 sẽ được mặc định bật, thay thế cho proguard trước đây.
* Khi enable shrinking, những công việc mà plugin sẽ làm
  * **Code shrinking (tree-shaking):** detect và remove unreachable code - những class, field, method và attribute không được sử dụng, trong *app* và *library mà app sử dụng*. Vd: bạn chỉ sử dụng vài API của một thư viện to đùng, những phần còn lại sẽ được bỏ đi
  * **Resource shrinking:** detect và remove unused resource trong *app* và *library mà app sử dụng* bao gồm cả những resource được tham chiếu đến bởi những đoạn code không được sử dụng (kết hợp với *Code shrinking* và bắt buộc phải enable *Code shrinking* để sử dụng được)
  * **Obfuscation:** Rút ngắn tên của các class và các thành phần của class, từ đó giảm kích thước của file DEX
  * **Optimization:** Quét và viết lại những đoạn code chưa tối ưu để tiếp tục giảm kích thước của file DEX. VD: một đoạn câu lệnh if/else có phần `else` không bao giờ được chạy, plugin sẽ loại bỏ đoạn code trong phần `else` đó.

### Enable và disable từng chức năng

Chỉ nên bật các chức năng trên khi build release bởi những lý do sau:
* Quá trình optimization sẽ làm tăng thêm thời gian build
* Dễ xuất hiện những lỗi không mong muốn vì không chỉ định những phần nào cần được keep lại.

Để enable shrinking, obfuscation và optimization, ta cần thêm đoạn code sau vào file `build.gradle` của **project**:

```
android {
  buildTypes {
      release {
          // Enable Code shrinking, Obfuscation và Optimization
          minifyEnabled true

          //todo
          useProguard true

          // Enable Resource shrinking, performed by the Android Gralde plugin
          // Cần enable Code shrinking trước khi enable chức năng này.
          shrinkResources true

          // Chỉ định file config để R8 quyết định những file được keep,...
          proguardFiles getDefaultProguardFile('proguard-android.txt'),
                  'proguard-rules.pro'
      }
  }
}
```

### R8 configuration files

*R8* cũng sử dụng các file chứa rules của *ProGuard* làm căn cứ để thực hiện các chứng năng shrinking, obfuscation hay optimization. Bởi vậy, khi migrate từ *ProGuard* lên *R8*, chúng ta gần như không cần phải sửa đổi gì nhiều để app chạy nuột như bình thường. Dưới đây là những file rules của *ProGuard* mà *R8* sẽ sử dụng:

| Source                               | Đường dẫn                       | Mô tả                                                                                                                                                  |
|--------------------------------------|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| Android Studio                       | <module-dir>/proguard-rules.pro | Đây là file rules được sinh ra bởi Android studio khi tạo project. Mặc định, file này không có gì cả và developer sẽ thêm rules của mình vào file này. |
| Android Gradle plugin                | a                               | a                                                                                                                                                      |
| Library dependencies                 | a                               | a                                                                                                                                                      |
| Android Asset Package Tool 2 (AAPT2) | a                               | a                                                                                                                                                      |
| Custom configuration files           | a                               | a                                                                                                                                                      |

Khi `minifyEnabled` có giá trị bằng `true`, *R8* sẽ kết hợp tất cả các rules ở tất cả files trên để shrink, obfuscate và optimize code. Đây là điều cần phải lưu ý khi fix bug với *R8* bởi có thể các *Library dependencies* sẽ làm ảnh hưởng đến rules chung. Bởi vậy, developer có thể list ra tất cả những rules mà *R8* apply bằng câu lệnh sau:

```
//Đường dẫn của file là tùy vào developer
-printconfiguration ~/tmp/full-r8-config.txt
```

Để xác định rules của mình, developer cần sửa file `<module-dir>/proguard-rules.pro` - file này mặc định rỗng khi project được tạo bằng Android Studio.

### Shrinking your code

Để enable *Code shrinking*, ta để giá trị của biến `minifyEnabled` là `true`.

*Code shrinking* là quá trình phân tích và loại bỏ những đoạn code sẽ không được chạy lúc run-time. Từ đó, nó sẽ làm giảm đáng kể size của file apk.

Quá trình này bắt đầu bằng việc *R8* xác định tất cả các entry point của app (tất nhiên là kết họp với các file config ở trên). Những entry point này bao gồm tất cả các đoạn code mà Android chạy để mở một Activity hoặc Service của app. Từ mỗi entry point, *R8* sẽ cho ra một graph về tất cả các method, member variable,... nói chung là tất cả các class mà app sẽ access đến lúc run-time. Những thành phần không được đưa vào graph sẽ là unreachable code và sẽ được loại bỏ.

![alt text](https://developer.android.com/studio/images/build/r8/tree-shaking.png)

Như ví dụ ở trên, *R8* xác định được 1 entry point là `MainActivity.class` và các các method `foo()`, `faz()` và `bar()` và class `AwesomeApi.class` sẽ được truy cập lúc run-time. Cùng với đó, method `baz()` là unreachable code và sẽ được loại bỏ sau quá trình *Code shrinking*

#### Xác định những đoạn code cần được keep

Trong các trường hợp sau, *R8* không đủ "smart" để xác định đoạn code này có được truy cập lúc run-time:

* Gọi một method từ JNI
* Sử dụng reflection.

Khi đó, *R8* sẽ remove những đoạn code tưởng là không được sử dụng, từ đó throw ra những exception như `ClassNotFoundException`,...

Bởi vậy, ta cần tự xác định những đoạn code nào cần giữ lại(không đổi tên) bằng cách thêm rule dòng vào file `proguard-rules.pro`. VD:

```
-keep public class com.example.demo.SomeClass
```

#### Rule

Một rule sẽ gồm 2 thành phần cấu thành:

- Phần đầu xác định "cách keep".
- Phần sau là định danh của class hoặc member class cần được keep.

//todo
![alt text]()

###### Keep cheatsheet

Mục đích của *R8* là giảm size của file APK đến mức thấp nhất có thể. Bởi vậy, chúng ta cần chọn chính xác phiên bản *-keep* để không làm giảm hiệu quả của *R8*. Những gạch đầu dòng dưới đây sẽ thể hiện quá trình nào sẽ được chạy (*✔* là có và *✘* là không)

* no rules: *R8* sẽ thực hiện cả 2 quá trình *shrink* và *obfuscate* với cả class hoặc member class.

![alt text](https://jebware.com/blog/wp-content/uploads/2017/11/none.png)

* *-keep*: *R8* sẽ **không** thực hiện cả 2 quá trình *shrink* và *obfuscate* với class hoặc member class.

![alt text](https://jebware.com/blog/wp-content/uploads/2017/11/keep.png)

* *-keepclassmembers*: *R8* sẽ chỉ thực hiện 2 quá trình *shrink* và *obfuscate* với class. Tức là nếu class không được sử dụng, class sẽ bị remove. Nếu class được sử dụng, class sẽ được giữ lại và bị đổi tên. Tuy nhiên, các thành phần bên trong sẽ được giữ nguyên (remove hay đổi tên gì cả)

![alt text](https://jebware.com/blog/wp-content/uploads/2017/11/keepclassmembers.png)

* *-keepnames*: *R8* sẽ chỉ thực hiện quá trình *shrink* với class hoặc member class. Nếu class hoặc member class được sử dụng, tên của chúng sẽ không bị thay đổi

![alt text](https://jebware.com/blog/wp-content/uploads/2017/11/keepnames.png)

* *-keepclassmembernames*: *R8* thực hiện quá trình *shrink* với cả class và member nhưng chỉ *obfuscate* với class. Tức là nếu class hoặc member class không được sử dụng chúng sẽ bị loại bỏ, nếu được sử dụng, chỉ class bị đổi tên, member class sẽ được giữ nguyên tên.

![alt text](https://jebware.com/blog/wp-content/uploads/2017/11/keepclassmembernames.png)

Ngoài ra, để hoàn thiện một rule, chúng ta cần chú ý đến phần đứng sau *-keep*. Đây là phần xác định

###### Định danh

```
-keepclassmembernames public class com.mitsubishimotors.mymitsubishiconnect.sections.** {
    public getClassNameAnalytic();
}
```

### Resource Shrinking

Để có thể enabled Resource shrinking, ta bắt buộc phải enable `minifyEnabled` bởi chỉ khi remove được unreachable code, những unused resource tương ứng mới xuất hiện hoàn toàn. Để enable Resource shrinking, ta thêm đoạn sau vào file `build.gradle` của project:

```
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}
```

#### Xác định những resource cần được keep

Để xác định chính xác những file resource nào cần được keep và những file nào cần được loại bỏ, ta tạo một file xml như sau và save vào một thư mục resource của app. VD: `res/raw/keep.xml` (tên file là tùy ý).

```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
    tools:discard="@layout/unused2" />
```

Với `tools:keep` là những file cần được giữ lại và `tools:discard` là những file cần được loại bỏ.

Sở dĩ ta cần xác định những file cần keep có thể là bởi app chỉ tham chiếu động đến resource đó:

```
val idString = "icon_" + (LOADING_START_FRAME + i)
val id = resources.getIdentifier(idString, "drawable", getContext().getPackageName())
val drawable = resources.getDrawable(id, null)
```

Với `tools:discard`, tại sao lại không chủ động xóa file resource đó đi mà lại phải phụ thuộc vào *R8*? Thực chất, `tools:discard` hiệu quả khi app sử dụng build variant, tùy vào flavor mà resource nào sẽ được xóa đi để giảm kích thước của file apk đến mức tối đa.

#### Remove unused alternative resource

Quá trình Resource shrinking chỉ loại bỏ những file resource mà code không reference đến. Điều đó có nghĩa là những file resource của các config khác nhau sẽ không bị remove. Để xác định file resource của config nào cần được remove. Ta làm như sau:

```
android {
    defaultConfig {
        ...
        resConfig "en", "fr"
    }
}
```

Với đoạn code ở trên, quá trình Resource shrinking sẽ loại bỏ những file resource không phải ngôn ngữ Anh hoặc Pháp. Việc này thích hợp khi bạn build nhiều file APK tương ứng với nhiều config device khác nhau.

### Obfuscate code

Mục đích của quá trình obfuscation là làm giảm size app bằng việc cắt ngắn tên các class, method và field của app. Vì thay đổi tên của các thành phần, developer có thể gặp một số trở ngại sau khi obfuscate code như khó khăn khi trace code, xác định crash... Ngoài ra, với những class được access thông qua reflection, bạn nên keep lại những file đó để app không crash vì không tìm thấy các thành phần tương ứng.


#### Decode obfuscated stack trace

Sau khi code được obfuscate, việc đọc code cũng như trace code là cực kỳ khó khăn. Bởi vậy, *R8* sẽ tạo một file `mapping.txt` với mỗi lần build. File này chứa thông tin của các file được obfuscate như tên class, method, field hay số thứ tự dòng mà *R8* đã thay đổi. Từ file này, ta có thể map từ file obfuscated thành file gốc để dễ hiểu, dễ đọc hơn. File `mapping.txt` này được lưu ở đường dẫn `<module-name>/build/outputs/<build-type>/`.

Lưu ý: Vì file `mapping.txt` sẽ được ghi đè mỗi lần build project, bạn nên lưu file này mỗi lần release để có thể trace lại code dù ở bất kỳ version nào của app.


### Code optimization


### Reference

* [Shrink, obfuscate, and optimize your app](https://developer.android.com/studio/build/shrink-code#shrink-code)
* [Distinguishing between the different ProGuard “-keep” directives](https://jebware.com/blog/?p=418)
