---
title: "Những viewport units mới trong CSS: Bự, Nhỏ, và Linh Động (đậy)"
author: kcjpop
date: 12-21-2022
---

**Cảnh báo**
Phần dưới đây sẽ dùng "_viewport_" để chỉ không gian (thường là hình chữ nhật) trên màn hình mà người dùng đang nhìn thấy. Đối với desktop thì đó là kích thước của trình duyệt, còn trên mobile thì toàn bộ màn hình. Ngoài ra nội dung dưới đây có thể _không chính xác 100%_ theo định nghĩa vì đã bị giản lược theo cách mình hiểu, nên mọi người thấy sai ở đâu chỉ ra giùm nha :pepecry:

---

Hồi đâu gần 10 năm trước, các trình duyệt (kể cả IE 11) đã hỗ trợ (hầu hết) viewport units trong CSS, bao gồm:

* `vw` 1% chiều dài (width) của viewport
* `vh`: 1% chiều cao (height) của viewport
* `vmin`: giá trị NHỎ HƠN giữa `vw` và `vh` = `min(vh, vw)`
* `vmax`: giá trị LỚN HƠN giữa `vw` và `vh` = `max(vh, vw)`

Vấn đề là giá trị của `vw` và `vh` được tính theo kích thước của "initial containing block", khi root element (aka thẻ `<html>`) vừa được tạo ra. Khi kích thước của "initial containing block" thay đổi thì giá thị của `vw` và `vh` cũng đổi theo, tuy nhiên vẫn không tính đến trường hợp có scrollbar xuất hiện.

Thành ra sẽ có trường hợp `width: 100vw` hay `height: 100vh` nhưng vẫn bị overflowed. Ngoài ra còn có trường hợp khi thanh địa chỉ ẩn/ hiện trên mobile nữa, như Hình 1.

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2019/10/100vh_problem.png?ssl=1)

> **JavaScript The More You Know**: 
> 
> `document.documentElement` chính là thẻ `<html>`, và 2 thuộc tính `document.documentElement.clientWidth`/ `document.documentElement.clientHeight` sẽ trả về chiều dài và chiều cao mà không tính scrollbar vào. Chỉ áp dụng cho `<html>` và `document.body` ở _quirks mode_ thôi.

Chrome 108 vừa phát hành hồi đầu tháng 12 năm nay đánh dấu sự có mặt của 3 nhóm viewport units mới trên tất cả các trình duyệt: Large, Small, và Dynamic.

Nhóm **Bự** là viewport KHÔNG bao gồm mấy cái linh tinh như thanh địa chỉ hay scrollbar. Tương tự như trên chúng ta sẽ có `lvw`, `lvh`, `lvmin`, và `lvmax`. Một ví dụ thực tế là dùng để tạo cover ở đầu bài viết chẳng hạn.

```css
article > header {
  width: 100lvw;
  height: 75lvh;
  background-image: url(cover-illo.jpg);
  background-size: cover;
}
```

Nhóm **Nhỏ** là viewport khi thanh địa chỉ hay scrollbar hiển thị. Bao gồm các units như `svw`, `svh`, `svmin`, và `svmax`. Một ví dụ cụ thể là khi làm modal và chúng ta muốn modal luôn nằm trong vùng nhìn thấy, thay vì bị UI của trình duyệt che lại.

```css
.warning {
  width: 40em;
  height: auto;
  max-width: 100svh;
  max-height: 100svh;
}
```

Hình 2 so sánh giữa Large và Small viewport units
![](https://images.prismic.io/12daysofwebdev/714b4e4d-0d28-467c-9571-6946ab487e35_viewports2.png?auto=compress,format)

Cuối cùng là vũ đoàn **LiDo**, bao gồm `dvw`, `dvh`, `dvmin`, và `dvmax`, theo ý nghĩa là:

* Nếu UI của trình duyệt hiển thị => dynamic viewport units = small viewport units
* Ngược lại, nếu UI của trình duyệt ẩn đi => dynamic viewport units = large viewport units

Vì giá trị của nhóm `dv*` tùy thuộc vào UI của trình duyệt, và UI của trình duyệt thì thay đổi liên tục, nên cần phải cân nhắc khi dùng các giá trị này vì có thể ảnh hưởng performance, cũng như tạo ra chuyển động không cần thiết.

Hớt.

P.S. Trong hình 2 có nhắc đến `*vb` với `*vi` units. Hai cái này là thay vì dùng `width/ height` lại dựa vào trục `block/ inline`. Còn trục `block/ inline` là gì thì để ai đó khác nói vậy :pepedeep: