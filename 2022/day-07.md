---
title: Animation Performance trên web hiệu quả
author: monodyle
date: 11-30-2022
---

Hồi trước lúc mình làm lại blog của mình, vui vui thế nào lại chèn quả animation vô chân trang cho nó có điểm nhấn. Hồi xưa nghèo còn xài màn hình cùi không để ý, giờ mình vẫn nghèo nhưng vay ngân hàng để mua được cái màn có tần số quét cao thì mới nhận ra bây giờ làm animation cho web cũng phải quan tâm tới performance một tý, vì bây giờ nhìn 30fps với 60fps cũng có rất nhiều điểm khác biệt.

Mặc dù sự phát triển của software rất nhanh và số lượng các browser cũng có thay đổi, tuy nhiên vẫn chỉ tồn tại một số ít browser engine xịn xò còn sống tới bây giờ và cơ chế render của chúng gần như là giống nhau, nên mình sẽ giới thiệu về cơ chế render một trang web trước.

Khi browser render một trang web tĩnh sẽ có 3 bước:
1. **Layout** (hay còn gọi là **reflow**): Trình duyệt tính toán không gian và vị trí của các phần tử trên màn hình. Ví dụ: chiều rộng của parent sẽ ảnh hưởng tới các phần tử con bên trong
2. **Paint**: Trình duyệt vẽ lại các phần tử trên màn hình. Mọi thứ trực quan nhất của phần tử sẽ được vẽ ra trong bước này và xếp trên nhiều layer khác nhau
3. **Composite**: Hiển thị các layers đã được vẽ theo đúng thứ tự để các phần tử không bị chồng lên nhau

Cả 3 bước trên sẽ lặp lại thành một quy trình, được gọi là **Composition**.

![image](https://user-images.githubusercontent.com/30283022/204863819-fccfb406-4104-4941-a1db-af0c492073fb.png)

Vì việc thực hiện quá trình render là tuyến tính nên phải xong bước này mới đến bước khác, do vậy nếu việc một thuộc tính thay đổi liên tục và thực hiện đủ sẽ dẫn tới việc browser render chậm hơn. Do vậy đại đa số animation chỉ đạt được 30 fps.

Tuy nhiên có vài thuộc tính sẽ không trigger cả 3 bước trong quá trình render. Mình sẽ chia các thuộc tính ra làm 3 nhóm:
- Heavy: Là các thuộc tính sẽ trigger toàn bộ quá trình render. Ví dụ: `width`, `font-size`,...
- Normal: Thường các thuộc tính này sẽ chỉ trigger hai bước sau. Ví dụ: `color`
- Light: Các thuộc tính ở nhóm này chỉ thực thi bước composite. Ví dụ: `opacity`

Do vậy, chúng ta có thể lợi dụng những thuộc tính này để làm animation hiệu quả hơn. Ví dụ thuộc tính `margin` được xếp vào nhóm đầu, nhưng để thay đổi vị trí của một phần tử, chúng ta có thể lợi dụng thuộc tính `transform` với value là `translateX` để thực hiện chuyển động tương tự.

Để xem các thuộc tính sẽ trigger những bước nào, các bạn có thể tham khảo ở trang này: (https://csstriggers.com/)[https://csstriggers.com/]

_Demo_: https://jsfiddle.net/monodyle/9t7ruwv1/

Mọi người có thể thay đổi css dòng 13 `use-margin` thành `use-transform` để trải nghiệm thử. Hơi khó nhìn nhưng mình lười quay video so sánh quá, nếu có thì sẽ dễ hình dung hơn =))

**Lưu ý nhỏ:** Mình không rõ browser engine khác nhau có cơ chế reduce composition khác nhau hay không. Nhưng có thể bạn không nên cùng sử dụng `margin` và `transform` trong cùng một screen, vì browser sẽ batch những thay đổi lại và render trong một lần. Chi nên nếu đặt cùng nhau thì performance cũng chỉ đạt được ở mức thấp nhất.
