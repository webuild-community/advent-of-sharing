**Author:** An

---

# Animated SVG Icon with Framer Motion

[Animated SVG icon with Framer Motion](https://an.cyou/blog/animated-icon-with-framer-motion)

![](https://an.cyou/favicon.png)

[https://an.cyou/blog/animated-icon-with-framer-motion](https://an.cyou/blog/animated-icon-with-framer-motion)

![](https://ogcool.vercel.app/templates/v1/tpl_TnllPg0CkP?d=eyJtb2RpZmljYXRpb25zIjpbeyJuYW1lIjoiVGV4dCIsInRleHQiOiJBbmltYXRlZCBTVkcgaWNvbiB3aXRoIEZyYW1lciBNb3Rpb24ifV19&sdk=ogcool%400.1.10)

---

Lần đầu tiên mình thấy SVG icon động là khi Resend.com ra mắt. Chúng giúp mình cảm giác bớt sự đơn điệu của dashboard khi chỉ toàn màu đen. Lúc đó, mình đã tự hỏi bao giờ mình có thể làm được thứ tương tự.

Qua hai năm, sau nhiều bài viết trên X, khóa học như [animations.dev](https://animations.dev/), hay các bài chia sẻ của [nandafyi](https://x.com/nandafyi), mình nghĩ mình đã hiểu thêm về cách SVG hoạt động.

Quay lại vài tháng trước, mình đã thử nghiệm với vài icon từ lucide-react để xem liệu việc animate icon có khó không. Và …nó không khó như mình tưởng: [/experiments](https://an.cyou/experiments).

### [Animate Icon](https://an.cyou/blog/animated-icon-with-framer-motion#animate-icon)

Trước khi đi sâu vào cách animate icon, bạn có lẽ muốn tìm hiểu thêm về SVG qua [Bài chia sẻ về SVG](https://github.com/xuanvan229/advent-of-sharing/blob/dc2a06faffd207798be783b09beee8766d321449/2023/day-24.md) từ AOS 2023.

Trong bài viết này, thay vì sử dụng builtin animation của SVG hay dùng CSS để animate, chúng ta sẽ sử dụng Framer Motion để tạo hiệu ứng cho icon. Framer Motion là 1 thư viện animation mạnh, dễ dùng, kể cả với người mới.

Dưới đây là một file SVG đơn giản:

```svg
<!-- Loading Plain Text code... -->
<!-- (Placeholder for the simple SVG file mentioned in the article) -->
```

Thành phần chính của SVG này là `<path>`, trong đó `d` (path data) mô tả cách path được vẽ.

Để thêm hiệu ứng cho icon, mình sử dụng attribute [`pathLength`](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/pathLength). Hiểu đơn giản, nó cho phép “vẽ” path từ 0% đến 100% mà không cần chúng ta xử lý từng điểm. Kết hợp với Framer Motion, bạn có thể đạt được hiệu ứng như sau:

```tsx
import { motion } from "framer-motion";

export default function Example1() {
  return (
    <>
      Default:
      <br />
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        strokeLinecap="round"
        strokeLinejoin="round"
        className="w-24 h-24"
      >
        <path d="M3 3v16a2 2 0 0 0 2 2h16" />
        <path d="M7 16h8" />
        <path d="M7 11h12" />
        <motion.path d="M7 6h3" />
      </svg>

      <br />
      <br />

      Animated:
      <br />
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
      >
        {/* Animated paths would be defined here */}
      </svg>
    </>
  );
}
```

#### [Cách hoạt động](https://an.cyou/blog/animated-icon-with-framer-motion#c%C3%A1ch-ho%E1%BA%A1t-%C4%91%E1%BB%99ng)

- Đổi qua dùng `motion.path` import từ Framer Motion thay vì `path` thông thường để dùng các function animation của Framer.
- Định nghĩa animation cho mỗi path:

  - `initial`: Trạng thái ban đầu, ví dụ `pathLength: 0` (path chưa được vẽ).
  - `animate`: Các trạng thái animation mà path sẽ đi qua.
  - `transition`: Cấu hình thêm cho animation như `duration`, `delay`, hoặc `repeat`.

Trong ví dụ trên, có ba path với các animation khác nhau:

- Path 1: `pathLength: [0, 1, 0]` — vẽ từ 0 đến 100% rồi reset về 0.
- Path 2: `pathLength: [0, 1, 1.2, 1, 0]` — vẽ từ 0 đến 100%, vượt quá rồi quay lại và reset.
- Path 3: `pathLength: [0, 1, 1.2, 1.4, 1.2, 1, 0]` — tương tự hai cái trên nhưng sẽ phức tạp hơn xíu.