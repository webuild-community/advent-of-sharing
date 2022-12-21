---
title: 60 khung hình 1 giây
author: huytd
date: 12-20-2022
---
Ở bài trước thì @ZeroX-DG cũng có đề cập đến chuyện 60 fps và requestAnimationFrame, trong bài này mình sẽ nói rõ hơn về vấn đề này một tí.

Hầu hết màn hình của các thiết bị điện tử ở thời điểm hiện tại (2022) đều có refresh rate là 60Hz, một số tay chơi nhiều tiền của như anh lái xe ôm ở một cộng đồng công nghệ nào đó thì thích xài màn 120Hz. Và con số này có nghĩa rằng, trình duyệt của bạn sẽ phải render 60 hoặc 120 khung hình (frames) 1 giây để đáp ứng kịp với tốc độ refresh của màn hình. Vậy, việc này có gây tốn kém gì không?

Trước hết, để render ra được một khung hình, trình duyệt có thể cần phải thực hiện những bước sau:

https://raw.githubusercontent.com/huytd/everyday/master/_meta/inside-a-frame.png![image](https://user-images.githubusercontent.com/613943/208791428-04162b06-fa6d-41a1-bce7-3e5b6fe963c5.png)

1. Handle các input event (như touch, mouse wheel, click, keypress,...), thực thi các đoạn code JavaScript trigger từ các timers (setTimeout, setInterval,...)
2. Nếu sau quá trình thực thi JavaScript mà có các thay đổi về layout hoặc style, thì trình duyệt sẽ phải recalculate lại layout và style cho từng element. Trước đó thì callback được scheduled bởi hàm `requestAnimationFrame` sẽ được gọi.
3. Sau khi tính toán xong layout và style, thì browser sẽ vẽ (paint) mọi thứ ra màn hình, chúng ta tạm không nói tới bước này ở trong bài viết này.

Mình xài từ "có thể" ở đây là vì một trong các bước trên hoàn toàn có thể được bỏ qua nếu như hành động trigger bước đó không xảy ra, ví dụ nếu user không tương tác gì trên trang web, thì sẽ không có event handler nào được trigger, hoặc  chúng ta chỉ đang dùng CSS animation đổi màu một element, thì bước layout calculation không cần phải xảy ra,...

Chi tiết hơn các bạn có thể xem trong hình sau:

https://raw.githubusercontent.com/huytd/everyday/master/_meta/frame-lifecycle.png![image](https://user-images.githubusercontent.com/613943/208791449-2fe28275-53cb-44b9-bfb2-b319cefa3a61.png)

Tất cả những bước trên cần phải được thực hiện trong phạm vi 1s / 60Hz ≈ 16ms. Nếu không thì sao?

Giả sử ở một frame, bước thực thi JavaScript của chúng ta tốn quá nhiều thời gian và vượt luôn cả khung thời gian dành cho `requestAnimationFrame` và bước Layout/Paint, thì chúng ta sẽ bị hụt đi frame đó (callback của `requestAnimationFrame` cũng sẽ bị skip ở frame này luôn). Nếu browser đang thực hiện animation, hoặc scroll, hoặc làm gì đó khác thì lúc này, trên màn hình sẽ xuất hiện hiện tượng giật hình.

Ngoài JavaScript, thì layout recalculation cũng là một bước xử lý rất tốn kém có thể chiếm phần lớn thời gian của một frame. Nếu vì một lý do gì đó chúng ta trigger việc layout recalculation (như là set thuộc tính `top`, `left` để làm animation như thời jQuery, đọc/thay đổi nội dung text của một element, load một đống hình nhưng không set trước kích thước của `<img>` element, để nó nhảy layout khi load xong,...) thì bạn cũng sẽ phải đối mặt với chuyện miss frame. Đọc đến đây thì các bạn có thể mở chiếc link đình đám này ra để tham khảo lại các tác vụ nào sẽ trigger reflow/layout calculation: https://gist.github.com/paulirish/5d52fb081b3570c81e3a

Và bất ngờ chưa, ngay cả bước style calculation cũng là một khâu cực kì tốn kém!!! [Xấp xỉ 50% thời gian](https://web.dev/reduce-the-scope-and-complexity-of-style-calculations/) của khâu style calculation là thời gian mà trình duyệt dành để tính toán các CSS selector, nên việc tránh dùng các selector phức tạp sẽ giúp cải thiện tốc độ xử lý ở khâu styling rất nhiều. Ví dụ những selector như `.box:nth-last-child(-n+1) .title` {} thì sẽ tốn nhiều thời gian để tính hơn là `.title {}`.

Có thể bạn đang nghĩ là làm CRUD app thì cần gì quan tâm chuyện này, chỉ khi nào làm những ứng dụng nặng về animation các kiểu như landing/demo page hay game các kiểu thì mới cần quan tâm đến vấn đề này?

Ừ thì đúng là như thế thật. Nhưng CRUD app thì vẫn sẽ có những trường hợp chúng ta cần quan tâm đến vấn đề budget thời gian của một khung hình, đơn giản nhất là vấn đề scroll lên xuống của user.

Để tối ưu performance khi render thì cần rất nhiều yếu tố mà chúng ta không thể đề cập hết trong phạm vi bài viết dược, ví dụ quan trọng nhất là việc đo lường (measurement) và phân tích (analyze) để tìm ra đâu là bottleneck gây ra hiện tượng giật/lag, để tìm hiểu thêm về phương pháp đo lường/phân tích, các bạn có thể tham khảo tại bài viết sau: https://developer.chrome.com/docs/devtools/evaluate-performance/

---

Hy vọng qua bài viết này, mặc dù chưa cần dùng đến ngay trong công việc hằng ngày, thì các bạn cũng gọi là biết thêm fact và để khi cần thì có thêm keyword mà lookup. 
