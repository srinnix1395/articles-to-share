### Collections vs Sequences
* Collections is eagerly evaluated & Sequences is lazily evaluated
* Collections
  - Các operation được thực hiện ngay khi được gọi
  - Một operation sẽ apply với toàn bộ phần tử và tạo ra một collection mới. Tiếp đó, các operation sau sẽ apply tương tự
  - VD:
```
    val yellowSquareCollection = shapes.map {
        it.copy(color = 3)
    }.first {
        it.edges == 4
    }
```
  Thứ tự thực hiện là:
    + `map` được apply với toàn bộ phần tử và tạo ra một collection mới
    + `first` được apply cho đến khi tìm thấy item thỏa mãn điều kiện

* Sequences
  - Có 2 loại operation:
    + intermediate: `map`, `distinct`, `filter`
    + terminal: `first`, `toList`, `count`
  - Intermediate operation không được thực hiện ngay mà chỉ được ghi nhớ
  - Khi terminal operation được gọi, từng intermediate operation mới thực hiện với từng phần tử và sau đó là terminal operation được gọi ngay sau đó.
  - VD:
```
    val yellowSquareSequence = shapes.asSequence().map {
        it.copy(color = 3)
    }.first {
        it.edges == 4
    }
```
  Thứ tự thực hiện là:
    + `asSequence` tạo ra một sequence từ collection ban đầu
    + `map` được gọi nhưng không thực hiện ngay lập tức mà được lưu vào một list các intermediate operation cần thực hiện
    + `first` được gọi. Vì đây là một terminal operation nên các intermediate operation sẽ được trigger để thực hiện với từng phần tử. Nhưng thay vì cần thực hiện `map` với toàn bộ phần tử, `map` được thực hiện với phần tử đầu tiên, ngay sau đó là `first` với phần tử đầu tiên, tiếp tục như vậy cho đến khi tìm được phần tử thỏa mãn điều kiện. Các item sau item thỏa mãn điều kiện sẽ không cần phải xử lý với `map` và `first`

![alt text](https://miro.medium.com/max/700/0*hdSVv06ug45AzTTr)

### Thứ tự transformation

Cho dù bạn sử dụng Collection hay Sequence, thứ tự các operation cũng là một vấn đề. VD:

![alt text](https://miro.medium.com/max/700/0*F4tikgy89TZZnUWP)

Với cả Collection và Sequence, `first` tìm phần tử thỏa mãn trước, sau đó mới apply `map` nên chỉ cần apply `map` với 1 phần tử duy nhất. Bởi vậy, hãy để ý đến thứ tự thực hiện các operation, n cũng sẽ giúp performance được cải thiện một chút.

### Cùng nhìn lại
- Collection is eagerly evaluated và phù hợp với list ít phần tử
- Sequence is lazily evaluated và phù hợp với list nhiều phần tử.
- Hãy chú ý đến thứ tự của các operation nữa.
