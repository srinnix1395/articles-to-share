# Control flow statements

Các câu lệnh điều khiển trong *Dart* bao gồm:
* `if` và `else`
* Vòng lặp `for`
* Vòng lặp `while` và `do-while`
* `break` và `continue`
* `switch` và `case`
* `assert`
* `try-catch` và `throw`

### `if` và `else`

Với câu lệnh `if`, điều kiện bất buộc phải có kiểu `bool`
```
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

### Vòng lặp `for`
