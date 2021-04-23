 [Android] Implement StickyHeaderRecyclerView với ItemDecoration của RecyclerView

 Tại Google I/O 2014, cùng với sự ra mắt của Android Lollipop, Google đã giới thiệu `RecyclerView` - _a better ListView_ với nhiều cải tiến cho phép gia tăng hiệu năng, đồng thời giải quyết nhiều vấn đề tồn tại của phiên bản `ListView` cũ. Khi nói đến `RecyclerView`, chúng ta có thể nhắc đến một số khái niệm mới: `ViewHolder` (pattern này cho phép tăng performance và vẫn có thể sử dụng được với `ListView` tuy nhiên không bắt buộc), `LayoutManager`, `ItemDecoration` và `ItemAnimator`. Đây đều là những chức năng người dùng có thể sử dụng `RecyclerView` để giải quyết những bài toán phức tạp hơn so với `ListView` hay `GridView` ngày xưa có thể làm được. Trong bài viết hôm nay, mình xin chia sẻ một chút về `ItemDecoration`, một tính năng hầu hết không được các dev sử dụng vì có thể các dev vẫn quen với tư duy của `ListView` cũ hoặc chưa thấy được hết sức mạnh mà `ItemDecoration` có thể mang lại.

Trong khuôn khổ bài viết này, mình sẽ viết code bằng `Kotlin`, một ngôn ngữ được Google chính thức support từ Google I/O 2017\. Nếu kết thúc năm 2017, bạn vẫn đang trì hoãn rằng mình sẽ học `Kotlin`, mình sẽ học `Kotlin`... Để mình nhắc lại rằng: hiện tại là năm 2018 rồi, đừng là kẻ trì hoãn nữa (như mình), học đi hoặc không bao giờ. Và có thể khi bạn bối rối không biết bắt đầu từ đâu, [repo](https://github.com/ngohado/Kotlin-Docs) này sẽ là một bước giúp bạn rõ hơn về những cú pháp cơ bản của `Kotlin`. Good luck!

# Giới thiệu

Đầu tiên, theo doc chính thức của _Android_:

> An ItemDecoration allows the application to add a special drawing and layout offset to specific item views from the adapter's data set. This can be useful for drawing dividers between items, highlights, visual grouping boundaries and more.

Nói một cách đơn giản, `ItemDecoration` là một công cụ dùng để decor các item trong `RecyclerView`. Bài viết hôm nay sẽ đi qua một số task mà chúng ta có thể sử dụng `ItemDecoration` để hoàn thành.

# Spacing

Với `ListView` ngày xưa, để ngăn cách giữa các item, bạn có thể sử dụng thuộc tính của ListView ngay trong file layout:

```
  <ListView
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:divider="@android:color/black"
      android:dividerHeight="2dp" />
```

Tuy nhiên, với `RecyclerView`, chúng ta sẽ không thể thêm trực tiếp divider mà phải sử dụng `ItemDecoration` để vẽ divider, và có lẽ các dev nghĩ rằng nó khá là rắc rối. Mình cũng vậy, trước khi biết `ItemDecoration`, thường để ngăn cách các item trong một list một khoảng 16dp, mình sẽ thường làm thế này: sử dụng marginBottom với chính item, đồng thời để paddingTop = 16dp và clipToPadding = false với `RecyclerView`. Cách làm này khá nhanh, tuy nhiên với một trường hợp phức tạo hơn: một list hiển thị dạng grid với 3 cột như _Instagram_ này chẳng hạn:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/yps0gthadt_Screenshot_2018-01-22-14-33-52.png)

Với cách làm như ở trên, bạn sẽ phải dựa vào position của từng item mà set margin sao cho phù hợp: item số 0, 3, 6... phải marginEnd, item 2, 5, 8... phải marginStart rồi set lại `LayoutParams` trong `ViewHolder`, vân vân và mây mây. Tuy nhiên, cách làm này sẽ mất khá nhiều effort. Với `ItemDecoration`, việc ngăn cách các item được làm một cách tổng quát hơn, hoàn toàn không liên quan gì đến layout của từng item.

Để bắt đầu với `ItemDecoration`, bạn cần tạo một class kế thừa từ `ItemDecoration`:

```
  class GridSpacingDecoration(private val spacing: Int,
                              private val spanCount: Int) : RecyclerView.ItemDecoration() {

    override fun getItemOffsets(outRect: Rect?, view: View?, parent: RecyclerView?, state: RecyclerView.State?) {
        val position = parent.getChildAdapterPosition(view)
        val column = position % 3 // item column

        outRect.left = column * spacing / spanCount
        outRect.right = spacing - (column + 1) * spacing / spanCount
        if (position >= spanCount) {
            outRect.top = spacing
        }
    }

    override fun onDraw(c: Canvas?, parent: RecyclerView?, state: RecyclerView.State?) {
        super.onDraw(c, parent, state)
        //do nothing
    }

    override fun onDrawOver(c: Canvas?, parent: RecyclerView?, state: RecyclerView.State?) {
        super.onDrawOver(c, parent, state)
        //do nothing
    }
  }
```

Chúng ta cần quan tâm đến 3 method khi implement `ItemDecoration`, đó là: `getItemOffsets`, `onDraw` và `onDrawOver`. Với 3 method, có lẽ 2 mehod sau đã có tên khá là tự giải thích cho chức năng của nó (vẽ và vẽ lên trên - nôm na là như thế :v).

Với method đầu tiên, method này chính là công cụ cho phép ngăn cách các item trong một list. Và đó cũng chính là cách để chúng ta giải quyết bài toán ở trên. Mình sẽ giải thích method này hoạt động như thế nào: với từng item, method này sẽ được gọi nhằm mục đích setup khoảng ngăn cách cho mỗi item (tương tự như `padding` hay `margin`). Đại diện cho mỗi item ở đây là `outRect` với mỗi chiều của `outRect` sẽ là khoảng cách theo các chiều của item (left, top, right, bottom) và đơn vị của các chiều ở đây là px. Mặc định, khi không được `override` lại, 4 chiều giá trị của `outRect` sẽ được set bằng 0\. Bởi vậy, để ngăn cách từng item, chúng ta phải dựa vào vị trí của từng item mà set các chiều tương ứng. Nếu bạn cần xác định vị trí của item đó trong adapter, chúng ta có thể gọi `getChildAdapterPosition(View)`.

Và cuối cùng, sau khi implement, ta sẽ thêm `ItemDecoration` cho `RecyclerView`:

```
  rv.addItemDecoration(GridSpacingDecoration(2, (layoutManager as GridLayoutManager).spanCount))
```

Run thử phát cho nó lung linh:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/n7lt8tnld7_Screenshot_2018-01-22-13-25-43.png)

# Divider

Để có cách divider ngăn cách các item, bạn thường làm thế nào?

```
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="56dp"
      android:orientation="vertical"
      android:paddingEnd="0dp"
      android:paddingStart="16dp">

      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="55dp"
          android:gravity="center_vertical">

          <ImageView
              android:layout_width="44dp"
              android:layout_height="44dp"
              tools:src="@tools:sample/avatars" />

          <TextView
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:layout_marginStart="8dp"
              tools:text="@tools:sample/full_names" />
      </LinearLayout>

      <View
          android:layout_width="match_parent"
          android:layout_height="1dp"
          android:background="@android:color/darker_gray" />

  </LinearLayout>
```

Các dev nhà mình thường sẽ thêm ngay một `View` vào dưới cùng của layout, set độ cao, màu... Và tất nhiên, cách làm ngắn gọn này sẽ có những tác dụng phụ. Một tác dụng phụ dễ thấy nhất là sẽ tăng thêm độ sâu cho layout (_layout hierarchy_, trong trường hợp này là 2 thay vì 1), đồng thời làm tăng số lượng `View` lên một cách không cần thiết. Ngoài ra, bạn sẽ khó control các divider hơn nếu chúng là một phần của từng `View` (bạn phải dựa vào position, data để setup divider tùy theo), và điều đó hoàn toàn không flexible chút nào. And `ItemDecoration` to the rescue!!!

Ở đây, bạn cần tìm hiểu tiếp một method nữa đã được đề cập đến ở trên: `onDraw`. Method này dùng để vẽ những gì bạn muốn decor cho `RecyclerView`. Tuy nhiên hãy lưu ý rằng, đoạn vẽ này sẽ được vẽ trước khi `RecyclerView` vẽ các item trong list. Nếu muốn vẽ lên trên, hãy kiên nhẫn đọc tiếp (hoặc bỏ qua phần này kéo đến) [phần sau]("").

Chúng ta sẽ tạo một class kế thừa từ `ItemDecoration` như phần trên:

```
  class SeparatorDecoration (context: Context, orientation: Int) : RecyclerView.ItemDecoration() {

    companion object {
        const val HORIZONTAL = LinearLayout.HORIZONTAL
        const val VERTICAL = LinearLayout.VERTICAL
    }

    private var mDivider: Drawable? = null

    private var mOrientation: Int = LinearLayout.VERTICAL

    fun setOrientation(orientation: Int) {
        mOrientation = orientation
    }

    fun setDrawable(drawable: Drawable?) {
        mDivider = drawable
    }

    override fun onDraw(c: Canvas, parent: RecyclerView, state: RecyclerView.State?) {
        //todo
    }

    override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State?) {
        //todo
    }
  }
```

Ở đây chúng ta có 2 method cần phải implement: `getItemOffsets` để ngăn cách các item ra một khoảng đúng bằng độ cao của divider, và `onDraw` sẽ vẽ divider tương ứng. Mình sẽ implement cho `LinearLayoutManager` vertical, phần horizontal cũng sẽ là tương tự.

```
  override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State?) {
      if (mOrientation == VERTICAL) {
          outRect.set(0, 0, 0, mDivider!!.intrinsicHeight)
      } else { //HORIZONTAL
          //todo
      }
  }
```

Và với `onDraw`:

```
  override fun onDraw(c: Canvas, parent: RecyclerView, state: RecyclerView.State?) {
      if (mOrientation == VERTICAL) {
          drawVertical(c, parent)
      } else { //HORIZONTAL
          //todo
      }
  }

  private fun drawVertical(canvas: Canvas, parent: RecyclerView) {
      val bounds = Rect()

      canvas.save()
      var left: Int
      var right: Int

      val paddingRect = mDivider!!.getPadding()
      val paddingLeftDrawable = paddingRect.left
      val paddingRightDrawable = paddingRect.right

      if (parent.clipToPadding) {
          left = parent.paddingLeft
          right = parent.width - parent.paddingRight
          canvas.clipRect(left, parent.paddingTop, right, parent.height - parent.paddingBottom)
      } else {
          left = 0
          right = parent.width
      }

      left += paddingLeftDrawable
      right -= paddingRightDrawable

      val childCount = parent.childCount
      for (i in 0 until childCount) {
          val child = parent.getChildAt(i)
          val bottom = bounds.bottom
          val top = bottom - mDivider!!.intrinsicHeight
          mDivider!!.setBounds(left, top, right, bottom)
          mDivider!!.draw(canvas)
      }
      canvas.restore()
  }
```

Nhìn có vẻ hơi lằng nhằng, nhưng ở đây, việc cần làm là: lấy padding của drawable divider, padding của `RecyclerView`, margin của từng item. Sau đó tính toán dài rộng của divider. Và cuối cùng, gọi thằng canvas vẽ là xong. Khi đã hiểu cách tạo ra divider bằng `ItemDecoration`, chúng ta có thể sử dụng một class được Google cung cấp sẵn: `DividerItemDecoration` - bạn cần truyền vào orientation và drawable:

```
  rvOption.apply {
      val divider = DividerItemDecoration(context, SeparatorDecoration.VERTICAL).apply {
          setDrawable(ContextCompat.getDrawable(context, R.drawable.divider))
      }
      addItemDecoration(divider)
  }
```

Bạn có thể nghĩ thầm: "Sao không phang cha `DividerItemDecoration` ngay từ đầu có phải nhanh không~~". Tuy nhiên, việc bạn hiểu một lib hoạt động như thế nào hoặc hơn nữa là cách để tạo ra lib đó sẽ tốt hơn cho kiến thức của bạn rất nhiều. Hãy là một lập trình viên chịu tìm hiểu, đừng là một kẻ ăn sẵn.

# StickyHeaderRecyclerView

Phù, vậy là mình đã đi qua 2 method đầu tiên, đây là phần mình sẽ nói về method còn lại: `onDrawOver`. Thực ra, method này chỉ khác method `onDraw` một xíu: đó là method này được gọi sau khi các item được vẽ, bởi vậy những thứ bạn vẽ thông qua method này sẽ nằm phía trên các item, thay vì nằm phía dưới như method `onDraw` (được vẽ trước khi các item được vẽ). Được rồi, chúng ta sẽ tận dụng điều này để implement một function mà các dev thường phải sử dụng một lib bên ngoài. Đó là _StickyHeaderRecyclerView_.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/hph1m2ku5a_stickyheader.gif)

Chúng ta sẽ phân tích một chút những việc sẽ làm: chúng ta sẽ vẽ một header, và nội dung của header (hoặc design của header sẽ tùy thuộc vào vị trí của item). Bởi vậy, ta sẽ để người dùng trả về những giá trị sau:
- Position của header của một item bấy kỳ là gì?
- Layout của header để có thể inflate và draw
- Không phải trả về, nhưng người dùng cần bindData cho từng header tương ứng
- Liệu item này có phải là header hay không, phục vụ việc dịch chuyển header trước khi gặp một header mới

Từ đó, ta sẽ tạo một interface để người dùng thực hiện những điều trên

  ```
   interface StickyHeaderInterface {

        fun getHeaderPositionForItem(itemPosition: Int): Int

        fun getHeaderLayout(headerPosition: Int): Int

        fun bindHeaderData(header: TextView, headerPosition: Int)

        fun isHeader(itemPosition: Int): Boolean
   }
  ```

  Tiếp theo, phần cốt yếu, để header mới đẩy header cũ lên trên khi 2 header tiếp xúc với nhau là ban đầu khi 2 header chưa tiếp xúc với nhau, header đang được stick sẽ được vẽ ở trên cùng của `RecyclerView`. Đến khi 2 header tiếp xúc với nhau, header cũ cần được vẽ lùi lên phía trên tùy theo vị trí của header mới, khi đó header cũ sẽ khuất dần cho đến khi header mới chiếm hoàn toàn vị trí của header cũ. Cứ như vậy, ta sẽ có cảm giác header được stick lên phía trên của `RecyclerView`.

  Như vậy, ta cần 2 method để vẽ trong 2 trường hợp:

  - Trường hợp 2 header không tiếp xúc với nhau, ta vẽ header được stick lên trên cùng của `RecyclerView`

```
  private fun drawHeader(c: Canvas, header: View) {
      c.save()
      c.translate(0F, 0F)
      header.draw(c)
      c.restore()
  }
```

- Trường hợp 2 header tiếp xúc với nhau, ta sẽ vẽ header cũ lùi lên trên, lên trên bao nhiêu sẽ tùy vào header tiếp theo:

```
  private fun moveHeader(c: Canvas, currentHeader: View, nextHeader: View?) {
      c.save()
      c.translate(0F, (nextHeader!!.top - currentHeader.height).toFloat())
      currentHeader.draw(c)
      c.restore()
  }
```

Thêm một điểm cần lưu ý nữa, ta cần có một method để xác định xem: liệu 2 header có tiếp xúc với nhau hay chưa:

```
  private fun getChildInContact(parent: RecyclerView, contactPoint: Int): View? {
      var childInContact: View? = null
      for (i in 0 until parent.childCount) {
          val child = parent.getChildAt(i)
          if (child.bottom > contactPoint && child.top <= contactPoint) {
              // This child overlaps the contactPoint
              childInContact = child
              break
          }
      }
      return childInContact
  }
```

`contactPoint` mà chúng ta cần truyền vào để kiểm tra kia, sẽ là tọa độ của điểm thấp nhất (`bottom`) của header. Ta sẽ duyệt tất cả child của `RecyclerView`, sau đó kiểm tra xem nếu `bottom` của child lớn hơn `contactPoint` và phần trên cùng (`top`) của child nhỏ hơn (đã đè lên) hoặc bằng (bắt đầu tiếp xúc) với `contactPoint`, khi đó ta sẽ trả về view mà tiếp xúc với `contactPoint` kia. Vậy câu hỏi được đặt ra khi nhìn thấy kiểu giá trị trả về của method này là: liệu có bao giờ, không có child view nào tiếp xúc với `contactPoint` không? Nghĩ đơn giản một chút, thì lúc nào mà chả có 1 item nào đấy đang contact với header cũ đúng không? Tuy nhiên, trước khi trả lời câu hỏi đấy, bạn cần trả lời một câu hỏi khác: khi bạn ngăn cách 2 item trong list một khoảng nào đó, bạn có thể sử dụng dù là margin hay override lại `getItemOffsets` của `ItemDecoration`, phần ngăn cách đấy có được tính là thuộc về item không? Nếu như đã đọc kỹ phần 1, câu trả lời là không (sử dụng margin, câu trả lời cũng tương tự). Vậy, đáp án của câu hỏi _trên trên một tí_ là có - sẽ có lúc, không có item nào tiếp xúc với header đang được stick. Và khi không tiếp xúc, chúng ta sẽ vẽ header như bình thường mà không cần phải quan tâm đến vị trí của header làm gì cả.

Cuối cùng, khi đã tìm được cách giải quyết của các vấn đề, ta sẽ implement method `onDrawOver`:

```
  override fun onDrawOver(c: Canvas, parent: RecyclerView, state: RecyclerView.State) {
      super.onDrawOver(c, parent, state)

      val topChild = parent.getChildAt(0) ?: return

      val topChildPosition = parent.getChildAdapterPosition(topChild)
      if (topChildPosition == RecyclerView.NO_POSITION) {
          return
      }

      getHeaderViewForItem(topChildPosition, parent)
      val contactPoint = headerView!!.bottom
      val childInContact = getChildInContact(parent, contactPoint)

      if (childInContact != null && mListener.isHeader(parent.getChildAdapterPosition(childInContact))) {
          moveHeader(c, headerView!!, childInContact)
          return
      }

      drawHeader(c, headerView!!)
  }
```

Ở đây, method `getHeaderViewForItem` sẽ làm nhiệm vụ inflate view cho header + bindData vào header đó.

# Túm cái váy lại

> Mọi con đường đều dẫn đến thành Rome

Tuy nhiên, con đường nào mà ít phải chịu hiểm nguy nhất là do bạn chọn. Khi còn là một kẻ học việc, bạn hoàn toàn có thể sử dụng những con đường tắt nhưng cũng đầy nguy cơ rình rập để đạt được kết quả như ý muốn. Nhưng một khi đã đạt được điều mình mong muốn, hãy quay trở lại, tìm tòi và chọn lại cho mình con đường đúng đắn nhất (và có thể là công cụ đúng đắn nhất), dù đó không phải là con đường ngắn nhất. Và khi đó, bạn sẽ có những bước đi an toàn và chắc chắn nhất.

Với `ItemDecoration` cũng vậy, nó thực sự là _a right tool_ để decor `RecyclerView` chứ không phải dùng những mẹo như các dev vẫn hay dùng. Ngoài ra, bạn có thể sử dụng nhiều `ItemDecoration` đối với một `RecyclerView`. Giới hạn bây giờ chỉ là ở khả năng sáng tạo của bạn mà thôi!

Các bạn có thể tham khảo [source code](https://github.com/srinnix1395/StickyHeaderRecylerView)
