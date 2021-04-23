[Android] Constraint layout - Phần 1: Thiết kế

Khi xây dựng một màn hình có dạng một list các item trong Android, ta sẽ phải thiết kế layout cho các item của list đó, list đó có thể như thế này chẳng hạn:

<div style="text-align:center"><img src ="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/lskfhol77v_components_lists_keylines_single9.png" /></div>

Với item hiển thị contact như trong hình trên, *ViewGroup* đầu tiên xuất hiện trong đầu bạn để thiết kế được layout của item là gì?

- *LinearLayout* rồi, còn dùng cái gì nữa!?!.
- *RelativeLayout* cũng được, tùy

Chắc chắn 2 loại *ViewGroup* này sẽ là 2 loại phổ biến nhất và cũng là 2 loại đầu tiên bạn có thể nghĩ đến. OK. Tiếp:

<div style="text-align:center"><img src ="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/c75dmv1beh_components_lists_keylines_two11.png" /></div>

Uhm, có vẻ phức tạp lên một chút rồi đấy? Ah mà kết hợp *RelativeLayout* với *LinearLayout* là được chứ gì.

Với một layout đơn giản như thế này, bạn đang sử dụng một layout với độ sâu là 2 tầng (một *RelativeLayout* và một *LinearLayout* ở bên trong). Tuy vậy, với các layout phức tạp hơn rất nhiều, việc thiết kế một layout có độ sâu quá lớn sẽ ảnh hưởng rất nhiều đến [performance](https://developer.android.com/topic/performance/rendering/optimizing-view-hierarchies.html) trong quá trình tính toán và vẽ các thành phần lên màn hình. Giải thích nôm na là vì mỗi lần bạn vẽ một *ViewGroup*, Android sẽ duyệt tất cả các thằng con của *ViewGroup* để tính toán và sắp xếp các view con. Nếu *ViewGroup* càng sâu, nỗi đau sẽ càng dai dẳng :v. Bởi vậy, tại Google I/O 2016, Google đã giới thiệu một loại *ViewGroup* mới: *ConstraintLayout*, giúp cho việc giảm thiểu độ sâu của các layout được thiết kế, cùng với đó là giúp cho việc thiết kế bằng kéo thả (design editor) được dễ dàng hơn (mình vẫn thích code tay hơn là kéo thả ~~)

<div style="text-align:center"><img src ="https://i.ytimg.com/vi/z53Ed0ddxgM/maxresdefault.jpg" /></div>

Với *ConstraintLayout*, ta có thể làm được những việc *RelativeLayout* và *LinearLayout* làm được và cả những việc 2 loại layout trên không làm được. Tò mò chứ? Băt đầu thôi, không thể thời gian bay đi nữa ~~

# Vị trí tương đối

Dựa vào tên, ta có thể đoán ra được việc sắp xếp các *View* con bên trong *ConstraintLayout* sẽ dựa vào các ràng buộc (constraint), tức là *View* này ràng buộc vào *View* kia mà sắp xếp. Nghĩ một chút thì cũng có vẻ giống giống *RelativeLayout* khi các *View* con sẽ được sắp xếp dựa theo vị trí tương đối so với các *View* khác. Tuy nhiên, *ConstraintLayout* còn làm được nhiều hơn thế. Đầu tiên, mình sẽ cho các bạn thấy *ConstraintLayout* giống *RelativeLayout* như thế nào:

Trong *RelativeLayout*, để căn cho 2 cạnh bên cùng một phía của 2 *View* được thẳng hàng với nhau, ta sử dụng các thuộc tính *layout_align\**. Để làm được điều tương tự với *ConstraintLayout*, ta sử dụng các thuộc tính để neo một phía của *View* này với phía tương tự của *View* kia:

```
    //RelativeLayout
    android:layout_alignStart="@id/view"
    android:layout_alignLeft="@id/view"
    android:layout_alignEnd="@id/view"
    android:layout_alignRight="@id/view"
    android:layout_alignTop="@id/view"
    android:layout_alignBaseline="@id/view"
    android:layout_alignBottom="@id/view"

    //ConstraintLayout
    app:layout_constraintStart_toStartOf="@id/view"
    app:layout_constraintLeft_toLeftOf="@id/view"
    app:layout_constraintEnd_toEndOf="@id/view"
    app:layout_constraintRight_toRightOf="@id/view"
    app:layout_constraintTop_toTopOf="@id/view"
    app:layout_constraintBaseline_toBaselineOf="@id/view"
    app:layout_constraintBottom_toBottomOf="@id/view"
```

Để thao tác nhanh bằng kéo thả, ta có thể làm như sau:

<div style="text-align:center"><img src ="http://g.recordit.co/ihySq89Whi.gif" /></div>

hoặc:

<div style="text-align:center"><img src ="http://g.recordit.co/rRYfeMJy35.gif" /></div>

Nếu muốn 2 *View* được sắp xếp bên cạnh nhau, ta sử dụng các thuộc tính *layout_to\*Of* đối với *RelativeLayout* và với *ConstraintLayout* là các thuộc tính để neo một cạnh của *View* này vào cạnh đối diện của *View* kia:

```
    //RelativeLayout
    android:layout_toStartOf="@id/view"
    android:layout_toLeftOf="@id/view"
    android:layout_toEndOf="@id/view"
    android:layout_toRightOf="@id/view"
    android:layout_above="@id/view"
    android:layout_below="@id/view"

    //ConstraintLayout
    app:layout_constraintStart_toEndOf="@id/view"
    app:layout_constraintLeft_toRightOf="@id/view"
    app:layout_constraintEnd_toStartOf="@id/view"
    app:layout_constraintRight_toLeftOf="@id/view"
    app:layout_constraintTop_toBottomOf="@id/view"
    app:layout_constraintBottom_toTopOf="@id/view"
```

Với phiên bản kéo thả, nó sẽ như thế này:

<div style="text-align:center"><img src ="http://g.recordit.co/CCpK4bWQOx.gif" /></div>

Tiếp theo, để neo được các *View* vào  các cạnh trên, dưới, trái hoặc phải của *ViewGroup* cha, ta sử dụng các thuộc tính *layout_alignParent\** đối với *RelativeLayout*. Còn với *ConstraintLayout*, layout này cung cấp một giá trị là `parent`, khi sử dụng các giá trị này với các thuộc tính ràng buộc đã nói ở trên, ta sẽ đạt được kết quả như với *RelativeLayout*

```
    //RelativeLayout
    android:layout_alignParentStart="true"
    android:layout_alignParentLeft="true"
    android:layout_alignParentEnd="true"
    android:layout_alignParentRight="true"
    android:layout_alignParentTop="true"
    android:layout_alignParentBottom="true"

    //ConstraintLayout
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
```

Phiên bản kéo thả:

<div style="text-align:center"><img src ="http://g.recordit.co/zmFVFy549d.gif" /></div>

Với các thuộc tính ở trên, ta chỉ có thể neo 1 cạnh của *View* vào một cạnh nào đó. Vậy, nếu ta neo nhiều cạnh của một *View* vào nhiều cạnh của các *View* khác. Điều gì sẽ xảy ra?

```
<TextView
    style="@style/TextAppearance.AppCompat.Large"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="hello world"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

<div style="text-align:center"><img src ="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/vlsycrsiag_Screenshot_20170826-121805.png" /></div>

Phiên bản kéo thả:

<div style="text-align:center"><img src ="http://g.recordit.co/0Sq5xlnPyC.gif" /></div>

Ô, vậy là giống với `layout_centerInParent`, `layout_centerHorizontal` và `layout_centerVertical` của *RelativeLayout* ah ~~. Yup, để các *View* được căn vào giữa của 2 cạnh nào đó, ta chỉ cần neo 2 cạnh (hoặc 4 cạnh) của *View* đó vào 2 cạnh (hoặc 4 cạnh) là được. Và đó chưa phải là điều hay nhất.

Ngoài ra, khi neo 2 cạnh đối diện của một view vào cùng một điểm neo, view đó sẽ được căn vào chính giữa của điểm neo đó:

```
<Button
    android:id="@+id/btn_visible"
    android:layout_width="72dp"
    android:layout_height="32dp"
    android:layout_marginStart="16dp"
    android:background="@color/colorAccent"
    app:layout_constraintStart_toStartOf="parent"/>

<Button
    android:id="@+id/btn_delete"
    android:layout_width="72dp"
    android:layout_height="32dp"
    android:layout_marginTop="8dp"
    android:background="@color/colorAccent"
    app:layout_constraintEnd_toEndOf="@id/btn_visible"
    app:layout_constraintStart_toEndOf="@id/btn_visible"
    app:layout_constraintTop_toBottomOf="@+id/btn_visible"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/XO5ueN5_T9y8kxdcnELYeg.png" /></div>

Thế nếu bạn không muốn nó vào giữa, mà là nằm ở vị trí 30% hoặc 70% chiều rộng của *ViewGroup* cha thì sao. *RelativeLayout* thì làm thế nào được ~~. Thế mà, *ConstraintLayout* lại cung cấp một thuộc tính cho phép đạt được điều này: *bias* - có thể hiểu là bạn muốn sắp xếp *View* này thiên về bên nào hơn. Giá trị của *bias* nằm trong khoảng từ 0 đến 1 và có kiểu *Float*. Thuộc tính này chỉ có tác dụng khi *View* đang neo 2 cạnh đối diện hoặc cả 4 cạnh. *ConstraintLayout* cung cấp thuộc tính này cho cả chiều ngang (horizontal) và chiều dọc (vertical). Và nếu khi đã neo 2 cạnh hoặc 4 cạnh, nếu không có giá trị *bias* nào được chỉ định, *bias* sẽ có giá trị mặc định là `0.5`, tức là vào giữa của 2 hoặc 4 điểm neo. VD:

```
<TextView
    style="@style/TextAppearance.AppCompat.Large"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="hello world"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintHorizontal_bias="0.3"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintVertical_bias="0.7"/>
```

<div style="text-align:center"><img src ="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/5qlf0wo8u1_Screenshot_20170826-123319.png" /></div>

Bằng kéo thả:

<div style="text-align:center"><img src ="http://g.recordit.co/p2J72PhCWI.gif" /></div>

Thêm một lưu ý: giá trị `match_parent` khi xác định width và height của một *View* sẽ không còn được hỗ trợ với *ConstraintLayout* nữa. Thay vào đó, *ConstraintLayout* giới thiệu một giá trị khác: `match_constraint` cũng có mục đích tương tự với `match_parent`. Ta sẽ sử dụng `match_constraint` bằng cách set `layout_width` hoặc `layout_height` bằng `0dp` và neo 2 cạnh đối diện của *View* vào 2 bên tương ứng để width/ height của *View* tràn ra và đạt được hiệu ứng như `match_parent`.

# Tỷ lệ giữa width và height (constraint dimension ratio)

Để hiển thị một list các item dưới dạng grid, như là hiển thị một list hình ảnh như Instagram, bạn sẽ thiết kế từng item thế nào:

<div style="text-align:center"><img src ="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/u27mw8dxvc_image.png" /></div>

Uhm, có thể custom lại *View*: trong *onMeasure* tính toán cho height bằng width là được,... Tuy nhiên, với một developer beginner như mình (thiết kế layout còn chả xong nữa là custom view ~~), thì việc chỉ cần sử dụng thuộc tính `layout_constraintDimensionRatio` của *ConstraintLayout* để thiết kế được item hình vuông đó thực sự dễ như một miếng bánh vậy ^^:

```
<ImageView
    android:id="@+id/textView"
    android:layout_width="100dp"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintDimensionRatio="1:1"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/Z-hpWWMlRwOpyHcm0ms65Q.png" /></div>

Giá trị của thuộc tính này là tỷ lệ giữa width và height của *View*. Và để sử dụng thuộc tính này, giá trị của *layout_width* hoặc *layout_height* hoặc cả 2 phải là `match_constraint`. Trong trường hợp cả 2 thuộc tính đều là `match_constraint`, Android sẽ lấy giá trị lớn nhất của width hoặc height thỏa mãn được việc sắp xếp trong *ViewGroup* cha mà vẫn giữ được tỷ lệ giữa width và height đã khai báo.

```
<ImageView
    android:id="@+id/textView"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:src="@color/colorAccent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintDimensionRatio="1:1"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/e6ru9cE8Sy_NE25iVfAGiQ.png" /></div>

Và bằng kéo thả:

<div style="text-align:center"><img src ="http://g.recordit.co/ffXkF0VzIW.gif" /></div>

Ngoài ra, với *ratio* chúng ta cũng có thể xác định rõ xem width phải ràng buộc theo height hay height phải ràng buộc theo width bằng cách thêm `W` hoặc `H` vào trước tỷ lệ.

```
<ImageView
    android:id="@+id/textView"
    android:layout_width="90dp"
    android:layout_height="0dp"
    android:src="@color/colorAccent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintDimensionRatio="W, 4:3"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/2r-ixdUhSCaMAdbiMniFZA.png" /></div>

Trong trường hợp ở trên, width ở đây phải ràng buộc theo height. Tuy nhiên, vì width đã được set một giá trị cố định là 90dp nên Android sẽ dãn height ra để tỷ lệ 4:3 vẫn được thỏa mãn. Bởi vậy, thực tế là tỷ lệ giữa width và height không còn là 4:3 nữa mà sẽ là 3:4 tức là height = 120dp

# Chuỗi (Chain)

Cơ chế của *LinearLayout* cho phép chúng ta có thể sắp xếp các *View* lần lượt cạnh nhau từ trái sang phải hoặc từ trên xuống dưới. Trong *ConstraintLayout*, ta cũng có thể làm điều này bằng cách liên kết các *View* đó thành một nhóm thông qua việc sử dụng *chain*.

Để tạo một *chain*, ta cần kết nối các *View* với nhau theo cả 2 hướng (**bi-directional connection**) và phải là từ 2 hướng. 2 hướng ở đây có nghĩa là ví dụ đuôi của view 1 neo vào đầu của view 2 và đầu của view 2 cũng cần được neo vào đuôi của view 1. Nếu kết nối chỉ có từ 1 phía, Android sẽ không thể nhận ra được các *View* đó nằm trong cùng một *chain*

```
<Button
    android:id="@+id/label_a"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:text="A"
    android:textSize="22sp"
    app:layout_constraintEnd_toStartOf="@+id/label_b"
    app:layout_constraintHorizontal_chainStyle="spread"
    app:layout_constraintStart_toStartOf="parent"/>

<Button
    android:id="@+id/label_b"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:text="B"
    android:textSize="22sp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toEndOf="@id/label_a"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/GKoqxIErSU_CLDBz9fbyUw.png" /></div>

Để tạo một *chain* mà khoảng 10 view thì việc kết nối 10 view đó mà lại bằng code thì đúng là chán như con gián ~~. Trong trường hợp này thì kéo thả thực sự có một sức mạnh to "nớn" ^^

<div style="text-align:center"><img src ="http://g.recordit.co/Y2FP52XJlo.gif" /></div>

Đối với một *chain*, các thuộc tính của cả *chain* đó sẽ được thiết lập ở phần tử đầu tiên của *chain* (*head chain*): phần tử phía bên trái nhất đối với *horizontal chain* và phần tử trên cùng với *vertical chain*

Ở ví dụ ngay phía trên, ta có thể thấy một thuộc tính xác định style của *chain*, giá trị của thuộc tính đó sẽ được xác định ở phần tử *head chain* và có thể là 1 trong 4 kiểu sau:

**1. Spread**: Các *View* sẽ được phân bố một cách đều nhau và đây là style mặc định của một *chain*.
`layout_constraintHorizontal_chainStyle="spread"`
`layout_constraintVertical_chainStyle="spread"`

<div style="text-align:center"><img src ="https://image.prntscr.com/image/V_Aia1y9SlWRVsmRu7W8ig.png" /></div>

Nếu sử dụng *LinearLayout*, chắc chắn là sẽ có một chút gì đó vất vả hơn

**2. Spread inside**: 2 phần tử đầu tiên và cuối cùng sẽ được neo vào 2 điểm ràng buộc ở đầu và cuối của *chain*, các phần tử ở giữa còn lại được phân bố một cách đều nhau như với style *spread*.
`layout_constraintHorizontal_chainStyle="spread_inside"`
`layout_constraintVertical_chainStyle="spread_inside"`

<div style="text-align:center"><img src ="https://image.prntscr.com/image/3IBJc44iQIWLM4PWja3hPw.png" /></div>

**3. Weighted**: Khi style của *chain* được set là *spread* hoặc *spread inside*, các khoảng trống giữa các *View* sẽ có thể được chiếm bởi chính các *View* đó. Để làm được điều này, ta cần set `layout_width` hoặc `layout_height` của view muốn chiếm chỗ là `match_constraint`. Ngoài ra, khi các *View* đều đã chiếm hết các khoảng trống, ta có thể xác định xem *View* nào chiếm nhiều phần hơn *View* nào bằng cách sử dụng thuộc tính `layout_constraintHorizontal_weight` hoặc `layout_constraintVertical_weight`. Các thuộc tính này hoạt động giống với `layout_weight` trong *LinearLayout*: weight lớn hơn thì chiếm nhiều phần hơn, weight bằng nhau thì chiếm phần bằng nhau:

<div style="text-align:center"><img src ="https://image.prntscr.com/image/4UYRV3GsS1C2B2oWXveXCQ.png" /></div>

Trong trường hợp dưới đây, *TextView* C đâu r nhỉ?

```
<TextView
    android:id="@+id/label_a"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:gravity="center"
    app:layout_constraintHorizontal_weight="3"
    android:text="A"
    android:textSize="22sp"
    app:layout_constraintEnd_toStartOf="@+id/label_b"
    app:layout_constraintHorizontal_chainStyle="spread"
    app:layout_constraintStart_toStartOf="parent"/>

<TextView
    android:id="@+id/label_b"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:gravity="center"
    app:layout_constraintHorizontal_weight="2"
    android:text="B"
    android:textSize="22sp"
    app:layout_constraintEnd_toStartOf="@+id/label_c"
    app:layout_constraintStart_toEndOf="@id/label_a"/>

<TextView
    android:id="@+id/label_c"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:text="C"
    android:textSize="22sp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toEndOf="@id/label_b"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/Vuvh6_wHQqqo8iwnTpAbTQ.png" /></div>

Nhớ nhớ: nếu một phần tử C trong chuỗi `match_constraint` nhưng lại không được set weight và các phần tử còn lại đều được set weight hoặc `wrap_content`, phần tử C đó sẽ không được hiển thị lên.

**4. Packed**: Các *View* trong *chain* sẽ được "bó" vào nhau. Với style này, bạn có thể xác định *bias* cho cả *chain* bằng cách thay đổi giá trị *bias* ở phần tử  *head chain*
`layout_constraintHorizontal_chainStyle="packed"`
`layout_constraintVertical_chainStyle="packed"`

<div style="text-align:center"><img src ="https://image.prntscr.com/image/NS832nxpTPyvK4sd19P3wg.png" /></div>

Và với *packed chain* có *bias* bằng 0.2
`layout_constraintHorizontal_bias="0.2"`

<div style="text-align:center"><img src ="https://image.prntscr.com/image/jaJKASd8RZ2BoDysxJ0POA.png" /></div>

Bằng kéo thả, việc thay đổi chain style, căn trái, phải, trên, dưới, hoặc center cả *chain* cũng rất dễ dàng:

<div style="text-align:center"><img src ="http://g.recordit.co/IGG4xptHBP.gif" /></div>

## Margin

Trong *ConstraintLayout*, *margin* sẽ chỉ có tác dụng trong trường hợp *View* đã được neo vào đúng hướng với hướng muốn margin. Một điểm khác nữa với *margin* trong *ConstraintLayout* là *margin_ không được là số âm.

Khi một view được `GONE` đi, mọi giá trị size của *View* đều sẽ bằng `0dp` nhưng vẫn sẽ tham gia vào quá trình tính toán các ràng buộc. Cùng với đó, tất cả các margin đối với những ràng buộc của *View* này cũng sẽ được set bằng `0dp`. Trong trường hợp đó, layout sẽ ngay lập tức bị phá vỡ. Bởi vậy, *ConstraintLayout* đã cung cấp thêm một số thuộc tính mới dành cho *margin* để xác định *margin* trong trường hợp *View* được lấy làm mỏ để neo biến mất - `visibility="GONE"`. Lưu ý: giá trị *margin* này sẽ chỉ có tác dụng trong trường hợp *View* làm mỏ neo bị biến mất:

Trường hợp button `btn_visible` còn visible:
```
<Button
    android:id="@+id/btn_visible"
    android:layout_width="72dp"
    android:layout_height="32dp"
    android:layout_marginStart="16dp"
    android:background="@color/colorAccent"
    app:layout_constraintStart_toStartOf="parent"/>

<Button
    android:id="@+id/btn_delete"
    android:layout_width="72dp"
    android:layout_height="32dp"
    android:layout_marginStart="8dp"
    android:background="@color/colorAccent"
    app:layout_constraintStart_toEndOf="@id/btn_visible"
    app:layout_goneMarginStart="98dp"/>
```

<div style="text-align:center"><img src ="https://image.prntscr.com/image/-ebXQlHJTuqtdFX_btbI_Q.png" /></div>

Và khi button `btn_visible` GONE đi - `visibility="gone"`, button `btn_delete` vẫn ở nguyên vị trí cũ bởi đã được margin start 98dp bằng chính độ rộng của button `btn_visible` và margin start của button `btn_visible`

<div style="text-align:center"><img src ="https://image.prntscr.com/image/KaPro50JTxurEdq73FWS-Q.png" /></div>

Bằng kéo thả cũng rất lẹ:

<div style="text-align:center"><img src ="http://g.recordit.co/EosYEVTW18.gif" /></div>

## Guideline

Android cũng cung cấp thêm một *Widget* mới là *Guideline* (có dạng như một đường thẳng) đóng vai trò như một điểm để neo các *View* khác vào. Vì chỉ nhằm mục đích thiết kế, *Guideline* sẽ chỉ hiển thị trên bản thiết kế (blueprint) hoặc preview editor chứ không hiển thị khi chạy ứng dụng

<div style="text-align:center"><img src ="https://s3-ap-southeast-1.amazonaws.com/kipalog.com/tdyfyttlx8_image.png" /></div>

```
<android.support.constraint.Guideline
    android:id="@+id/guideline1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    app:layout_constraintGuide_begin="80dp"/>

<android.support.constraint.Guideline
    android:id="@+id/guideline2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    app:layout_constraintGuide_percent="0.5"/>
```

*Guideline* chỉ có 2 thuộc tính bạn cần chú ý đến: một là hướng của *Guideline*: `horizontal` hay `vertical`. Thứ hai là vị trí của *Guideline* trên bản thiết kế: ở một vị trí xác định `layout_constraintGuide_begin` hoặc ở một vị trí tương đối `layout_constraintGuide_percent`. Vì đây chỉ là một *Widget* có tác dụng hỗ trợ việc thiết kế chứ không được hiển thị lên, bạn sẽ không thể neo được *Guideline* vào đâu cả.

Kéo thả lần cuối cùng ~~:

<div style="text-align:center"><img src ="http://g.recordit.co/F1MbE9YpbW.gif" /></div>

* * *
Phù ~~, nếu đã đọc đến đây (hoặc chỉ kéo xuống dưới và vô tình thấy dòng này), thì bạn thực sự là một người nhẫn nại (hoặc là một tên lười đọc). Sau một đống lý thuyết loằng ngoằng, liệu bạn đã tìm ra cách để thiết kế layout ở đầu bài viết mà chỉ cần dùng 1 *ViewGroup* duy nhất hay chưa? Nếu vẫn còn mông lung như một trò đùa, thì em xin dang tay rút lui thôi :v Đùa đấy, một chút gợi ý [ở đây]() nhớ.

Và cuối cùng, hãy nghiền ngẫm thêm về *ConstraintLayout* và đợi chờ phần 2 - một phần chắc chắn sẽ đẹp mắt và thú vị hơn nhớ ^^

## Tham khảo:

- [Constraint layout](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html)
- [Build a Responsive UI with ConstraintLayout](https://developer.android.com/training/constraint-layout/index.html#adjust-the-view-margins)
- [Guide to constraint layout](https://medium.com/@loutry/guide-to-constraintlayout-407cd87bc013)
- [Constraint layout chains](https://medium.com/@nomanr/constraintlayout-chains-4f3b58ea15bb)
