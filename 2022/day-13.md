---
title: Flexbox Layout hoạt động như thế nào?
author: ChatGPT
date: 12-06-2022
---

Bài viết ngày hôm nay được thực hiện bởi một vị khách mời đặc biệt, là anh bạn [ChatGPT](https://chat.openai.com/chat), có thể nói, đây là một nước đi lịch sử khi lần đầu tiên AI tham dự chương trình post bài tại WeBuild :dogekek: Mình (huytd) chỉ thực hiện 2 việc: Đặt câu hỏi cho ChatGPT và dịch lại câu trả lời của bạn ấy từ tiếng Anh sang tiếng Việt. Các câu hỏi được đặt ra là:

- “How come Flexbox layout helped reduce the reflow cost?”
- “Could you explain in details how the Flexbox algorithm works?”

Và bây giờ là nội dung bài viết.

---

Flexbox là một phương pháp tổ chức layout hiệu quả hơn rất nhiều so với các phương pháp layout khác như block, inline hay float. Flexbox được thiết kế để giảm thiểu số lần trình duyệt phải tính toán lại layout (recalculate layout), từ đó giảm thiểu chi phí cho việc reflow.

Đối với layout truyền thống như block, inline hay float, vị trí (position) và kích thước (size) của **tất cả mọi element trên trang** đều được tính toán lại mỗi khi layout được update. Và việc tính toán này rất tốn kém, nhất là khi bạn có layout phức tạp hoặc có rất nhiều element trên trang.

Ngược lại, khi dùng flexbox layout, trình duyệt chỉ cần tính toán lại vị trí và kích thước của các element có vai trò là flex container (là các element có thuộc tính `display: flex`), và dùng kết quả tính toán đó để tính tiếp kích thước và vị trí của các element con bên trong container đó. Điều này có nghĩa là, trình duyệt chỉ cần recalculate lại các element con khi một flex container bị thay đổi, từ đó giảm đáng kể số lượng element cần được recalculate, và giảm chi phí reflow rất nhiều lần.

**Chi tiết về cách hoạt động của thuật toán flexbox:**

Đầu tiên, khi có sự thay đổi trên một hoặc một vài flex container, thuật toán flexbox sẽ tính toán lại kích thước của các flex container đó, bằng cách dùng các thuộc tính `width`, `height`, `margin` và các thuộc tính như `flex-direction`, `flex-wrap`, `justify-content`.

Sau khi có được kích thước của flex container, thuật toán sẽ tính tiếp vị trí và kích thước của từng child element, bằng cách: Dùng các thuộc tính như `flex-grow`, `flex-shrink`, `flex-basis` hoặc `align-items`, `align-self` của cả flex container lẫn các child elements để quyết định phần không gian hiện có sẽ được phân chia giữa các child elements, rồi dựa vào đó để tính ra kích thước và vị trí của từng child element.

Cuối cùng, sau khi đã tính xong các thông số ở trên, thuật toán sẽ tiến hành update layout của trang web. Bằng cách set các thuộc tính `position`, `width`, `height` của child elements, cũng như thuộc tính `order` của flex container hoặc các element con trong đó.
