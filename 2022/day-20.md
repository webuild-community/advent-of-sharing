![image](https://user-images.githubusercontent.com/19520278/207461165-ee850d3d-ee70-45bd-826a-eaf00ed90c4e.png)

## Tổng quan

Trước khi có variable fonts, mỗi kiểu chữ trong 1 loại font được chia thành file riêng biệt.

Với variable fonts, tất cả các kiểu chữ có thể được chứa trong 1 file.

```sh
. Traditional fonts
├── static/Inter-Thin.ttf
├── static/Inter-ExtraLight.ttf
├── static/Inter-Light.ttf
├── static/Inter-Regular.ttf
├── static/Inter-Medium.ttf
├── static/Inter-SemiBold.ttf
├── static/Inter-Bold.ttf
├── static/Inter-ExtraBold.ttf
├── static/Inter-Black.ttf
└── static/Inter-Thin.ttf

. Variable fonts
└── Inter-VariableFont_slnt,wght.ttf
```

Variable fonts cung cấp một tập hợp các `axes` (số nhiều của axis) khác nhau tuỳ vào người thiết kế và phát triển font.

Thường chia thành hai loại axes:

- Axes có sẵn (registered axes) gồm `weight`, `width`, `optical size`, `slant` và `italics`
- Axes tuỳ biến (custom axes) có số lượng phụ thuộc vào sự sáng tạo của người thiết kế

Để phân biệt với registered axes, tag của custom axes sẽ được viết hoa.

| Axis name    | CSS value |
| ------------ | --------- |
| Weight       | wght      |
| Width        | wdth      |
| Slant        | slnt      |
| Optical Size | opsz      |
| Italics      | ital      |

## Tại sao gọi là axis?

Để dễ hình dùng hơn ta hãy dùng hệ tọa độ mặt phẳng XY làm ví dụ.

- Trục X = weight, có giá trị từ 300 đến 1000
- Trục Y = casual, có giá trị từ 0 đến 1

Với mỗi tọa độ khác nhau sẽ cho ra một kiểu chữ khác nhau.
Mỗi variable font sẽ có hệ trục khác nhau, và giá trị của mỗi trục cũng khác nhau luôn.

Multiverse of axis <=> more axes more styles.

![image](https://user-images.githubusercontent.com/19520278/207461448-ed290dfe-ab62-47fa-a3bd-f82f9f75ee00.png)

## Làm sao biết hết axes của font?

Theo kinh nghiệm cá nhân thì có 3 cách:

- Dùng tool như [Wakamai Fondue](https://wakamaifondue.com/)
- Dùng Type Tester trên Google Fonts (chỉ áp dụng với fonts được host trên này)
- Dùng Safari Devtool là nhanh nhất (Chrome Devtool và Firefox Devtool không có)

![Safari Devtool](https://user-images.githubusercontent.com/19520278/207461751-a39fb548-8822-4c0b-89e9-fef32d50cdf5.png)

![Google Fonts Type Tester](https://user-images.githubusercontent.com/19520278/207461869-500820ef-e1bc-4fb1-86a0-577e53473735.png)

## Tại sao nên dùng variable fonts?

- Tiết kiệm dung lượng tới mức đáng kể so với tổng các file fonts gộp lại.
- Tăng trải nghiệm và lựa chọn thiết kế với nhiều kiểu chữ khác nhau.
- Giảm độ trễ network request => tăng thời gian render trang web.
- Font animation ([Anicons](https://typogram.github.io/Anicons))

## Sử dụng với CSS

Variable fonts cũng tương tự traditional fonts về cơ chế khai báo, tùy vào tooling và hosting sẽ có cách khai báo khác nhau.

```tsx
import { Recursive } from "@next/font/google";

const recursive = Recursive({
  display: "swap",
  axes: ["slnt", "MONO", "CRSV", "CASL"],
});
```

Ví dụ trên sử dụng `@next/font` để khai báo Recursive với 1 registered và 3 custom axes.

### Dùng registered axes

- `wght` axis có thể được set bằng `font-weight`, thay vì các keyword như _medium|semibold|bold_, hãy thoải mái dùng giá trị nằm trong range cho phép.
- `wdth` axis thì dùng `font-stretch`
- `ital` axis thường mang giá trị on/off, dùng `font-style: italic/normal`
- `slnt` axis cũng là chữ nghiêng nhưng không phải italic, nó khiến cho chữ nghiêng theo 1 góc nhất định, dùng `font-style: oblique <angle>`
- `opsz` axis khá hiếm dùng, hữu ích cho việc thay đổi chi tiết ký tự ở các kích thước khác nhau, dùng `font-optical-sizing`

Tuy nhiên có những font bị hạn chế bởi CSS Specs chưa đáp ứng được, nên họ sẽ set giá trị axis qua `font-variation-settings` giống như custom axes

```css
.between-regular-and-medium {
  /* Should be font-weight: 450; */
  font-variation-settings: "wght" 450;
}

.stretch {
  /* Should be font-stretch: 22; */
  font-variation-settings: "wdth" 22;
}

.italic {
  /* Should be font-style: italic; */
  font-variation-settings: "ital" 1;
}

.slanted {
  /* Should be font-style: oblique 10deg; */
  font-variation-settings: "slnt" 10;
}

.optical-size {
  /* Should be font-optical-sizing: 8pt; */
  font-variation-settings: "opsz" 8;
}
```

### Dùng custom axes

Chỉ có `font-variation-settings` mới đáp ứng được các giá trị của custom axes. Ví dụ Roboto Flex có Grade axis (`GRAD`).

```css
.grade-light {
  font-variation-settings: "GRAD" -200;
}

.grade-normal {
  font-variation-settings: "GRAD" 0;
}

.grade-heavy {
  font-variation-settings: "GRAD" 150;
}
```

## Gỡ rối CSS

Một vấn đề của `font-variation-settings` đó là không có tính kế thừa. Axis nào mà không được khai báo thì sẽ tự động reset về giá trị mặc định.

Cùng xem ví dụ dưới đây:

```html
<span class="slanted grade-light">
  I should be slanted and have a light grade
</span>
```

Đầu tiên là `font-variation-settings: 'slnt' 10` được apply, sau đó là `font-variation-settings: "GRAD" -200`, tuy nhiên `slnt` axis sẽ bị reset về 0 vì chỉ nhận được giá trị của `GRAD`. Kết quả là đoạn text trên sẽ không có độ nghiêng mong muốn.

Để giải quyết vấn đề này cần dùng tới CSS Variables:

```css
:root {
  --slnt: 0;
  --GRAD: 0;
}

.slanted {
  --slnt: 10;
}

.grade-light {
  --grad: -200;
}

.slanted,
.grade-light {
  font-variation-settings: "slnt" var(--slnt), "GRAD" var(--GRAD);
}
```
