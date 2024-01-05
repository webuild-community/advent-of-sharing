---
title: An over-engineering guide to sync Obsidian for free
author: huytd
date: 2023-12-19
---

Obsidian là công cụ được rất nhiều người lựa chọn để tạo và quản lý ghi chú, hoạt động trên cả máy tính lẫn các thiết bị di động. 

Điểm mạnh của Obsidian là sử dụng markdown – nên bạn có thể thoải mái migrate dữ liệu sang các ứng dụng khác, không bị phụ thuộc vào ai cả. Thêm nữa, Obsidian hỗ trợ rất nhiều plugins nên có khả năng mở rộng gần như là không giới hạn.

Một vấn đề quan trọng khi sử dụng các công cụ ghi chú là khả năng đồng bộ giữa các máy tính khác nhau. Obsidian có gói cloud sync giá 8 đô một tháng, tuy nhiên phần lớn người dùng đều chọn các giải pháp thay thế như sync lên iCloud, Dropbox hay Git,...

Nói sơ một chút về các giải pháp này thì:

- Dropbox sync: Đòi hỏi máy tính phải cài thêm desktop client của Dropbox, nên rất khó để setup trên điện thoại.
- iCloud: hoạt động rất hiệu quả, nếu các bạn chỉ xài điện thoại và máy tính của Apple. Có một edge case cho vụ này là một số công ty có policy chặn luôn iCloud, thì cách này không xài được.
- Git: Rất tiện dụng và ổn định, thao tác sync đơn giản chỉ là commit và push lên một cái git repo nào đó do người dùng lựa chọn. Tuy nhiên cũng vì xài Git nên tốc độ sync chậm (mặc định là 15 phút sau mỗi thay đổi), và khả năng xảy ra conflict rất cao. Trên mobile thì Git client cũng chưa hoạt động tốt cho lắm.

Ngoài các giải pháp trên thì hôm nay mình sẽ giới thiệu một phương pháp sync mới có tên là [LiveSync](https://github.com/vrtmrz/obsidian-livesync).  Plugin này có ưu điểm là rất dễ setup trên cả máy tính lẫn điện thoại, và việc sync diễn ra trong thời gian thực, dữ liệu được lưu trên máy chủ của bạn.

![d2](https://github.com/webuild-community/advent-of-sharing/assets/613943/b1d2eafa-c095-4220-92de-25c26032fc91)

Để sử dụng LiveSync thì bạn cần làm 2 việc:

- Dựng một cái CouchDB instance (để làm server lưu trữ)
- Cài plugin LiveSync vào Obsidian

Việc deploy CouchDB cũng khá là đơn giản, bạn có thể deploy lên server cá nhân hoặc tiện hơn là xài Fly.io – chi phí để run một instance CouchDB cho nhu cầu ghi chú cá nhân rất rẻ, ít khi nào vượt quá hạn mức 5 đô miễn phí của Fly.

Chi tiết về cách cài đặt có thể xem tại link  https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_flyio.md, sau bước này thì bạn sẽ có một instance CouchDB chạy trên Fly.io, và có địa chỉ kiểu https://your-couchdb-instance.fly.dev

Lưu ý là trong hướng dẫn trên, username và password mặc định của CouchDB là `campanella`, khi cài đặt thực tế, các bạn nên đổi thành username và password tự chọn của riêng mình.

Sau cùng là cấu hình cho plugin LiveSync kết nối tới instance CouchDB của mình, chi tiết xem tại https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/quick_setup.md

Khi cần setup Obsidian ở trên một máy tính khác (hay điện thoại), các bạn chỉ cần tạo một Vault mới trên máy tính đó, cài plugin LiveSync vào, và kết nối tới CouchDB instance của bạn.

Lưu ý là LiveSync plugin chỉ sync vault content, nhưng không sync thư mục cấu hình của vault (thư mục `.obsidian`, đây là nơi chứa các thiết lập của vault như theme, danh sách plugin đã cài,...). Nếu muốn sync thì các bạn vẫn có thể bật chế độ "Sync hidden files" trong phần Sync settings của plugin.
