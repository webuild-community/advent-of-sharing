---
title: Hãy nói về User Activation
author: Bần làm vườn
date: 12-02-2022
---
(Disclosure: Chưa được support hoàn toàn nhưng có trong spec, soon :party-cat:)

**Context:** Web standard ngày càng phát triển và unlock nhiều tính năng chúng ta có thể khai thác để tăng trải nghiệm người dùng. Có thể lấy ví dụ như [Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API), được bắt đầu implement vào khoảng 2018 và đến nay đã support gần như hoàn chỉnh bởi các browser lớn nhất, cực kì hữu ích. Nhưng mà như Uncle Ben nói với anh nhện thì "with great power comes great responsibility", các nhà làm web standard phải có cơ chế để hạn chế tính năng mới bị lợi dụng để gây hại cho người dùng web. Một trong những requirement cần phải có để có thể dùng các hàm của Clipboard API là phải tồn tại transient user activation. Hiểu nôm na là user vừa active ở đây xong.

**Mechanism:** Khi user tương tác với page hay một thành tố nào đó trên page, gây trigger một trong các event keydown mousedown pointerdown pointerup touchend thì browser sẽ giữ lại một timestamp. Và dùng timestamp đó để quyết định trạng thái user activation để unlock các tính năng có yêu cầu.

- _Transient user activation:_ transient có nghĩa là tạm thời, mọi người có thể hiểu như là "recently". Nếu khoảng thời gian tính từ hiện tại cho đến timestamp lúc tương tác cuối cùng < khoảng activation duration (do browser quyết định, nhưng mà theo spec có đề nghị is expected be at most a few seconds) thì tức là user đang active (UserActivation.isActive có giá trị là true), và có tồn tại trạng thái transient user activation :point_right: Thõa mãn điều kiện xài Clipboard API (nếu đã xin permission), và nhiều API khác.

- _Sticky user activation:_ sticky ở đây có nghĩa là nó là trạng thái tồn tài xuyên suốt và không có timeout. Chỉ cần user có tương tác ít nhất một lần là sẽ có trạng thái này (UserActivation.hasActive có giá trị là true)

**Conclusion:** Nếu lỡ có một ngày đẹp trời nào đó các bạn dùng API mới mà không đọc doc (shame on you :bell:) và không hiểu sao API nó không work như các bạn kì vọng thì đây là một chỗ để các bạn dive vào. Nhớ check trạng thái trước khi dùng và có fallback hợp lí :pepewink:

=== Phần râu ria, skip cũng được ====

**Extra note:** Một số API khi dùng sẽ consumed hoàn toàn trạng thái transient user activation, tức là API đó xài rồi thì nó sẽ reset hoàn toàn, những API sau sẽ không dùng đc nữa dù vẫn còn trong duration cho phép. Lúc này user sẽ phải tương tác lại để có trạng thái activation mới.

**Extra note 2:** Off topic, nhưng mà behind the scene phần timestamp mình có nhắc đến ở trên nó có 2 giá trị đặc biệt, vô cực dương để represent là chưa từng có tương tác, và vô cực âm để represent là nó đã bị consume như note ngay trên.

**Extra note 3:** Lúc check các event để lưu timestamp tương tác thì nó sẽ có check thêm thuộc tính Event.isTrusted , thuộc tính này là read-only và chỉ tồn tại khi nó event đc tạo ra bởi chính user, thay vì là một hàm nào đó.
