---
title: Reduced Motion
author: nvh95
date: 11-28-2022
---
:runner: Animation/ Motion là một phần rất quan trọng trong các ứng dụng web, khiến các trang web trở nên sống động, thú vị và “thật” hơn rất nhiều. Tuy nhiên, nhiều người sẽ không thể chịu được việc có quá nhiều chuyển động, sẽ khiến họ bị chóng mặt, hoa mắt, đau đầu hay buồn nôn.

:brain: Bình thường thì hệ tiền đình (gồm tai trong và các bộ phận của não) sẽ giúp chúng ta cân bằng. Tuy nhiên đối với nhiều người, animation có thể sẽ khiến hệ tiền đình của họ bị mất cân bằng và gây ra chứng rối loạn tiền đình (vestibular disorders), tạo nên cảm giác khó chịu cho người dùng, mà triệu chứng điển hình nhất là chóng mặt.

:question: Vậy, là một frontend developer, chúng ta cần chú ý đến chi tiết này để không gây ra cảm giác khó chịu cho người dùng. Nhưng chúng ta cũng không thể bỏ hết các motion của một trang web đi được, như thế đối với phần lớn những người dùng có hệ tiền đình ổn định, họ sẽ không có trải nghiệm tốt nhất.

👩‍💻 Rất may là CSS có một media query là `prefers-reduced-motion`. Media query này sẽ kiểm tra xem người dùng có muốn “giảm” chuyển động của trang web không. Settings này sẽ dễ dàng thiết đặt trong hệ điều hành (ảnh 2). Cú pháp của “prefers-reduced-motion” như sau:

```css
.box:hover {
  transform: scale(1.2);
}

@media (prefers-reduced-motion: reduce) {
  .box {
    transition: none;
  }
}

@media (prefers-reduced-motion: no-preference) {
  .box {
    transition: transform 300ms;
  }
}
```

:gift: Ở đoạn code trên, khi hover vào cái hộp (box), nếu người dùng muốn giảm chuyển động, chiếc hộp sẽ to lên 1.2 lần ngay lập tức. Còn bình thường, nó sẽ to lên 1.2 lần trong 300 ms. Cách này, phần lớn người dùng sẽ có trải nghiệm có animation, còn phần ít hơn số người dùng lựa chọn reduced motion, sẽ không bị chóng mặt.

:movie_camera: Video đính kèm mô tả tính năng này đến từ phần mềm rất quen thuộc là Slack. Khi chúng ta kích hoạt reduced motion, nếu có tin nhắn mới, thông báo sẽ hiện dần dần lên. Còn bình thường, thông báo sẽ đi từ phải trang trái.

:sunglasses: Đây là những tiểu tiết nhỏ, nhưng sẽ là những phần kiến thức giúp bạn trở thành một frontend developer tốt hơn. Luôn đặt mình vào vị trí của người dùng (cụ thể trong trường hợp này là những người dùng gặp chứng rồi loạn tiền đình) sẽ giúp chúng ta tạo ra được những ứng dụng web chất lượng tốt và thân thiện với người dùng.

https://user-images.githubusercontent.com/613943/204171890-1ff95622-a23c-45a0-8305-2a08d1ce339c.mp4

https://user-images.githubusercontent.com/613943/204171900-e0378bb5-8d68-4bb6-b8ff-73b2c9474226.mp4

Nếu dùng Mac các bạn có thể activate Reduced Motion ở đây:

![Navigating to Accessibility in Mac Settings to activate Reduced Motion](https://user-images.githubusercontent.com/613943/204170698-6f50d4aa-470d-4df6-ba87-991e8d95d806.png)

