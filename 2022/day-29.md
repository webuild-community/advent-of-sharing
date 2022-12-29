---
title: "Hướng dẫn dùng Chrome DevTools để đo runtime performance"
author: quannt
date: 12-22-2022
---

Gần đây có một số bạn nhắn tin hỏi mình cách đo runtime performance bằng Chrome DevTools, thấy trên mạng cũng có nhiều bài hướng dẫn nhưng đa số là đã outdate và viết cho các phiên bản Chrome cũ, nay mình sẽ viết một bài mới dùng Chrome bản mới nhất (108.0.5359.124). Lưu ý là bạn nên hiểu qua về cách browser hoạt động và render thì mới thấy bài viết này có tác dụng được, nếu chưa nắm rõ thì có thể kéo lên trên đọc [bài AoF 27 của huytd](./day-27.md) hoặc đọc [bài viết này](https://web.dev/rendering-performance/). Ok chúng ta bắt đầu.

Việc đầu tiên là mở Incognito mode, mở Devtools, ấn vào tab Performance, ở ví dụ này chúng ta sẽ dùng https://googlechrome.github.io/devtools-samples/jank/ làm ví dụ (hoặc các bạn có thể tải file example01.json đính kèm), sau đó ấn vào nút tròn để bắt đầu Record, sau đó quay lại trang web và thực hiện vài thao tác bạn đang nghi ngờ có performance kém, xong xuôi thì ấn stop, sẽ thấy màn hình kết quả như dưới:

<!-- Image 1 -->
<img width="1256" alt="" src="https://user-images.githubusercontent.com/8603085/209944976-b7081564-e3af-48dd-acb4-171c970f88ef.png">

Thoạt nhìn sẽ thấy rất choáng ngợp vì quá nhiều thông tin được hiển thị, không sao, chúng ta sẽ chỉ quan tâm đến phần nào dễ thôi :yaomin:. Như đã đánh dấu trên hình, khung đầu tiên hiển thị performance của CPU bằng area chart, màu vàng là scripting (JS), màu tím là rendering (Style, Layout), màu xanh là painting (Paint, Composite). Nhìn qua ảnh ở trên thì các bạn có thể thấy ngay phần lớn cái area chart là màu tím, vậy suy ra phần đang ảnh có ảnh hưởng nhất là rendering (Style, Layout).

Ngoài ra, các bạn cũng cần để ý đường màu đỏ, đó là những lúc FPS drop khá nhiều và sẽ ảnh hưởng đến UX.

Ok, tiếp theo Zoom kĩ hơn vào phần Main ở dưới để xem browser đã thực sự làm gì, chỗ này mình hơi lười viết nhưng chắc nhìn ảnh các bạn cũng hiểu:

<!-- Image 2 -->
<img width="1251" alt="" src="https://user-images.githubusercontent.com/8603085/209945034-bee8ed31-daad-45d3-9635-de3e316c7092.png">

Bước tiếp theo thì đơn giản thôi, click vào 1 trong 3 cái menu ở bên cạnh và mò từng dòng một thôi, ở ví dụ này thì chúng ta đã tìm ra thủ phạm khá đơn giản, dĩ nhiên trong thực tế thì các bạn sẽ phải mò khá nhiều qua tầng tầng lớp lớp call stack của các framework:

<!-- Image 3 -->
<img width="1255" alt="" src="https://user-images.githubusercontent.com/8603085/209945041-568cfbb0-6b30-4a43-b689-068f1fd5893d.png">

Ok vậy là các bạn đã có thể nắm sơ qua về phần CPU và Main thread cũng như phần Summary của tab Performance, mình sẽ đưa nốt một ví dụ để các bạn hiểu về phần Frame, lần này chúng ta sẽ dùng http://output.jsbin.com/woyoce/1/quiet hoặc các bạn có thể tải file [example02.json](./media/29/example02.json) lên rồi làm theo. Đến đây thì các bạn sẽ tự hỏi tại sao phần CPU ở đây trong lại khác thế, câu trả lời là ... mình cũng không biết :yaomin: but anyway, nếu zoom in vào phần màu đỏ, sẽ thấy được rất nhiều frame bị drop, dẫn đến tình trạng giật lag:

<!-- Image 4 -->
<img width="1148" alt="" src="https://user-images.githubusercontent.com/8603085/209945053-f92a3c9a-4343-4a18-b38d-d38f8249a9cc.png">
<!-- Image 5 -->
<img width="1144" alt="" src="https://user-images.githubusercontent.com/8603085/209945062-0eb6bae7-b5a6-4a9f-bd24-11ba9d8cc16f.png">

Áp dụng kiến thức vừa học, zoom vào main thread, click vào 1 Task:

<!-- Image 6 -->
<img width="1145" alt="" src="https://user-images.githubusercontent.com/8603085/209945075-37c390cf-7a37-4ce0-b0d8-4721d43b1905.png">

sau đó vào Call Tree mò, sẽ tìm ra thủ phạm, là dòng code sau đây:

```js
for (var b = 0; b < boxes.length; b++) {
  boxes[b].style.webkitTransform = makeTransform(b);
}
```

<!-- Image 8 -->
<img width="1145" alt="" src="https://user-images.githubusercontent.com/8603085/209945086-8890b184-7a62-489d-9fc9-c5e046b60b8f.png">

Ngoài ra thì DevTools còn rất nhiều tính năng khác như là đo memory, fps meter, xem composite layer, các bạn có thể đăng ký học course này để biết thêm https://www.udacity.com/course/browser-rendering-optimization--ud860 , lưu ý là course này làm từ 2015 nên đã khá outdated.
