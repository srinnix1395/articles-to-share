1. Định nghĩa
- Định nghĩa cách activity được xử lý so với task stack. Có 2 cách để apply
  + manifest: "launchMode"
  + add/set flag cho Intent
note: nếu cả 2 cùng được set, flag của intent sẽ được ưu tiên hơn.

2. Giải thích
* Apply bằng manifest:
  - standard: Default khi ta không xác định giá trị cho launchMode
  - singleTop: Nếu activity muốn được tạo chính là activity on top, thay vì một instance mới được khởi tạo, activity cũ sẽ được sử dụng và intent sẽ được truyền qua onNewIntent
  - singleTask: Nếu activity đó chưa được tạo, một instance mới sẽ được tạo và chạy trên chính task gọi intent(nếu không chỉ định taskAffinity) và khởi tạo trên task được chỉ định = taskAffinity. Nếu trong toàn bộ system, activity đó đã tồn tại, sẽ dùng lại instance cũ và truyền intent thông qua onNewIntent(toàn bộ activity nằm phía trên activity đó trong stack sẽ bị kill)
  - singleInstance: Tương tự như singleTask nhưng activity được chỉ định là member duy nhất trong task đó.
* Apply thông qua Intent
  - FLAG_ACTIVITY_NEW_TASK: ~ singleTask
  - FLAG_ACTIVITY_SINGLE_TOP: ~ singleTop
  - FLAG_ACTIVITY_CLEAR_TOP: nếu activity đã tồn tại nhưng k on top thì toàn bộ các activity ở trên trong task stack sẽ bị kill
