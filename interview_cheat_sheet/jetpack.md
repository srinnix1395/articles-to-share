1. Định nghĩa
> Jetpack is a suite of libraries to help developers follow best practices, reduce boilerplate code, and write code that works consistently across Android versions and devices so that developers can focus on the code they care about.

2. LiveData
  * LiveData là một observable data holder class, có lifecycle aware
  * Chỉ update data khi view ở trạng thái active (STARTED/RESUMED)
  * Tự động ngắt kết nối khi ở trạng thái DESTROYED -> no memory leaks, no crash due to stopped activities.
  * Transformation: lazily calculated, chỉ thay đổi khi LiveData lắng nghe thay đổi
    - map: lắng nghe và thay đổi value
    - switchMap: lắng nghe và phải trả về một cái LiveData khác
  * Merge multiple LiveData source
    - MediatorLiveData

3. ViewModel
