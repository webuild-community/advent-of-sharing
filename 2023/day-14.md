---
title: Side project ký sự - ChatUML
author: huytd
date: 2023-12-14
---

Hẳn mọi người đã từng nghe qua dự án ChatUML. Mình không [dẫn link](https://chatuml.com) hay giới thiệu nhiều về sản phẩm này vì mình bài viết này không nhằm mục đích PR (nếu có thì chỉ là accident mà thôi), mà chỉ muốn kể lại quá trình mình build và grow sản phẩm này từ một bản prototype vớ vẩn trở thành một sản phẩm (vẫn vớ vẩn) phục vụ 50k lượt người dùng mỗi tháng.

## Disclaimer

Trước khi bắt đầu bài viết thì mình cũng xin phép set expectation một tí cho người đọc: Những gì mình viết ra dưới đây là kinh nghiệm rút ra từ trải nghiệm cá nhân, trên một sản phẩm không mấy nổi trội (chỉ là con kiến, nếu bạn tính so với các sản phẩm của các nhân vật nổi tiếng trong cộng đồng indie hackers như Tony Dinh hay Daniel Nguyen), và bản thân mình cũng chỉ là first time founder, tính tới thời điểm này chỉ có mỗi sản phẩm này là kiếm được tiền. Có thể sẽ có chỗ đúng, có thể sẽ có nhưng chia sẻ không đúng, các bạn đọc cho vui là chính.

## The prototype và cơn AI hype

Chuyện bắt đầu vào cuối tháng 3/2023, khi mình đang build một sản phẩm khác, lúc đấy mọi người bắt đầu nói nhiều đến LLM và Generative AI. Với bản tính tò mò thích khám phá (dịch là: mình đã bắt đầu chán cái product đang làm, chuẩn bị drop), mình dành ra một buổi chiều để làm một chiếc demo nho nhỏ, với mục đích tìm hiểu cách là việc với LLM (mà cụ thể là OpenAI API).

Và đây là kết quả, ngày 26/3 mình post lên Twitter một bản demo giúp convert yêu cầu của người dùng dưới dạng văn bản thành biểu đồ, sử dụng cú pháp Mermaid.js. Mình đặt tên nó là SequenceGenius, vì lúc đấy mình chỉ chỉ viết prompt để focus vào sequence diagram. Để sử dụng, user cần nhập vào OpenAI API key của họ.

![](https://notes.huy.rocks/posts/img/chatuml-prototype.png)

Về mặt kĩ thuật thì bản demo này không có gì phức tạp, bất cứ ai đã từng sử dụng OpenAI API (hay ChatGPT) thì đều có thể làm được trong vòng 30 phút. Nhưng bản demo này vẫn thu hút được một lượng tương tác nhất định trên Twitter. Có lẽ vì ở thời điểm đó, điều quan trọng nhất là AI hype, mọi người đều quan tâm. Rồi ChatGPT vẫn chưa support add-ons, mọi thứ liên quan đến AI vẫn chỉ đơn thuần là giao diện chat qua lại giữa người và máy, trong khi bản demo này thể hiện kết quả generate của AI dưới dạng hình ảnh nên mọi người khi xem qua thì thấy rõ tính ứng dụng của nó hơn.

## Vì sao muốn làm product kiếm tiền?

Thông thường thì đây là lúc mình dừng lại, publish code lên Github, và cái demo được cất vào kho, chờ ngày mọc rêu. Trước đây mình build rất nhiều thứ, nhưng không lần nào kiếm ra tiền, hoặc không có ai xài. Cũng chính vì thế nên mình có cái định kiến rằng: Mình không thể kiếm được tiền từ product đâu, làm cái gì thì chỉ cần open source ra là đủ rồi.

Nhưng lần này, mình lại nghĩ: Nếu quả thực khả năng kiếm tiền của mình bằng 0, thì có thử hay không nó cũng không có kết quả, vậy thì tội gì không thử? Bước vô game tay trắng thì cùng lắm lại trở về với trắng tay thôi.

## Monetize plan

Thế là mình rồi làm một cái form khảo sát gắn lên trang demo, đại ý để hỏi: Nếu giờ SequenceGenius thu phí $2/mo thì mọi người có xài không? Lúc đấy mình có khoảng 500 visitors một ngày, và có tổng cộng... 8 người reply. Trong đó chỉ có 2 người trả lời là "Có" 🤣 Cảm giác lúc đấy nó chán không thể tả được. Hơi mất tự tin một tí, nhưng đã muốn làm thì cứ làm thôi.

Rồi mình thảo ra một bản "Kế hoạch kinh doanh" trong đầu, kiểu như này:

Sẽ có 2 loại user: Free user và User trả phí. Đối với user trả phí, thì mình chọn cách monetize dựa trên credits. Họ sẽ không cần tự nhập API key nữa, thay vào đó họ sẽ có một lượng credit nhất định trong tài khoản, khi xài hết thì có thể trả tiền để mua thêm. Mình xác định sản phẩm này sẽ phục vụ user theo kiểu: người ta có nhu cầu generate diagram thì sẽ tìm tới để xài, xài hết credit cần thì mua thêm, không thì thôi. Giống như khi mình xài mấy cái dev tools online như JSON Formatter vậy. Đặt mục tiêu mỗi tháng có 100 users trả tiền, mỗi người 2 đô, vậy là được 200 đô, ngon lành. Để thu hút lúc ban đầu, thì mình sẽ cung cấp gói unlimited, không giới hạn số credit.

## Từ prototype thành "sellable prototype"

Thế là bắt tay vào tích hợp Supabase Login và Stripe. Lúc này vẫn chưa làm chức năng sign up mà chỉ có sign in thôi. Vì nghĩ bụng khi nào nhiều người xài rồi làm cũng không muộn. Tiếp theo sau đó là chỉnh sửa lại cái UI để nó ra dáng sản phẩm một tí, vì không thể nào đem bán với cái demo như thế được. Các bạn có thể xem hình bên dưới để thấy sự khác biệt giữa user trả phí và miễn phí.

Giao diện cho user miễn phí:

![](https://notes.huy.rocks/posts/img/chatuml-free-user-ui.png)

Giao diện cho user trả phí:

![](https://notes.huy.rocks/posts/img/chatuml-paid-user-ui.png)

## Đi chào bán sản phẩm và cú deal nghìn tỷ

Xong rồi thì mình mới đem lên WeBuild để quảng cáo, chính xác hơn là make a deal. Kiểu: "Mình muốn monetize cái sản phẩm này nè, mà chưa có ai xài hết. Giờ mình bán gói early bird, cho xài không giới hạn luôn, rồi kêu gọi anh em mua để ủng hộ, bán giá rẻ rề 2 đô một license thôi. Đổi lại anh em mua nhiều thì mình sẽ lấy con số doanh thu đó lên Twitter khoe để làm marketing."


![](https://notes.huy.rocks/posts/img/chatuml-first-payment.png)

Thế là mình lùa được 19 "chú gà con", người đầu tiên là [@monody](https://github.com/monodyle/). Đùa tí thôi, Đây là 38 đô quan trọng nhất đối với ChatUML, nếu không có số tiền ban đầu này thì mình nghĩ sẽ rất khó để grow được sản phẩm trong thời gian này. Chân thành cảm ơn anh em WeBuild rất rất nhiều.

## Đi bán thiệt, và bán được thiệt

Sau đó thì mình làm một trang landing page và bắt đầu mở bán gói early bird này ra public, đồng thời đổi tên sản phẩm từ SequenceGenius thành ChatUML (cái tên được [@unreal](https://github.com/unrealhoang) suggest). Có lẽ phần vì tò mò vì sao sản phẩm vớ vẩn thế mà vẫn bán được tiền, càng lúc càng có nhiều người mua.

## Về competitors

Trên Twitter lúc này mình cũng bắt đầu tìm ra một vài bạn đang build sản phẩm giống y hệt mình, và nhiều sản phẩm lớn, tên tuổi như LucidChart cũng bắt đầu thử nghiệm feature tương tự (sau này còn có cả Excalidraw và nhiều sản phẩm khác). Cảm giác ban đầu là khó chịu và rất lo. Nhưng sau một thời gian thì mình thấy đây là việc hết sức bình thường. Thứ nhất, về mặt technical, sản phẩm này không hề có moat 🥲 ai cũng build được nên việc này không có gì là lạ. Thứ hai, có nhiều competitors chứng tỏ là mình đang đi đúng hướng, đúng market. Thứ ba, mình có cơ hội quan sát xem đối thủ họ làm gì và outcome như thế nào, là một cách validate mà không cần phải bắt tay vào làm gì hết. Thứ tư, đối với các đối thủ lớn, mình sẽ có lợi thế là... nhỏ con, nên sẽ move nhanh hơn và mình có thể tự do thử nghiệm một cách aggressive hơn họ. Thứ năm, mỗi sản phẩm đều cung cấp một trải nghiệm riêng cho người dùng, và mỗi user sẽ có một nhu cầu khác nhau phù hợp với một loại sản phẩm khác nhau, nên luôn luôn sẽ có user sẵn sàng thử nghiệm sản phẩm của mình.

Nói sơ một tí về khác biệt của ChatUML so với các sản phẩm khác, thì hiện tại ít có sản phẩm vẽ diagram nào cho phép chỉnh sửa diagram dùng AI ngoài ChatUML. Lý do? Vì hầu hết các sản phẩm đó (Excalidraw, LucidChart, Draw.io,...) sử dụng AI để generate ra Mermaid.js, sau đó convert tiếp Mermaid.js về DSL của từng app (Excalidraw JSON hoặc Draw.io XML). Làm vậy, ưu điểm là user có thể thoải mái edit bằng tay sau khi generate (đây cũng là nhược điểm của ChatUML). Nhưng nhược điểm là không có cách nào để đưa trạng thái của diagram ở dạng DSL vào cho AI xử lý tiếp.

## Từ 0 đô đến 500 đô

Sau khi chạm mốc 100 đô thì mình bắt đầu bỏ chức năng cho phép người dùng tự sử dụng OpenAI API key cá nhân, bắt buộc tạo account để sử dụng, tăng giá, từ 2 đô lên 5 đô và sau đó là 8 đô. Lý do thực hiện những thay đổi kia chỉ là để test thử user xem đến mức nào thì mọi người quay lưng lại với sản phẩm, thế nhưng sau mỗi đợt tăng giá, thì càng có nhiều người mua hơn, chắc do FOMO, phần quan trọng chắc tại vẫn còn quá rẻ. Sau khoảng 1 tháng rưỡi thì mình chạm mốc 500 đô với khoảng 100 user.

![](https://notes.huy.rocks/posts/img/chatuml-500.png)

Suốt thời gian đó hầu như mình không code thêm được feature nào, mà phần lớn thời gian là để đi quảng cáo khắp nơi và để trả lời email từ user, fix bug. Có lúc 1, 2h sáng đang ngủ cũng phải bật dậy để gửi mail invite cho user khi họ vừa mới purchase (đến thời điểm này thì việc onboarding user vẫn được thực hiện bằng tay, theo kiểu: User purchase, mình nhận đc thông báo, sau đó thì login vào Supabase để tạo account và gửi mail invite).

Cũng trong giai đoạn này, mình nhận ra tầm quan trọng của việc nói chuyện với user. Khi xài một sản phẩm SaaS thì thứ mình ghét nhất là nhận đc mail của founder, nên mình nghĩ ai cũng như mình. Hóa ra vẫn có những user rất tâm huyết, họ test đủ mọi case và không ngại call trực tiếp để describe vấn đề mà họ gặp phải, người thì request thêm feature mới, nhưng cũng có rất nhiều người chửi mắng và chê bai, rất nhiều trong số đó là free users.

Nói sơ một chút thì trước khi run sản phẩm này, mình rất ngại và nghĩ mình rất dở trong việc marketing, ngại nói chuyện với người lạ. Đến bây giờ, thì mặc dù vẫn dở, nhưng mình nghĩ việc nói chuyện với người khác đôi lúc nó có cái gì đó rất enjoy.

Lúc này thì mình rủ [@chip](https://github.com/lqt93) tham gia phát triển cùng, cũng nhờ vậy mà tụi mình ship được rất nhiều chức năng quan trọng, như tăng cường độ ổn định của AI cũng như cơ chế rotate API key để tránh bị rate limit. Quan trọng hơn hết, @chip vào refactor phần lớn những những chỗ mình code ẩu, nhờ thế mà fix được một đống bug. Tiếc là thời gian sau đó thì doanh thu bắt đầu đi xuống, có rất ít user đặt mua và quỹ thời gian của mình càng lúc càng hạn chế (do phải lo chạy deadline của day job 😂) nên đành tạm gác lại dự án.

_(Kết thúc phần 1, mời các bạn đón đọc phần 2 kể về giai đoạn kết thúc mùa sale early bird, ProductHunt launch, vượt mốc 5k đô, cú đâm từ Supabase, và vấn nạn lừa đảo credit card/charge back)._
