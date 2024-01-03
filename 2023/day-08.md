---
title: Bàn về developer productivity
author: quannt
date: 2023-12-08
---

Năng suất lao động (productivity) luôn là một chủ đề được quan tâm. Từ thời tiền sử sống trong xã hội bầy đàn, tiến lên xã hội hiện đại với cách mạng công nghiệp; rồi gần đây hơn là thời đại của máy tính, internet, block chen, AI,... loài người luôn ám ảnh với việc làm được nhiều hơn, sản xuất nhiều hơn, nhanh hơn. Ngành phần mềm cũng không phải ngoại lệ.

Vậy productivity trong ngành phần mềm được định nghĩa như thế nào?

Câu hỏi nghe đơn giản nhưng câu trả lời lại khá phức tạp. [Một research gần đây][1] tại Microsoft cho thấy developer có xu hướng nghĩ về input: số bug/task đã hoàn thành, số lượng commit, thời gian tập trung cho deep work, thời gian teamwork khi được hỏi về productivity. Trong khi các manager có xu hướng tập trung vào output như performance, velocity, cũng như chất lượng của sản phẩm được tạo ra. Và nếu bạn hỏi những người ở vị trí non-tech như CEO, CFO, khả năng cao họ chỉ quan tâm team phần mềm làm được ra bao nhiêu tiền và đốt bao nhiêu vào quỹ lương.
Như rất nhiều thứ khác trong ngành phần mềm, it depends. Sẽ không thể tìm được một câu trả lời chung nào cho định nghĩa về productivity để áp dụng cho tất cả các team. Vậy thì các công ty phần mềm lớn nhỏ ngoài kia họ đang làm thế nào để đo lường productivity, hãy điểm qua một số cách thông thường.

Cách thứ nhất: không làm gì cả. Nghe có vẻ hơi anti-pattern, nhưng khá phổ biến ở các team nhỏ, ở những team này manager có thể nắm khá rõ về khối lượng công việc cũng như output của từng thành viên, cũng như của cả team, việc cụ thể hóa thành con số không có nghĩa lý và ít mang lại hiệu quả.

Cách thứ hai: vậy team lớn thì họ dùng gì, hiện tại trong giới có hai tiêu chuẩn khá nổi là [DORA][2] và [SPACE][3]. DORA đề ra bốn metric cho một engineering team, focus chính vào việc delivery phần mềm

- Deployment Frequency – tần suất release lên production.
- Lead Time for Changes – thời gian dành ra từ khi bắt đầu đến khi release của ticket.
- Change Failure Rate – tỉ lệ các release tạo lỗi trên production.
- Time to Restore Service – thời gian để fix production khi có release lỗi.
trong khi SPACE đề ra năm tiêu chí,
- Satisfaction and wellbeing: mức độ hạnh phúc và thỏa mãn của các thành viên trong team.
- Performance: chất lượng đầu ra của sản phẩm.
- Activity: số lượng output/task hoàn thành.
- Communication and collaboration: mức độ teamwork và collab giữa các thành viên trong team.
- Efficiency and flow: thời gian focus vào deep work của team.
Điểm cộng của DORA là các metric được định nghĩa rõ ràng, điểm trừ là tập trung quá nhiều vào việc delivery và hoàn toàn bỏ qua yếu tố con người. Team của mình hiện tại cũng đang dùng DORA và tác dụng khá hạn chế, khó nêu được điểm yếu cần cải thiện, và các metric khá dễ bị game, có thể dẫn đến phản tác dụng.
SPACE được ra đời để khắc phục nhược điểm của DORA, đề cao input và yếu tố con người, nhấn mạnh việc lấy survey từ các thành viên trong team, nhưng nhược điểm lại là các metric phức tạp, mơ hồ và cũng khó đo đạc báo cáo.

Cách thứ ba: tự đặt ra metric riêng cho mình, thường được áp dụng ở các big tech, ví dụ như [Uber][4], Meta, Amazon đề cao số lượng diff (commit),  [Linkedin][5] dùng top-down approach, bắt đầu với ba key metric Efficiency, Effectiveness, Happiness sau đó đi sâu vào và mở rộng ra các key metric nhỏ hơn như time to review, user satisfaction score, commit to publish time, etc. Việc xây dựng hệ thống để thu thập và báo cáo số liệu thường được đảm nhiệm bởi một team gọi là Developer Experience.

Tạm kết ở đây đã vì chủ đề này phức tạp và nếu nói nữa thì chắc phải viết cả series, bài viết cũng chỉ nhằm mục đích giới thiệu những khái niệm cơ bản ở bề mặt. Tóm lại thì mình thấy nhu cầu đo đạc productivity là chính đáng, tuy nhiên đo thế nào và đo cái gì là bài toán các team phần mềm sẽ phải tự tìm cho câu trả lời của riêng mình. Hãy nhớ rằng đằng sau các metric và con số là những con người, họ cũng có buồn vui hỉ nộ ái ố, và những con số thì khó có thể model hết hành vi của con người.

[1]: <https://arxiv.org/pdf/2111.04302.pdf>
[2]: <https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance>
[3]: <https://queue.acm.org/detail.cfm?id=3454124>
[4]: <https://newsletter.pragmaticengineer.com/p/uber-eng-productivity>
[5]: <https://newsletter.pragmaticengineer.com/p/linkedin-engineering-efficiency>
