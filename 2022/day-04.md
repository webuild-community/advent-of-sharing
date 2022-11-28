---
title: Clone website một cách có hiệu quả với Vite
author: ducan-ne
date: 11-28-2022
---
Trong cuộc sống thường ngày của FE dev có trường hợp cần clone 1 website, ko kể website đấy sử dụng framework hay stack gì, đôi khi đó là khách hàng yêu cầu hay là công ty yêu cầu.

Theo như bình thường chúng ta sẽ có 2 cách để clone:

- Clone bằng cách nhìn từng component và rewrite lại từng phần
- Clone bằng cách Ctrl S

Hôm nay mình sẽ chia sẻ hướng clone website bằng Ctrl S, nhưng theo 1 cách hiệu quả hơn.

Từ khi [Vite@3 chuyển qua parse5](https://twitter.com/patak_dev/status/1564265006627176449) (thư viện parse html giống với cách của trình duyệt), chúng ta có thể clone website và custom website đấy theo 1 cách có hiệu quả, các bước như sau:

- Tìm website bạn cần clone
- Ctrl S/ Cmd S để download toàn bộ website + resources xuống local
- Tạo 1 project vite bằng lệnh npx create-vite hoặc bất kì cách nào bạn muốn.
- Khi này file html mà bạn download xuống đã parseable bởi Vite và việc bạn cần làm là copy file html vào index.html, folder "XXX files" vào folder public của vite đã tạo ở trên.
- Khi này vite đã có thể parse file html của bạn và tạo 1 dev server bằng lệnh vite , vậy tại sao chúng ta lại cần làm thế này thay vì mở trực tiếp file trên browser và code, có 1 số điểm lợi khi dùng dev server như sau:

Điểm lợi :star:

- Hot reload + Live reload
- Host trên localhost thay vì mở file url trên browser, dùng link file trực tiếp gây ra 1 số vấn đề  khi hosting (origin, call api, etc)
- Dễ dàng deploy lên FE hosting platform (Ví dụ Vercel)
- Nếu sử dụng Vercel việc từ zero to url ready (hoặc nếu sử dụng custom domain) chỉ trong vòng 10 phút.
- Customizable cloned website với bundler, ví dụ thêm react app vào trang web đó.
- Sử dụng vite plugins.
- Sử dụng tailwind + jit

Thay vì nhiều lời chi bằng đọc code, đây là 1 repo mình clone trang landing của supabase: https://github.com/ducan-ne/supabase-clone/tree/main

Tiếp theo, nếu bạn muốn thêm 1 vài logic mới cho web, việc bạn cần làm là tạo 1 file `src/index.ts(js)` và thêm `<script type="module" src="src/index.ts"></script>` và boom, bạn có thể dev trực tiếp logic mới ngay trên giao diện website đang clone (cùng với hot reload, live reload), khi chạy `vite build` nó đồng thời cũng sẽ bundle ra 1 file asset js cho entry đấy. Nhưng lưu ý output sẽ là esm, nếu bạn cần support browser cũ thì hãy sử dụng plugin legacy của vite

Sau đó, tinh chỉnh lại project, Ctrl S sẽ ko clone đầy đủ website của bạn, ví dụ như thiếu 1 số file img trong css. Tất nhiên thì bạn có thể sử dụng các giải pháp clone website thay thế, thay vì Ctrl + S, nhưng ở ví dụ này chúng ta sẽ sử dụng Ctrl + S

Sau khi việc dev xong xuôi, việc bạn cần làm là deploy, ở ví dụ này mình sẽ sử dụng Vercel:

- Dùng lệnh vercel để init
- Vì vercel đã có pattern để biết project sử dụng vite, bạn chỉ việc nhấn enter từ đầu đến cuối.
- Chờ build & deploy lên vercel với url là project-cua-ban.vercel.app
- Sau khi deploy, nếu bạn cần custom domain, việc của bạn là vào vercel.com project setting => domains, add domain => check => new url ready.

Tips: sử dụng `vercel build --prod && vercel deploy --prebuilt --prod` để build locally và push built code lên vercel thay vì sử dụng CI của vercel, deploy sẽ nhanh hơn.

Vậy trong vòng 10 phút, bạn đã hoàn thành clone 1 website từ A-Z
Nếu thấy hữu ích với bạn hãy cho mình xin 1 like nhé.
