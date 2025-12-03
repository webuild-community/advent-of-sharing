**Author:** huytd

---

# Side-project ký sự: Leetcode Slack Bot | Huy's Blog

Đáng ra thì bài tiếp theo trong sê ri Side Project Ký Sự này mình sẽ viết phần 2 về ChatUML, nhưng có nhiều vấn đề khiến mình ko thể tiếp tục share được, đành cáo lỗi với các bạn. Tóm tắt nhanh thì tổng doanh thu của dự án đến thời điểm này là 20k USD, và đã ngừng tăng trưởng. Nhưng mà thôi, mình sẽ tiếp tục viết về các side project khác.

![icon](https://notes.huy.rocks/apple-touch-icon.png)

https://notes.huy.rocks/posts/building-algorithm-bot.html

![cover](https://notes.huy.rocks/img/default.jpg)

---

## Algorithm Bot là gì?

Algorithm Bot là một con Slack bot hoạt động trong channel `#algorithm` ở [cộng đồng WeBuild VN](https://join.slack.com/t/we-build-vn/shared_invite/zt-2tvw0xd5s-I7qanEm6kJ_wSn6gcwOtRw). Được chính các thành viên làm ra để tự động hoá việc nhắc nhở các thành viên tham gia luyện Leetcode mỗi ngày, nâng cao tay nghề, với motto là: **"A leetcode a day, keep layoff away".**

Nhưng vì một lý do nào đó, mà từ khi làm ra con bot này thì mình đi phỏng vấn toàn rớt.

Trong bài viết này, mình sẽ chia sẽ một tí về cách mà con bot này được làm ra. Mục đích không phải để các bạn hiểu thêm về những công nghệ tối tân đằng sau con bot, mà chỉ để khoe mẽ. Vì có thể các bạn chưa biết, một engineer giỏi không phải là người có thể build được tất cả mọi thứ, mà là một người biết cách khoe những gì mình đã build.

---

## Tính năng nhìn từ bên ngoài

Ở ngoài nhìn vào thì Algorithm Slack Bot có các chức năng sau:

- Cho phép user đăng ký tham gia giải bài và nhận thông báo mỗi ngày
- Tự động gửi tin nhắn vào channel `#algorithm`, tag các thành viên đã đăng ký để nhắc làm bài
- Khi user giải xong bài trên Leetcode, thì có thể bấm vào nút **"Tui giải xong rồi"** trên Slack để điểm danh. Algorithm Bot sẽ kiểm tra tài khoản Leetcode của user để xem có đúng là đã giải xong bài chưa, nếu chưa thì sẽ bị nó mắng.

Bạn nào giải bài và điểm danh xong thì sẽ bị khen thưởng:

![checkin](https://notes.huy.rocks/posts/img/algobot/algorithm-bot-checkin.png)

Còn những ai lề mề chậm trễ, chờ tới cuối ngày mới điểm danh thì sẽ được phạt:

![late checkin](https://notes.huy.rocks/posts/img/algobot/algorithm-bot-latecheckin.png)

---

## Bên trong con bot

Ở trong nhìn ra, thì Algorithm Bot thực chất chỉ là [một file TypeScript 800 dòng](https://github.com/huytd/algorithm-slackbot/blob/main/main.ts), chạy trên Deno Deploy. Toàn bộ dữ liệu chỉ là 1 cục JS object lưu trong [Deno KV](https://docs.deno.com/deploy/kv/manual/on_deploy/). Ngoài mình ra thì còn có hai bạn khác là [Tuan Cá Thu (Tuna)](https://iamtuna.org/) và [Huấn Pa Tê (Pêtr)](https://github.com/hmhuan) tham gia phát triển.

Mô hình hoạt động của con bot có thể được tóm tắt bằng sơ đồ bên dưới:

![flow](https://notes.huy.rocks/posts/img/algobot/algorithm-bot-flow.png)

Chúng ta có 3 thành phần chính, handle 3 flow khác nhau:

---

## 1. Flow đăng ký

Để Algorithm Bot có thể nhận diện được mình là ai ở trên Slack và trên Leetcode, thì bạn có thể gửi Slack command:

```bash
/enroll <leetcode-id>
```

Khi gặp lệnh này, Slack sẽ gửi một HTTP request đến Algorithm Bot server.

Ở [phía server](https://github.com/huytd/algorithm-slackbot/blob/main/main.ts#L439-L469), chúng ta đọc dữ liệu từ Deno KV để lấy ra bảng tham chiếu:

```text
[Slack ID : Leetcode ID]
```

Rồi lưu hoặc update Leetcode ID mới nếu cần. Nếu bạn đổi account Leetcode thì chỉ cần chạy lại lệnh `enroll` là được.

Tương tự với flow đăng ký nhận thông báo khi có bài mới được post, ở đây chúng ta dùng lệnh:

```bash
/subscribe
/unsubscribe
```

Các bạn có thể [tham khảo source](https://github.com/huytd/algorithm-slackbot/blob/main/main.ts#L380-L437) để xem cách implement.

---

## 2. Cron job hằng ngày

Cron job chạy mỗi ngày vào lúc `00:05 UTC`, tương đương với `7:05 sáng giờ VN`, 5 phút sau khi Leetcode mở bài daily.

Khi chạy, Algorithm Bot sẽ:

1. Gửi một GraphQL request với query `questionOfToday` đến Leetcode.
2. Lấy thông tin về bài daily.
3. Soạn một Slack message có nội dung giống như screenshot ở trên.
4. Lấy danh sách những ai đăng ký nhận thông báo (từ flow phía trên) và tag vào message.

Nút bấm để điểm danh cũng sẽ được tạo, và đính kèm với ID của câu hỏi daily, khi các bạn điểm danh thì server sẽ biết được các bạn đang điểm danh cho bài nào.

---

## 3. Flow xác thực điểm danh

Cuối cùng là **flow xác thực việc điểm danh**. Khi nhấn nút điểm danh, một HTTP request khác sẽ được gửi đến Algorithm Bot server, kèm với đó là thông tin người điểm danh.

Dựa vào data đã lưu, chúng ta:

1. Xác định được Leetcode ID.
2. Gửi GraphQL request với query `getACSubmissions`.
3. Lấy ra 5 submission gần nhất để xem bài đang giải có nằm trong đó không.
4. Nếu có thì gửi tin nhắn thông báo vào thread trên Slack, rồi lưu kết quả điểm danh vào DB.

Việc xác định thứ hạng khi điểm danh dựa vào hai yếu tố:

- 3 thành viên điểm danh đầu tiên sẽ được huy chương vàng 🥇
- Từ thành viên thứ 4 trở đi, nếu bài của bạn được accept trong vòng 3 tiếng kể từ lúc post bài (chứ không phải lúc điểm danh) thì bạn sẽ nhận được huy chương bạc 🥈
- Trong vòng 3 đến 6 tiếng thì huy chương đồng 🥉
- Lâu hơn nữa thì... 😘

Ngoài ra Algorithm Bot còn có chức năng tạo challenge phụ, bằng cách paste link của bài Leetcode vào, cách hoạt động tương tự như trên. Nhưng vì vẽ xong sơ đồ ở trên rồi mình mới nhớ ra, nên nếu các bạn nhìn vào sơ đồ thì sẽ thấy mình không nhắc đến chức năng này.

---

## Vì sao chọn TypeScript, Deno và Deno Deploy?

Nói sơ một chút về lý do tại sao chọn TypeScript, Deno và Deno Deploy, trong khi có nhiều [ngôn ngữ cool ngầu hơn](https://www.rust-lang.org/) ở ngoài kia?

Sau khi xác định được sẽ build cái gì, thì mình nhận thấy con Slack bot cần được deploy lên một nơi có những thứ sau:

- Một cái DB để lưu dữ liệu  
- Khả năng chạy cron job để chạy task gửi bài  
- Có thể serve một cái HTTP server để làm những công việc khác

Đến khúc này thì có rất nhiều giải pháp:

- Mình có thể thuê một con VPS và có thể setup tất cả mọi thứ, từ DB đến cron job, nhưng mặc dù mình là một người hết mình vì cộng động, không đời nào mình chịu bỏ ra 5 đô một tháng để thuê server hết.
- OK, không dùng server riêng thì setup một cái HTTP server trên Fly.io hoặc Vercel, Netlify, xong rồi dùng Github Action hoặc Vercel CRON để chạy task cũng được, rồi lại phải tìm một service khác để handle DB. Như vậy để run một cái app nhỏ, cần dùng đến 3, 4 service khác nhau, rồi về sau ai maintain cái đống đó? Mình thích những thứ gì nó tinh gọn, tốt nhất là tất cả nên nằm cùng 1 chỗ.

Vậy nên sau đó thì mình tình cờ tìm ra **Deno Deploy**, và thấy nó tick đủ hết các yếu tố mình cần:

- Built-in database: có  
- Support cron job: có  
- Dễ dàng deploy: có  
- Có thể manage tất cả service ở cùng 1 chỗ: có  
- Miễn phí: có luôn  
- Tác giả blog này đẹp trai tài giỏi: có luôn.

Chỉ có một vấn đề duy nhất đó là Deno Deploy chỉ hỗ trợ Deno/TypeScript, trong khi dự định ban đầu mình muốn xài Rust hoặc Java, nhưng mà không sao, chuyện gì cũng phải có trade off cả.

---

## Kết

Rồi xong, giờ thì các bạn còn chờ gì nữa, join WeBuild và giải algo với tụi mình nào.