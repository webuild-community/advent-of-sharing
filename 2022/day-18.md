---
title: Why should you care about half pixel?
author: ZeroX-DG
date: 12-10-2022
---

Tưởng tượng vào một ngày đẹp trời và bạn vẽ một đường thẳng lên canvas:

```jsx
context.moveTo(1, 0);
context.lineTo(1, 375);
```

Nhưng ôi không, cái đường thẳng của bạn nó lại bị mở mịt như cái tương lai của bạn vậy.

![blurry line on canvas](https://i.stack.imgur.com/VEuAv.png)

Bạn kẹp đầu giữa gối khóc như chị tiếp viên máy bay hướng dẫn nhưng bạn biết nó cũng chẳng thể cứu được bạn.

Bạn zoom trình duyệt to ra, nhỏ lại, và ô kìa, sao nó lại ra thế này? Sao ở một vài cấp độ zoom nó lại rõ mà ở những cấp khác nó mờ?

Hiện tượng này diễn ra do 2 lý do chính:

- Độ lớn của line trong canvas phụ thuộc vào thuộc tính `lineWidth`.
- `lineWidth` thì to ra từ giữa line. Ví dụ toạ độ line là `(10, 10)` , `lineWidth` là `5` và độ dài line là `20` thì thay vì có hình dạng là `Rect(x=10, y=10, width=5, height=20)` thì line sẽ được render theo hình dạng `Rect(x=5, y=10, width=5, height=20)` tức là có sự dịch chuyển toạ độ `x` vì cái `width` của line luôn lớn lên từ ở giữa. (`x_line = specified_x - line_width / 2`)

Nên nếu vẽ line ở toạ độ `(1, 0)` và với `lineWidth=1` thì toạ độ thực sự của nó là `x = 1 - 1 / 2 = 0.5` tức là line sẽ nằm giữa 2 pixel và chiếm một nửa của mỗi pixel. Điều này khiến trình duyệt phải approximate phần nửa chưa bị chiếm còn lại của 2 pixel đó và render bằng một màu đậm bằng một nửa so với phần bị chiếm gây nên hiệu ứng line bị mờ.

![line blurry when render in half pixel](http://diveintohtml5.info/i/canvas-half-pixels-1.jpg)

Để tránh trường hợp này xảy ra thì nếu `lineWidth` của bạn là số chẵn thì toạ độ của bạn nên là số nguyên, ngược lại thì bạn nên thêm `0.5` để tránh render giữa 2 pixel như trên. Ví dụ `lineWidth = 10` thì bạn có thể dùng toạ độ số nguyên `(10, 20)` còn `lineWidth = 5` thì nên dùng `(10.5, 20)` .

![line not blurry when render in full pixel](http://diveintohtml5.info/i/canvas-half-pixels-2.jpg)

Đọc thêm ở đây:

- [http://diveintohtml5.info/canvas.html](http://diveintohtml5.info/canvas.html)
- [https://stackoverflow.com/questions/7530593/html5-canvas-and-line-width/7531540#7531540](https://stackoverflow.com/questions/7530593/html5-canvas-and-line-width/7531540#7531540)
- [https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Applying_styles_and_colors#a_linewidth_example](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Applying_styles_and_colors#a_linewidth_example)
