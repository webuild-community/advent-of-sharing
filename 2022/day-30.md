---
title: SSR Hydration
author: thanhle
date: 12-24-2022
---

Đầu tiên phải tìm hiểu làm tươi là làm gì đã

[https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056522/patterns.dev/prog-rehy-2.mp4](https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056522/patterns.dev/prog-rehy-2.mp4)

> In web development, hydration or rehydration is a technique in which client-side JavaScript converts a static HTML web page, delivered either through static hosting or server-side rendering, into a dynamic web page by attaching event handlers to the HTML elements.

Với việc hầu hết framework frontend hiện tại đều support SSR giúp tối ưu FCP First (Contentful Paint) và SEO nhưng lại trade off là TTI (Time to interact) khá tệ và và cực kì khó để tối ưu. Nó giống như kiểu website nhìn load nhanh đó nhưng click vào button nào cũng không có phản hồi gì cả. Vì đâu ra nông nỗi này?

## Do đâu cần hydration?

Việc support SSR của các Frontend framework về cơ bản là như này

Component + State ⇒ HTML code rồi send cho browser

Tuy nhiên, môi trường server nodejs lại rất khác nên quá trình ở trên chỉ có thể output ra HTML mà không thể “đính kèm” với các event listener tương ứng với từng node.

Do đó, sau khi browser nhận HTML code và render cho user, nó cần chạy lại code để biết cái event listener này gắn vào dom node nào. Quá trình hydration này là bắt buộc, vì nếu không có thì mọi interact từ user coi như bỏ đi.

## Cái hydration khá take time

![Untitled (6)](https://user-images.githubusercontent.com/9281080/209437846-78656b22-572a-4a72-acec-6fa7e5efd6be.png)

- Browser nhận HTML từ SSR process, render ra cho user
- Download JS file tương ứng
- Parse JS
- Execute js đã được parse để biết được gắn event listener để gắn vào cái DOM node tương ứng
- Done

### Một số cách tối ưu Hydration

Có 3 cách để tối ưu hydration

- Break long task thành nhiều small task, để user interaction có thể chen giữa vào các small task đó
- Lazy load/run hydration. Thực tế là users nhiều khi chỉ interact với vài phần chính của website nên nếu chỉ run hydration nhưng phẫn mà user tương tác thì sẽ tối ưu hơn.
- Bỏ mọe cái concept hydration này đi

### React suspense

Thực ra vụ suspense sẽ giải quyết nhiều vấn đề hơn là chỉ tối ưu cho quá trình Hydration. Nhưng suspense có cung cấp cho bạn cớ chế để break long task hydration thành nhiều task nhỏ async.

Cách apply khá dễ, lâu lâu wrap component trong React.suspense là xong, với điều kiệu bạn đang chạy React 18

[https://twitter.com/dan_abramov/status/1529688977413898241](https://twitter.com/dan_abramov/status/1529688977413898241)

### Island architecture

[https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056521/patterns.dev/prog-rehy-5.mp4](https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056521/patterns.dev/prog-rehy-5.mp4)

Concept khá simple & power: Component thằng nào thằng đó lo. Nghĩa là mình ship full HTML xuống, nhưng có thể chọn được component nào được hydration, và thứ tự hydration ⇒ Không phải chờ full website được hydration thì mới tương tác được.

Một điểm hơi khác nữa so với React suspense ở trên là mình control được khi nào download js bundle của component tương ứng và run hydration, thay vì React suspense vẫn phải download full js về

[Có thể thử với Astro](https://astro.build/)

### Resumability & Serialization

![Untitled (7)](https://user-images.githubusercontent.com/9281080/209437852-433bb2c2-bcbd-41b5-a91d-bfeafb28453b.png)

Thằng đưa ra concept này là [Qwik](https://qwik.builder.io/docs/think-qwik/). Về cơ bản, phải chạy hydration là vì không thể ship được code JS nào (WHAT) gắn vào cây DOM nào (WHERE) nên mới phảy chạy lại JS để biết điều đó.

Qwik giải vụ này bằng việc Serialize chỗ nào có event và gắn vào HTML ship xuống cho browser luôn. Sau đó, sẽ có 1 đoạn script JS nhỏ (1kb) listen tất cả event, gặp thằng nào được gọi thì Fetch js tương ứng về rồi run luôn.

Ưu điểm hơn Island architecture vì kiểu gì thì mình cũng không cần chạy lại code để hydration. Tradeoff là mỗi first interaction sẽ có một ít delay vì phải download JS bundle tương ứng.

Hy vọng qua bài viết này, bạn biết được tradeoff khi dùng SSR và các approach khá hay ho khi bạn đang muốn build một website có nhiều interaction
