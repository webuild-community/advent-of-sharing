---
title: Sử dụng native ESM browser để dựng 1 Micro FE App
author: ducan-ne
date: 2023-12-23
---

# Sử dụng native ESM browser để dựng 1 Micro FE App
## Mở đầu
Micro FE là 1 khái niệm không quá lạ lẫm với hầu hết các FE dev, có rất nhiều hướng tiếp cận để dựng 1 app với kiến trúc micro fe, có thể kể đến như
- Module federation
- Iframe
- Single SPA
- Vân vân

Mỗi hướng tiếp cận sẽ có pros và cons khác nhau, hôm nay mình mang tới 1 hướng tiếp cận mới, sử dụng esm có sẵn ở trình duyệt và esm.sh để shared libs. Mang tới một số benefits cũng như giảm sự phức tạp của implementation

## Motivation
Các hướng tiếp cận trước bắt buộc chúng ta phải implement những thứ như shared dependencies, shared state,  module system... Nhưng bằng cách sử dụng native esm ở browser và import lib từ esm.sh, hầu hết các vấn đề có thể gặp đã được bỏ qua với ESM.

## Demo
- https://microfe-esm-shell.pages.dev
- App 1: https://microfe-esm-app-1.pages.dev/entry.js
- App 2: https://microfe-esm-app-2.pages.dev/entry.js


## Đi vào thực hiện
Trong demo này, mình sử dụng Vite (/rollup) để thực hiện hướng tiếp cận này,
lí do bởi Vite sử dụng esm ở dev mode và ở build mode sử dụng rollup, 
giúp mình chỉ build entrypoint thay vì main.jsx, config đơn giản hơn, ngoài ra việc alias react tới https://esm.sh/react để typing tốt hơn, cũng dễ hơn rất nhiều (https://github.com/ducan-ne/microfe-esm-browser/blob/main/vite.config.js#L9C19-L9C19). (Có thể bạn ko biết: webpack import http url sẽ pull code về và build, ko giống như ở Vite)
Ở dev mode, từ shell mình import tới dev server App-1 http://localhost:3001/App.jsx và cảm ơn tới Vite vì việc này rất dễ dàng với Vite dev mode (bạn có thể tham khảo bài viết trước của mình về Vite dev mode), Vite tận dụng rất tốt esm browser ở dev mode, sau sử dụng annotation `/* @vite-ignore */` với dynamic import để Vite bỏ qua kiểm tra phần import này.

Khi build production code:
- Shell: replace http://localhost:3001/App.jsx qua production url https://microfe-esm-app-1.pages.dev/entry.js
- Sub app: chỉ build App.jsx => entry.js vì chúng ta chỉ cần code ở App.jsx, ngoài ra config thêm preserveEntrySignatures để giúp App component ko bị xoá khỏi bundle file, hiệu quả đạt được là Vite giúp chúng ta có dev server rất tốt, nhưng ở production code thì rollup giúp chúng ta như đang build 1 package js vậy

Note: thay vì esm.sh thì chúng ta có thể sử dụng importmap
Mọi người có thể test thêm ở trang Demo nhé :pray:

## So sánh
ESM browser mang tới một số lợi ích mới cũng như đơn giản hoá hơn so với các hướng tiếp cận cũ, có thể gọi là độ phức tạp bằng 0, một số lợi ích có thể kể đến như:
- Không còn dính issue 2 instance React cùng tồn tại
- Shared state đơn giản hơn bằng cách import trực tiếp tới module state, ví dụ `import { useStore } from 'https://thecompany.com/app-1/store.js'`
  - Hoặc 1 shared app chứa các shared modules được import bởi nhiều app? `import { useStore } from 'https://thecompany.com/shared/store.js'` , deploy independent
- Mental model đơn giản hơn vì đây là cách browser esm work, không còn các concept của federation, remote module, container,....
- Load app nhanh hơn vì esm.sh sử dụng http3 và nếu app của bạn cũng đang host trên http3 (ví dụ cloudflare)
- Dev server dễ dàng (solution khác khá khó) và code nhất quán với production (:hand: nếu bạn đã từng gặp issue tương tự với module federation)
- Dev server import tới staging urls (app-1 import app-2) => Hoàn toàn có thể
- Không cần build tool nếu bạn thích bundleless như DHH hoặc có thể chỉ sử dụng esbuild để transpile file ts => js
- Native hơn: ko còn rely vào 1 implementation bên ngoài như module federation, code của người khác mình sẽ ko tin bằng code của mình
- Scalable: mở ra nhiều hướng custom dễ hơn cho phù hợp với yêu cầu của app
- Astro: nếu bạn đang sử dụng nhiều framework, Astro có lẽ là 1 lựa chọn tốt và cũng đỡ implement nhiều thứ hơn.
- Ko nhất thiết chỉ sử dụng cho use case micro fe.

Về chung, đây là 1 giải pháp khá hứa hẹn, tuy nhiên, hãy xem xét kĩ khi cân nhắc sử dụng hướng tiếp cận này vì nó cũng có 1 số điểm cần lưu ý
- Browser support:
  - https://caniuse.com/?search=ESM
  - https://caniuse.com/?search=importmap
- Cache: bởi vì khi mình bundle ra 1 file js nên hãy cân nhắc để code mình sau 1 con gateway support purge cache theo deployment (hoặc quản lý thêm version ?version=), ví dụ như Vercel, Cloudflare, Netlify, Kong (self host). Hoặc tắt cache-control: no cache cũng là 1 lựa chọn khá tốt. Hoặc implement thêm FE discovery (https://github.com/awslabs/frontend-discovery) và sử dụng hash (Vite hỗ trợ export manifest.json)....
....


## Kết luận
Mặc dù trong hầu hết các app hiện đại đều ko cần tới micro fe và cảm giác như khá vô ích ở thực tế khi học những kiến thức về micro-fe, nhưng không có nghĩa kiến thức đó là useless, có thể là để hiểu thêm về cách mà browser hoạt động thay vì các "metaframework" che dấu nó đi và bạn chỉ cần viết code, hay là giúp bạn có sự cân nhắc tốt hơn trước khi ra quyết định cho use case của bạn (Fact: framer.com là 1 website có kiến trúc universal modules sử dụng ESM, nghe qua rất là micro fe nhưng thực tế ko phải micro fe).

Bài viết khá ngắn vì viết ở Slack khá giới hạn nội dung (đã thử Canvas nhưng ko share nội dung vào đây dc), nên mình chỉ share sơ lược về implementation mà ko đi vào details, nên hãy AMA về phần code hoặc hơn nhé :pray:

Mình đã implement đầy đủ ví dụ kể trên, tham khảo code kĩ hơn tại https://github.com/ducan-ne/microfe-esm-browser