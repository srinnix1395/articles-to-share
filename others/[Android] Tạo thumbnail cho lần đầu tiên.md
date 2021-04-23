Upload ảnh là một chức năng thường thấy ở các ứng dụng trên smartphone hiện nay. Tuy nhiên, khi upload một tấm ảnh lên, các ứng dụng thường sẽ scale down tấm ảnh đó xuống đến một size nào đó để giảm dung lượng lưu trữ, thay vì upload tấm ảnh original.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/7buwns1cbt_photo-1530251985675-fa6a8ceb0d63.jpg) Facebook luôn scale down bức ảnh với max size là 960px * 640px. Nguồn ảnh: https://unsplash.com/

Vậy, để tạo một thumbnail cho một bức ảnh trước khi upload lên server, chúng ta cần làm gì? Các bước sẽ là load bức ảnh đó vào memory, tạo một một cái "scaled bitmap" và compress nó lại thành một tấm ảnh bình thường (JPG, PNG...). Nghe có vẻ phức tạp chút xíu nhưng Android đã có những function hỗ trợ hết rồi, việc của chúng ta chỉ là nghiên cứu một chút xem "đút" (insert :v) cái gì vào giữa ...2 dấu ngoặc đơn khi gọi hàm, rồi gọi chúng ra, là chấm hết.

## Load một tấm ảnh vào memory  

Google nào, xem Android trang bị cho chúng ta cái gì nào?

```
    Bitmap bitmap = BitmapFactory.decodeResource(resources, R.drawable.background_activity_main);
```

Với `BitmapFactory`, bạn có thể decode ra một bitmap từ nhiều nguồn khác nhau bằng các hàm như: `decodeByteArray()`, `decodeFile()`, `decodeResource()`... Lưu ý một chút là việc decode này là một task không nhẹ nhàng gì, có thể (hoặc gần như chắc chắn), nó sẽ kéo dài quá 16ms (khoảng cách giữa 2 lần update của main thread), và nếu không muốn ứng dụng của bạn lắp ba lắp bắp hay thậm chí là chào cờ luôn, hãy nhớ "move it to a background thread". Tuyệt, vậy coi như là xong bước 1? Chưa đâu, chúng ta sẽ thử load bức ảnh trên vào memory xem sao (thông số của bức ảnh khi chưa được decode: size 1500px * 1000px, dung lượng trên disk: khoảng 248 KB và được lưu ở định dạng JPG).

Sau khi decode, ta sử dụng hàm `bitmap.byteCount` để lấy ra dung lượng của bức ảnh khi ở trên memory. Kết quả là 6.000.000 byte - tương đương với khoảng 5.7MB (ở đây mình đang test trên một device có độ phân giải chuẩn là 160dpi, trên các thiết bị có độ phân giải nhỏ hoặc lớn hơn, dung lượng của ảnh khi được load vào memory cũng sẽ phải tỷ lệ tương ứng). Bạn sẽ thấy có gì đó sai sai, vì dung lượng cửa bức ảnh khi lưu trên disk chỉ là khoảng 248KB. Tuy nhiên, điều đó không sai. Lý do là vì khi lưu lên disk (dưới dạng JPG, PNG...), ảnh sử dụng các thuật toán tương ứng với từng format để nén ảnh lại, làm cho dung lượng của ảnh giảm xuống rất nhiều. Thực tế khi load vào memory, ảnh sẽ được bung ra hoàn toàn, nên coi như đây mới là dung lượng thật của bức ảnh.

Đến đây chúng ta thử ngẫm một chút: chúng ta mới load duy nhất một tấm ảnh cỡ trung bình vào memory và đã ngốn gần 5.7MB. Nếu chúng ta hiển thị nhiều bức ảnh một lúc như là một list các bức ảnh bằng RecyclerView chẳng hạn, mỗi item hiển thị một hình ảnh thì với 1 ứng dụng như gallery, hiển thị cùng lúc vài chục bức ảnh. Lượng memory mà chỗ ảnh đó ngốn sẽ là khá lớn. Ngoài ra, việc load như vậy còn dễ làm cho chúng ta gặp phải *java.lang.OutOfMemory* exception, khi mà bộ nhớ của các smartphone là rất khiêm tốn. Vậy với những library chúng ta hay dùng như glide, piccaso... dù là list gần trăm tấm ảnh, FPS vẫn ở mức 60 như thường. Đây là giải pháp: giảm size của bức ảnh xuống đến mức hợp lý khi load vào memory. Nếu một ImageView chỉ có size là 100px * 100px thôi, vậy tại sao chúng ta phải load một bức ảnh 1500px * 1000px vào memory để làm gì?

Để làm được điều đó, chúng ta cần chú ý đến tham số thứ 3 khi sử dụng các hàm decode bitmap: *BitmapFactory.Options*.

```
    val options = BitmapFactory.Options()
    options.inJustDecodeBounds = true
    BitmapFactory.decodeResource(resources, R.drawable.background_activity_main, options)
```

Khi để **inJustDecodeBounds** bằng **true**, bitmap trả về sẽ bằng `null`. Tuy nhiên, thông tin của ảnh vẫn sẽ được trả về: width, height, mimeType... Từ đó, chúng ta có thể tính toán xem size của bitmap muốn load lên memory là bao nhiêu, bằng cách sử dụng một thuộc tính nữa của *BitmapFactory.Options*. Đó là **inSampleSize**. Đây là giá trị để quyết định xem size của bitmap sẽ được scale xuống bao nhiêu lần sau khi decode. Ví dụ: bức ảnh nguyên bản có size là 1500px * 1000px, nếu chúng ta để **inSampleSize** bằng **5**, khi load lên memory, ta sẽ được một bitmap có size là 300px * 200px.

```
    options.inSampleSize = 5
```

Tiếp tục, vậy **inSampleSize** bằng bao nhiêu là hợp lý? Điều đó là *tùy thuộc vào bạn*, bạn thích để bao nhiêu thì để :v Nói vậy thôi, phải tính toán đàng hoàng chứ, vì nếu không tính toán mà cứ hardcode đúng một giá trị trong tất cả các trường hợp thì chắc chắn là không ổn tẹo nào. Tuy nhiên, việc tính toán thế nào cũng vẫn là *tùy thuộc vào bạn* :v. Bạn có thể viết một thuật toán để tính toán chẳng hạn. Trong trường hợp không có yêu cầu gì quá đặc biệt, bạn có thể sử dụng cách tính *inSampleSize* mà document của Android đã đề cập đến, nó được tính dựa trên số mũ của 2 (giá trị này luôn luôn phải là lũy thừa của 2, với các giá trị không phải là lũy thừa của 2, Android sẽ làm tròn xuống đến giá trị gần nhất là lũy thừa của 2).

```
fun calculateInSampleSize(options: BitmapFactory.Options, reqWidth: Int, reqHeight: Int): Int {
        // Raw height and width of image
        val height = options.outHeight
        val width = options.outWidth
        var inSampleSize = 1

        if (height > reqHeight || width > reqWidth) {

            val halfHeight = height / 2
            val halfWidth = width / 2

            // Calculate the largest inSampleSize value that is a power of 2 and keeps both
            // height and width larger than the requested height and width.
            while (halfHeight / inSampleSize >= reqHeight && halfWidth / inSampleSize >= reqWidth) {
                inSampleSize *= 2
            }
        }

        return inSampleSize
    }
```

Sau khi đã tính được *inSampleSize* bằng bao nhiêu, chúng ta hãy load lại bức ảnh vào memory xem size của bức ảnh bây giờ là bao nhiêu nhớ. Nhớ đổi giá trị cho **inJustDecodeBounds** bằng **false**.

```
    val options = BitmapFactory.Options()
    options.inJustDecodeBounds = true
    BitmapFactory.decodeResource(resources, R.drawable.background_activity_main, options)

    options.inSampleSize = calculateInSampleSize(options, 100, 100)
    options.inJustDecodeBounds = false;
    val bitmap = BitmapFactory.decodeResource(resources, R.drawable.background_activity_main, options)

    Log.e("size", "${bitmap?.byteCount}")
    Log.e("width", "${options.outWidth}")
    Log.e("height", "${options.outHeight}")
```

Bây giờ, **bitmap.byteCount** trả về giá trị 375.000 byte - tương đương với khoảng 366KB, width và height bây giờ là: 375px và 250px. Từ **6.000.000 byte** xuống còn **375.000 byte** - giảm đến 93.75% dung lượng của bức ảnh.

## Nén bức ảnh trở lại

Như đã nói ở trên, khi lưu ở trên disk, ảnh sẽ được nén lại để giảm dung lượng. Vì vậy, bước tiếp theo sẽ là nén bức ảnh lại trước khi upload lên server.

```
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, bos);
    byte[] bitmapData = bos.toByteArray();
```

Chúng ta sẽ sử dụng hàm `compress` của *Bitmap* với các tham số cần truyền vào là định dạng muộn lưu(JPG, PNG hoặc WEBP), chất lượng ảnh khi nén (100 là giữ nguyên chất lượng với ảnh gốc) và *ByteArrayOutputStream*. Một lưu ý nhỏ: hãy sử dụng định dạng JPG nếu muốn thay đổi chất lượng của bitmap, định dạng PNG thì không làm được như vậy.

Với các tham số như trên, sau khi nén lại, chúng ta được một bức ảnh có dung lượng là .... Vậy là xong, chúng ta sẽ upload cái *ByteArray* đó lên server.

Khi upload xong rồi và lên storage để check lại, chúng ta lại thấy một cái gì đó sai sai~~


Góc của ảnh đã bị xoay đi 90 độ, khác với góc chúng ta đã chụp trên device. Vậy là sao nhỉ?

## Góc xoay của ảnh

Hầu hết camera của các dòng điện thoại là landscape - tức là nếu bạn chụp một bức ảnh với điện thoại khi mà điện thoại được để dọc ra như bình thường chúng ta sử dụng, khi đó thực chất camera của điện thoại đã bị xoay đi một góc 90 độ. Để thấy rõ hơn điều này, bạn có thể vào ứng dụng gallery của điện thoại và xem thông tin chi tiết của một bức ảnh mà bạn chụp khi xoay dọc điện thoại, bạn có thể thấy thông tin về góc xoay của tấm ảnh như sau:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/y51qyua5ef_37369310_1953125774731562_122589610852220928_n.png)

Vì sao ứng dụng gallery vẫn hiển thị đúng góc mà bạn vẫn chụp, mà khi upload lên lại bị xoay ngang. Đó là vì ứng dụng gallery có thể đọc được góc xoay của camera khi chụp ảnh thông qua việc đọc [Exif](https://en.wikipedia.org/wiki/Exif#Example) của ảnh
. Từ đó, ứng dụng sẽ xoay ảnh một góc đúng bằng góc của của camera khi chụp bức ảnh để ảnh được hiển thị đúng như góc ta chụp bằng camera. Tuy nhiên, khi upload lên server, ảnh sẽ hiển thị ra với góc chụp nguyên bản - tức là góc 0 độ -> ảnh bị xoay tùm lum~~

Dựa vào cách làm của ứng dụng gallery, ta có thể bắt chước để xoay bức ảnh về đúng với góc mà ta đã chụp. Và khi ta upload lên server, bức ảnh sẽ đúng như chúng ta mong muốn. Trước hết, ta cần get được góc xoay của bức ảnh là bao nhiêu?
Để đọc được Exif của ảnh trong *Android*, ta sử dụng **ExifInterface**:

```
    val exif = ExifInterface(BufferedInputStream(stream))
    val exifOrientation = exif.getAttributeInt(ExifInterface.TAG_ORIENTATION, ExifInterface.ORIENTATION_NORMAL)
    stream.close()
```

Ở đây, ta cần truyền một Stream vào khi khởi tạo `exif` để có thể đọc được thông tin của bức ảnh. Sau đó, ta lấy góc xoay của bức ảnh bằng hàm `getAttributeInt()` với tham số thứ 2 là giá trị mặc định khi không tìm thấy giá trị của góc xoay. Tiếp theo, ta sẽ xoay bức ảnh như góc xoay khi chụp ảnh:

```
    var degrees = 0f
    when (exifOrientation) {
        ExifInterface.ORIENTATION_ROTATE_90 -> degrees = 90f
        ExifInterface.ORIENTATION_ROTATE_180 -> degrees = 180f
        ExifInterface.ORIENTATION_ROTATE_270 -> degrees = 270f
    }

    val resultBitmap = rotateBitmap(inputBitmap, degrees)
    inputBitmap.recycle()
```

Và cuối cùng, hàm `rotateBitmap()`:

```
    fun rotateBitmap(bitmap: Bitmap, degrees: Float): Bitmap {
        val matrix = Matrix()
        matrix.postRotate(degrees)
        return Bitmap.createBitmap(bitmap, 0, 0, bitmap.width, bitmap.height, matrix, true)
    }
```

Với `Matrix`, ta có các hàm để xoay ảnh: `preRotate()` - trừ đi một góc bao nhiêu độ, `postRotate()` - cộng thêm một góc bao nhiêu độ và `setRotate()` - set cụ thể rotate cho bức ảnh. Ở đây, ta sử dụng `postRotate()` vì ta cần xoay bức ảnh thêm một góc để bức ảnh như khi chúng ta chụp.

## Lên mây^^

Cuối cùng, ta đã có thể upload bức ảnh lên server. Và ta sẽ có một bức ảnh thumbnail đúng như ta mong muốn. (bow)

##Tham khảo

* [Loading Large Bitmaps Efficiently in Android](https://android.jlelse.eu/loading-large-bitmaps-efficiently-in-android-66826cd4ad53)
* [Loading Large Bitmaps Efficiently](https://developer.android.com/topic/performance/graphics/load-bitmap)
