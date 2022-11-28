---
title: Dùng Radix Primitives với Tailwind như nào?
author: hai
date: 11-25-2022
---
Radix components được cung cấp dưới dạng thô, cho bạn toàn quyền kiểm soát việc tô vẽ CSS.

Bạn có thể dùng bất cứ giải pháp CSS nào cũng được (vanilla CSS, CSS preprocessors, CSS-in-JS).

Ở đây mình chia sẻ cách sử dụng cùng Tailwind CSS.

Vẫn như thường lệ, ở đâu có `className`, ở đó có thể dùng Tailwind. Nhưng bạn sẽ gặp trường hợp như sau đối với Radix components:

![image](https://user-images.githubusercontent.com/613943/204167855-d138393b-c710-4c96-9a9d-cdabf850ce53.png)

Một số Radix components là stateful, nên state của chúng được thể hiện thông qua thuộc tính `data-state`.

Ví dụ như `Checkbox.Root` có `[data-state] = "checked" | "unchecked" | "indeterminate"`

Để thuận tiện hơn trong việc sử dụng cùng Tailwind, thay vì dùng arbitrary value, bạn có thể config như sau:

![image](https://user-images.githubusercontent.com/613943/204167923-f3b2b466-6805-42b2-b902-d1f757b67852.png)

Với cách cấu hình như trên, việc quản lý và tốc độ styling sẽ được cải thiện hơn rất nhiều.
