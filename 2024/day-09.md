**Author:** Tuna

---

# LeetCode Starter Kit

Một vài toolkit và tips cho những bạn mới bắt đầu với LeetCode một cách hiệu quả. Bên cạnh đó, mình cũng chia sẻ một số nguồn tham khảo để học về thuật toán và cấu trúc dữ liệu…

![icon](https://iamtuna.org/assets/apple-touch-icon.png)

https://iamtuna.org/2024-12-08/Leetcode-starter-kit

![my-leetcode-stats-small](https://iamtuna.org/images/2024/12/my-leetcode-stats.png)

[LeetCode starter kit link](https://iamtuna.org/2024-12-08/Leetcode-starter-kit)

![my-leetcode-stats](https://iamtuna.org/images/2024/12/my-leetcode-stats.png)

---

## TL;DR

- Bắt đầu LeetCode càng sớm càng tốt.
- Dùng Python để code, nếu không bắt buộc dùng một ngôn ngữ nhất định.
- Code bằng IDE nhưng tắt auto-complete.
- Đo thời gian giải bài.
- Note lại các thông tin về bài toán: cách tiếp cận, các lưu ý, time/space complexity, các edge cases, v.v.
- Giải theo chủ đề.
- Làm bài daily để duy trì thói quen và động lực.

Nếu bạn nóng lòng xem danh sách về các tài liệu học Cấu trúc dữ liệu và giải thuật, hãy chuyển tới [Các nguồn tham khảo để học về DS&A](https://iamtuna.org/2024-12-08/Leetcode-starter-kit#dsa-resources).

---

### Giới thiệu

Mình không giỏi về thuật toán và cũng không phải dân chơi [CP (Competitive Programming)](https://en.wikipedia.org/wiki/Competitive_programming), nên khi bắt đầu với LeetCode, mình đã bị ngợp và không biết bắt đầu từ đâu, mất rất nhiều thời gian để giải những bài đầu tiên. Mình đoán nhiều bạn cũng có cảm giác tương tự khi bắt đầu với LeetCode. Bài viết này tổng hợp một vài thứ có thể hữu ích cho những bạn giống mình.

> Trong bài này, thuật ngữ “Thuật toán” hoặc “Algorithm” được dùng để chỉ các thuật toán theo sách giáo khoa hoặc cần một vài bước suy luận và các cách tiếp cận cụ thể, khác với nghĩa của từ “thuật toán” nói chung, vốn có thể dùng cho tất cả logic trong code.

---

### 0. Tại sao lại là [LeetCode](https://leetcode.com/)?

Hiện nay có khá nhiều nền tảng để luyện tập thuật toán như [TopCoder](https://www.topcoder.com/), [Codeforces](https://codeforces.com/), [CodeChef](https://www.codechef.com/), [Kattis](https://open.kattis.com/), v.v. LeetCode hay được nhắc đến như một mặc định khi nói về luyện tập cho coding interview. Mặc dù mỗi người có lý do riêng để chọn nền tảng phù hợp, dưới đây là những lý do mình chọn LeetCode:

- Không yêu cầu parse input/output, giúp bạn tập trung vào logic giải quyết vấn đề.
- Lời giải tương tự như viết một hàm thông thường, không phải bận tâm về cách input/output được format như thế nào.
- Mô tả ngắn gọn và straight-forward, giúp bạn hiểu được yêu cầu của bài toán ngay khi đọc đề thay vì phải đọc và suy nghĩ nhiều.

By the way, lựa chọn nền tảng nào cũng được, điều quan trọng là chốt lại một nền tảng và **bắt đầu**.

---

### 1. Lúc nào thì nên bắt đầu LeetCode?

Câu trả lời ngắn gọn là **bây giờ**.

> Nếu bạn chưa từng học về Data Structures & Algorithms (DS&A), hãy học một khoá về DS&A trước khi bắt đầu làm LeetCode. Điều này sẽ giúp bạn có cái nhìn tổng quan về các thuật toán và cấu trúc dữ liệu, giúp bạn dễ dàng hơn khi tiếp cận các bài toán phức tạp hơn cũng như có thể đọc hiểu được các lời giải từ các bài viết trên mạng.

Với các bạn đã học DS&A (chuyên ngành CSE hoặc tự học), nếu ít khi phải giải quyết vấn đề bằng một thuật toán nào đó, khả năng bạn sẽ mất cảm giác về việc sử dụng thuật toán, các cấu trúc dữ liệu phức tạp ngoài `ArrayList` hoặc `HashMap`, v.v. Bắt đầu sớm sẽ giúp bạn lấy lại cảm giác và xây dựng được một [mental model](https://en.wikipedia.org/wiki/Mental_model) về các thuật toán và cấu trúc dữ liệu cần thiết để giải các bài khó hơn hoặc áp dụng được vào công việc hằng ngày.

Không cần phải phân vân liệu có cần học lại DS&A trước hay không, chỉ cần bạn **bắt đầu**. Khi gặp phải một bài toán cần một giải thuật cụ thể, lúc đó hãy tham khảo các tài liệu liên quan (xem [các nguồn tham khảo](https://iamtuna.org/2024-12-08/Leetcode-starter-kit#dsa-resources)). Phương pháp tiếp cận vừa làm vừa ôn / học này sẽ đơn giản và có hướng đi rõ ràng hơn việc học lý thuyết thuần rồi mới bắt tay vào làm bài tập. Công thức của mình là:

> Làm LeetCode.  
> Gặp câu không giải được.  
> Xem lời giải.  
> Tìm hiểu các cấu trúc dữ liệu và giải thuật liên quan.  
> Giải các bài liên quan để củng cố.

Tuy nhiên, để tránh nản lòng, bạn **không nên chọn một câu ngẫu nhiên để làm bài đầu tiên**. Nếu câu LeetCode đầu tiên của bạn yêu cầu đến những khái niệm khó như Dynamic Programming, LinkedList, hay Binary Tree, bạn sẽ dễ cảm thấy bế tắc. Vì vậy, hãy bắt đầu với những câu cơ bản, dễ hiểu để xây dựng nền tảng.

Một vài bài LeetCode đơn giản giúp bạn khởi động:

> - Two Sum (Easy - Array): Một bài cơ bản giúp làm quen với cách xử lý mảng và sử dụng HashMap.  
> - Reverse String (Easy - String): Một bài luyện tập đơn giản liên quan đến thao tác chuỗi.  
> - Merge Two Sorted Lists (Easy - LinkedList): Giúp bạn làm quen với cách thao tác trên danh sách liên kết.  
> - Best Time to Buy and Sell Stock (Easy - Array): Một bài cơ bản để hiểu cách tìm giá trị tối ưu với một vòng lặp đơn.  
> - Valid Parentheses (Easy - Stack): Một bài tập phổ biến để làm quen với cấu trúc dữ liệu Stack.

Bắt đầu với những bài như thế này không chỉ giúp bạn làm quen với nền tảng LeetCode mà còn mang lại cảm giác thành công, từ đó tăng thêm động lực để tiếp tục hành trình.

---

### 2. Dùng ngôn ngữ lập trình gì cho LeetCode?

Có ba ngôn ngữ bạn nên cân nhắc lựa chọn khi làm LeetCode:

1. **Ngôn ngữ mà team hoặc công ty bạn nhắm đến đang sử dụng.**  
   Nếu bạn nhắm đến ứng tuyển vào một công ty hoặc team yêu cầu sử dụng ngôn ngữ cụ thể, hãy ưu tiên học và thực hành bằng ngôn ngữ đó.

2. **Ngôn ngữ bạn đang sử dụng và quen thuộc.**  
   Sử dụng ngôn ngữ mà bạn đã nắm vững sẽ giảm bớt áp lực khi vừa phải học thuật toán vừa phải làm quen với cú pháp mới. Việc này giúp bạn tập trung nhiều hơn vào tư duy giải quyết vấn đề và cải thiện logic lập trình.

3. **Python.**  
   Với mình, Python là lựa chọn lý tưởng sau *pseudo code* để mô phỏng thuật toán. Ngôn ngữ này rất dễ học, cú pháp đơn giản và không yêu cầu nhiều bước chuẩn bị. Ví dụ:

   - Để tạo một array list, bạn chỉ cần viết:

     ```python
     array = []
     ```

   - Với hash map, chỉ cần:

     ```python
     m = {}
     ```

   Điều này rất tiện lợi vì bạn không cần bận tâm về việc import các thư viện như `ArrayList` trong Java hay khai báo các template như C++.

#### Lợi ích của Python khi làm LeetCode

1. **Cú pháp ngắn gọn**

   Python cho phép bạn viết mã ngắn gọn và tập trung hơn vào logic.

   ```python
   nums = [1, 2, 3]
   mapping = {"a": 1, "b": 2}
   ```

   Trong khi ở C++ hoặc Java, bạn cần khai báo kiểu dữ liệu của biến, import thư viện liên quan, làm code trở nên dài hơn và phải nhớ đường dẫn của thư viện đó.

2. **Thư viện tích hợp sẵn**

   Python có số lượng thư viện tích hợp lớn, hỗ trợ nhiều cấu trúc dữ liệu và thuật toán. Một vài ví dụ:

   - `heapq`: Hiện thực Priority Queue.
   - `collections.deque`: Hỗ trợ Double-ended Queue.
   - `math`: Hỗ trợ các hàm toán học cơ bản như `gcd`, `factorial`, `sqrt`, v.v.

3. **Code block bằng indent**

   Một điều ít khi được để ý là việc dùng indent (4-space) để định nghĩa các code block thay vì các cặp `{...}` hay `begin...end` trong Pascal thực sự hữu ích, nhất là trong phỏng vấn khi dùng bảng, vì nó giúp code ngắn gọn và tiết kiệm không gian rất nhiều.

   Ví dụ, khi phải viết 2 vòng lặp lồng nhau:

   Java / C++:

   ```cpp
   for (int i = 0; i < n; i++) {
       for (int j = 0; j < m; j++) {
           // do something
       }
   }
   ```

   Python:

   ```python
   for i in range(n):
       for j in range(m):
           # do something
   ```

---

### 3. Hãy đo thời gian giải bài

Khi bạn bắt đầu giải bài, hãy đo thời gian mà bạn đã dành cho mỗi bài toán. Điều này giúp bạn đánh giá được kĩ năng của mình đã tiến bộ như thế nào, có dạng bài nào mà bạn mất nhiều thời gian hơn, hay cần cải thiện thêm.

Ngoài ra, đặt ngưỡng thời gian cho mỗi level của bài toán (Easy, Medium, Hard) để không rơi vào việc suy nghĩ quá lâu và thường dẫn đến việc dùng trick hay các hacky solution để được accept.

Ngưỡng thời gian mình đặt cho mỗi level (bạn có thể điều chỉnh ngưỡng thời gian cho phù hợp với mình):

> - Easy: Dưới 15 phút.  
> - Medium: Dưới 30 phút.  
> - Hard: Dưới 60 phút.

Nếu vượt quá thời gian mà vẫn chưa tìm ra lời giải, mình sẽ xem hướng dẫn hoặc lời giải để học hỏi cách tiếp cận. Sau đó, mình không vội code và submit ngay, mà để dành bài đó cho ngày hôm sau, nhằm kiểm tra xem mình đã thực sự hiểu và áp dụng được phương pháp hay chưa.

---

### 4. Viết note

Khi giải một bài toán, hãy note lại các thông tin sau:

> - **Intuition**: Từ mô tả bài toán, suy luận các giải thuật, cấu trúc dữ liệu, hoặc chiến lược có thể dùng để giải bài toán, các edge cases cần lưu ý, v.v.  
> - **Time & Space Complexity**: Đánh giá Big-O của lời giải của bạn.  
> - **Các ghi chú khác**: ví dụ, đã sai ở đâu, cần bao nhiêu lần chạy thử, thời gian giải bài, v.v.

Những ghi chú này giúp bạn luyện tập kỹ năng trao đổi trong quá trình phỏng vấn, cũng như theo dõi lỗi sai và sự tiến bộ của mình qua từng bài toán. Dưới đây là *mẫu ghi chú* mà bạn có thể dùng:

```markdown
# Problem: <problem-name-or-link>

## Intuition
- Ý tưởng chính:
- Cấu trúc dữ liệu / thuật toán sử dụng:
- Edge cases:

## Approach
- Bước 1:
- Bước 2:
- ...

## Complexity
- Time: O(...)
- Space: O(...)

## Attempts / Notes
- Attempt 1: Lỗi gì, tại sao?
- Attempt 2: Cải thiện gì?
- Thời gian giải: ~XX phút
```

---

### 5. Giải theo chủ đề

Sau giai đoạn *kickoff*, khi bạn đã quen với cách tiếp cận bài toán của LeetCode, đây là lúc chuyển sang giải theo chủ đề. Phương pháp này giúp bạn xây dựng sự tự tin và thành thạo trong việc áp dụng thuật toán cũng như cấu trúc dữ liệu liên quan đến từng loại bài toán cụ thể.

#### Tại sao nên giải theo chủ đề?

1. **Xây dựng kiến thức có hệ thống**

   Việc tập trung vào từng chủ đề giúp bạn hiểu sâu hơn về thuật toán và cấu trúc dữ liệu liên quan, thay vì chỉ giải rải rác và không có định hướng. Luyện tập nhiều bài toán liên quan liên tiếp nhau sẽ giúp bạn hình thành *mental model* cho giải thuật và cấu trúc dữ liệu đó hiệu quả hơn.

2. **Cải thiện khả năng nhận diện bài toán**

   Nhiều bài toán phức tạp thường thuộc các chủ đề quen thuộc. Khi bạn đã thành thạo một chủ đề, bạn sẽ dễ dàng nhận ra giải pháp tiềm năng khi gặp các bài toán tương tự.

#### Bắt đầu từ đâu?

Chủ đề mình gợi ý bạn nên dùng để bắt đầu là [Binary Search](https://en.wikipedia.org/wiki/Binary_search).

Đây là một thuật toán cơ bản nhưng cực kỳ quan trọng. Binary Search không chỉ xuất hiện trong các bài toán trực tiếp mà còn là nền tảng cho nhiều dạng bài phức tạp hơn như tìm kiếm trên không gian câu trả lời (*searching on answer space*).

Một khi bạn hiểu rõ cách hoạt động của Binary Search, bạn sẽ dễ dàng áp dụng nó vào các bài toán như tìm kiếm trên mảng, tối ưu hóa giá trị, hoặc các bài toán liên quan đến đồ thị.

Cách hiện thực của Binary Search rất dễ rơi vào vòng lặp vô hạn hoặc sai lệch, việc luyện tập nhiều bài toán liên quan sẽ giúp bạn hiểu rõ hơn về cách xử lý các trường hợp đặc biệt và cách debug hiệu quả.

Một thông tin giúp bạn thêm tự tin là [thư viện chuẩn của Java từng có bug trong code của hàm Binary Search](https://dev.to/matheusgomes062/a-bug-was-found-in-java-after-almost-9-years-of-hiding-2d4k).

(Bạn có thể tham khảo Study Plan về [Binary Search](https://leetcode.com/studyplan/binary-search/) trên LeetCode để bắt đầu.)

Tiếp theo, bạn có thể chọn chủ đề luyện tập dựa trên sở thích cá nhân hoặc các dạng bài toán bạn thường gặp. Đừng quên tận dụng [Study Plan](https://leetcode.com/studyplan/) của LeetCode để xây dựng lộ trình phù hợp.

Ngoài ra, bạn cũng có thể tham khảo các curated list có sẵn như:

- [LeetCode 75](https://leetcode.com/studyplan/leetcode-75/)
- [Grokking the Code Interview similar list](https://gist.github.com/tykurtz/3548a31f673588c05c89f9ca42067bc4)

để luyện tập hiệu quả hơn.

---

### 6. Một số toolkit khác

#### Làm bài daily

![my-daily-complete](https://iamtuna.org/images/2024/12/my-daily-complete.png)

LeetCode daily problem đôi lúc khó, đôi lúc dễ, và thường mang tính ngẫu nhiên. Tuy nhiên, làm bài daily mang lại hai lợi ích lớn:

- **Duy trì động lực**: Khi bạn đã làm liên tục trong 30 ngày, streak này sẽ trở thành động lực để bạn không bỏ cuộc, tạo thói quen luyện tập đều đặn.
- **Mở rộng kiến thức**: Sự ngẫu nhiên của daily problem giúp bạn tiếp cận các dạng bài mới hoặc học thêm về các Data Structure và Algorithm mà bạn chưa từng gặp, bổ sung thêm vào kỹ năng của mình và cân bằng với việc luyện tập theo chủ đề.

![badges](https://iamtuna.org/images/2024/12/badges.png)

#### Hãy dùng IDE khi mới bắt đầu

Hầu hết chúng ta đều phụ thuộc khá nhiều vào các công cụ hỗ trợ như auto-complete, auto-import, hay thậm chí gần đây là AI để viết code. Chính vì vậy, việc viết trực tiếp trên LeetCode editor có thể khiến bạn gặp khó khăn, đặc biệt với các lỗi như sai cú pháp, thiếu import,…

Để tránh những rào cản ban đầu này, hãy dùng IDE khi luyện tập, với các công cụ tự động hóa được giới hạn hợp lý. Ví dụ, chỉ bật auto-import và tắt auto-complete cùng AI để rèn luyện khả năng viết code thủ công. Cá nhân mình dùng [VS Code - Insiders](https://code.visualstudio.com/insiders/) (phiên bản early release của VS Code) cho LeetCode và cấu hình mặc định tắt hết tất cả các tool automation đi.

Ngoài ra, việc debug trên local vẫn tiện hơn so với việc dùng LeetCode run, nhất là khi code của bạn rơi vào vòng lặp vô hạn.

Bạn có thể tham khảo thêm bài viết [Let’s code with Leetcode](https://iamtuna.org/2024-12-08/letscode) về việc tạo ra một trình chạy test cho các bài giải LeetCode của mình.