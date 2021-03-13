1. Gradle là gì?
- Android studio sử dụng Gradle - an advanced build toolkit
- Gradle chạy độc lập với Android studio, do đó có thể build app trên máy mà không cần cài Android studio(CI server)
2. Quá trình build

[![](https://developer.android.com/images/tools/studio/build-process_2x.png)]

b1. Compiler chuyển source code thành file DEX (Dalvik excecutable) và resource thành compiled resource
b2. APK Packager kết hợp DEX file vào compiled resource thành APK
b3. Sign key cho APK đó bằng debug hoặc release key
 - Debug version: Packager sử dụng debug keystore được AS tự động config
 - Release version: Cần cung cấp release keystore
b4. Trước khi gen ra APK, Packager sửa dụng zipalign tool để optimize app nhằm sử dụng ít memory hơn khi chạy trên 1 device

3. Custom build config

* Buid types
- Thường gắn với các stage  của chu trình phát triển debug và release

* Product flavors
- Mỗi một flavor đại diện cho một version của app: bản free và bản trả phí. Với Product flavor, bạn dùng chung những phần chung và thay đổi những phần khác như resource hoặc code.

* Build variant
- Mỗi build variant là sự kết hợp của một product flavor và một build type: free debug version, paid debug version, free release version, paid release version

* Dependency
- Qản lý các dependency local và remote thay vì phải tự download và add vào app

* Signing
- Có thể chỉ định các Signing config để khi build sẽ tự sign key theo signing config đã chỉ định

* Code and resource shrinking
- Lược bỏ những đoạn code và resource không dùng đến -> giảm size của APK

* Multiple APKs

4. Build configuration files

![](https://developer.android.com/images/tools/studio/project-structure_2x.png)

* The Gradle settings file - `settings.gradle`
- config xem project có bao nhiêu module

```
include ‘:app’
```

* build.gradle của file project
- chứa config của toàn bộ module trong project
- gồm 2 phần
  + block buildscript: config *repository* và *dependency* cho chính Gradle
    . *repository*: jcenter, maven
    .*dependency*:
  + block allprojects: config *repository* và *dependency* cho tất cả module trong project: plugins và 3rd-party libaries. Tuy nhiên, thay vì config ở đây, bạn nên config ở file build.gradle của từng module
- Định nghĩa các extra properties bằng block ext để tất cả các module có thể dùng được
```
ext {
    // The following are only a few examples of the types of properties you can define.
    compileSdkVersion = 28
    // You can also create properties to specify versions for dependencies.
    // Having consistent versions between modules can avoid conflicts with behavior.
    supportLibVersion = "28.0.0"
    ...
}
```
Cách truy cập từ các module: `rootProject.ext.compileSdkVersion`

* build.gradle của file module
