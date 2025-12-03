**Author:** huytd

---

# What's the OKLCH? | Huy's Blog

Cách đây 2 năm Advent of Sharing 2022 mở màn với bài What's the HSL? viết về màu sắc, năm nay vinh dự được mở màn AoS 2024, mình cũng sẽ tiếp tục với chủ đề màu sắc.

![icon](https://notes.huy.rocks/apple-touch-icon.png)

https://notes.huy.rocks/posts/what-the-oklch.html

---

Cách đây 2 năm Advent of Sharing 2022 mở màn với bài [What's the HSL?](https://github.com/webuild-community/advent-of-sharing/blob/main/2022/day-01.md) viết về màu sắc, năm nay vinh dự được mở màn AoS 2024, mình cũng sẽ tiếp tục với chủ đề màu sắc.

[OKLCH](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch) là một hệ màu mới được đưa vào [CSS Color 4 specification](https://www.w3.org/TR/css-color-4/), đã được ứng dụng rất nhiều, một ví dụ điển hình đó là TailwindCSS từ phiên bản 4 đã chuyển bảng màu mặc định [từ](https://tailwindcss.com/docs/v4-beta#modernized-p3-color-palette) [rgb()](https://tailwindcss.com/docs/v4-beta#modernized-p3-color-palette) [sang dùng](https://tailwindcss.com/docs/v4-beta#modernized-p3-color-palette) [oklch()](https://tailwindcss.com/docs/v4-beta#modernized-p3-color-palette).

Trước khi nói về chủ đề chính thì chúng ta hãy nói về khái niệm không gian màu (color space), đây là thuật ngữ để chỉ phạm vi màu sắc mà một thiết bị có thể tái hiện được (ví dụ như màn hình, các thiết bị in ấn, máy ảnh,...). Vậy nên, không gian màu càng rộng thì hình ảnh được tái hiện sẽ càng sống động, chân thực.

NTSC là không gian màu chuẩn của thời kỳ truyền hình analog, và từ hàng chục năm nay thì sRGB là không gian màu tiêu chuẩn cho các thiết bị kĩ thuật số như màn hình, máy ảnh. Adobe RGB và CMYK thường được dùng trong in ấn. Và DCI-P3 là không gian màu đước sử dụng nhiều trong lĩnh vực phim ảnh. Về sau Apple phát triển một phiên bản P3 của riêng họ và đặt tên là Display P3, sử dụng trong màn hình máy mac. Phần lớn các thương hiệu màn hình máy tính sau này đều đã support Display P3.

Nhìn vào biểu đồ bên dưới, các bạn có thể thấy được sự khác biệt về phạm vi thể hiện màu sắc giữa các không gian màu khác nhau (các vùng hình tam giác). Còn cái vùng bao ngoài cùng là phạm vi màu sắc mà mắt thường có thể quan sát được.

![color space compare](https://notes.huy.rocks/posts/img/oklch/color-space-compare.png)

Image source: [BenQ](https://www.benq.com/en-us/knowledge-center/knowledge/what-is-dci-p-color-space.html)

Rồi nhé, giờ quay lại chủ đề chính, OKLCH.

Các hệ màu thường dùng trong CSS như hex hay rgb, HSL đều được dùng để diễn đạt màu trong không gian sRGB. Hệ màu OKLCH cho phép chúng ta diễn đạt màu sắc trong không gian P3. Điều này có nghĩa là chúng ta có thể tạo ra những trang web có màu sắc sống động hơn, nhìn đã con mắt hơn.

## Cú pháp màu OKLCH

Một giá trị màu trong hệ OKLCH có cú pháp như này:

```css
color: oklch(<lightness> <chroma> <hue> / <alpha>);
```

Trong đó 3 tham số `L`, `C`, `H`, và tham số `A` lần lượt là:

- **L** – Lightness (độ sáng), quyết định độ sáng tối của màu  
- **C** – Chroma (sắc độ), quyết định độ tươi hay tái của màu  
- **H** – Hue (tông màu), yếu tố chính quyết định màu sắc, là màu trên [color wheel](https://en.wikipedia.org/wiki/Hue) giống như khi dùng HSL  
- **A** – Alpha (độ trong suốt)

Theo như quảng cáo thì cách diễn đạt này gần với cách mà mắt người cảm nhận màu sắc hơn. Và khi làm việc với màu OKLCH, việc thay đổi cũng sẽ đơn giản hơn, giống như ví dụ dưới đây, chúng ta có thể chỉnh độ đậm nhạt của màu bằng cách thay đổi tham số lightness:

![oklch lightness](https://notes.huy.rocks/posts/img/oklch/oklch-lightness.png)

```css
.bg-custom {
  background-color: oklch(0.7 0.15 250);
}

.bg-custom-light {
  background-color: oklch(0.9 0.15 250);
}

.bg-custom-dark {
  background-color: oklch(0.5 0.15 250);
}
```

Hoặc độ tươi của màu, bằng cách thay đổi giá trị chroma:

![oklch chroma change](https://notes.huy.rocks/posts/img/oklch/oklch-chroma-change.png)

```css
.color-muted {
  color: oklch(0.7 0.02 250);
}

.color-normal {
  color: oklch(0.7 0.08 250);
}

.color-vivid {
  color: oklch(0.7 0.16 250);
}
```

## Gradient với OKLCH

Một đặc tính đáng chú ý khác của OKLCH đó là khả năng thể hiện hiệu ứng màu gradient tự nhiên, khác với khi làm gradient trên sRGB, chúng ta thường thấy có một vùng xám ở phần giao giữa 2 điểm màu, trên OKLCH không bị hiện tượng này:

![oklch gradient](https://notes.huy.rocks/posts/img/oklch/oklch-gradient.png)

```css
background: linear-gradient(
  90deg,
  oklch(0.7 0.16 250),
  oklch(0.7 0.16 340)
);
```

## Tóm lại

Hệ màu OKLCH đem lại khá nhiều những lợi ích, như là:

- Gần với cách mà mắt thường cảm nhận màu sắc hơn  
- Hỗ trợ dải màu dài hơn, cho phép chúng ta thiết kế với màu sắc sống động và phong phú hơn  
- Chỉnh sửa màu (thay đổi độ sáng tối, tương phản, tạo bảng màu tự động) dùng CSS dễ dàng hơn  
- Gradient xịn màu hơn  

Mặc dù đây là một hệ màu còn rất mới, các công cụ thiết kế như Figma, hay các CSS framework vẫn chưa hỗ trợ đầy đủ, nhưng hiện tại OKLCH đã được hỗ trợ trên tất cả mọi trình duyệt. Nếu bạn đang làm frontend, thì bạn đã biết cách làm gì để được tăng lương rồi đấy ;)

## Đọc thêm

- https://evilmartians.com/chronicles/oklch-in-css-why-quit-rgb-hsl  
- https://uploadcare.com/blog/oklch-in-css/  
- https://lea.verou.me/blog/2020/04/lch-colors-in-css-what-why-and-how/