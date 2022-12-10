---
title: You probably using setTimeout wrong
author: huytd
date: 12-10-2022
---
Có lẽ vì cái tên `setTimeout`, nên rất nhiều người sử dụng hàm `setTimeout` với mục đích thực thi một hàm callback sau một khoảng thời gian timeout nhất định. Nhưng đây không phải là cách mà `setTimeout` hoạt động.

Để mà nói cho đúng, thì chúng ta nên đổi tên hàm `setTimeout` thành `setMiniumTimeout` hoặc `runAfterAMinimumTimeout`. Vì mục đích của hàm này là schedule một hàm callback để nó được thực thi sau khi chờ ít nhất một khoảng thời gian nhất định, và không có gì đảm bảo rằng, khi bạn gọi `setTimeout(fn, 100)` thì đúng 100ms sau, hàm `fn` sẽ được gọi, nhưng có một điều chắc chắn là, hàm `fn` sẽ không bao giờ được gọi khi chúng ta chưa chờ đủ 100ms.

Để hiểu rõ hơn, thì chúng ta hãy xem lại cách mà JS thực thi code.

Chắc bạn cũng từng nghe đến khái niệm callstack rồi. Nó là một cái stack LIFO (last in first out, vào sau ra trước) mà JS engine dùng để push vào thông tin của một hàm khi hàm đó được gọi, và khi hàm đó thực thi xong thì sẽ bị pop ra khỏi callstack.

Và chắc các bạn cũng từng nghe nói đến chuyện, JS thực thi code trên một thread duy nhất (single threaded), thread này thường được gọi là main thread. Khi một đoạn code được thực thi, thì những đoạn code đằng sau nó phải chờ. Thật ra không phải mọi thứ trong JS thì đều phải được thực thi trên main thread. Những chức năng như browser's API (`setTimeout`, `setInterval`, event handlers, `fetch`,...) đều có thể được thực thi trên một thread riêng lẽ, quản lý bởi browser, và kết quả sẽ được trả về main thread thông qua một hàm callback.

Để thực thi mỗi một hàm callback như này, thì browser sẽ tạo ra một object gọi là Task, chứa thông tin của hàm callback cần thực thi. Và trong quá trình hoạt động thì sẽ có rất nhiều Task được tạo ra (ví dụ nhiều event handler được kích hoạt, nhiều `setTimeout` hoàn thành), các Task này sẽ được lưu vào một nơi gọi là Task Queue.

Khi tất cả mọi function call đã hoàn thành và callstack về trạng thái trống (empty), thì browser sẽ bắt đầu thực thi các Task có trong Task Queue.

Điều này có nghĩa là, nếu một hàm callback được queued, nhưng main thread vẫn đang bận xử lý những function calls khác, thì hàm callback đó phải chờ cho đến khi không còn cái gì ở trong callstack nữa. Đây cũng là nguyên nhân vì sao khi có một tác vụ gì đó tốn tài nguyên đang chạy, thì người dùng không thể click hay scroll gì được.

Quay trở lại với câu lệnh `setTimeout(fn, 100)`, đến lúc này chắc các bạn cũng đã hình dung ra, sau khi hết thời gian timeout là 100ms, một Task mới sẽ được tạo ra để báo cho browser biết rằng hàm callback `fn` cần được thực hiện. Task này sẽ được push vào Task Queue và... chờ tiếp. Nếu trong lúc này, browser vẫn đang bận xử lý một tác vụ X, thì hàm callback `fn` vẫn phải tiếp tục chờ. Và nếu X tốn 10 phút để hoàn thành, thì sau 10 phút đó, `fn` mới được thực thi.
