**Author:** An

---

# next/font is more complicated than you think

An see u

![](https://an.cyou/favicon.png)

https://an.cyou/notes/next-font-is-more-complicated-than-you-think

![](https://ogcool.vercel.app/templates/v1/tpl_TnllPg0CkP?d=eyJtb2RpZmljYXRpb25zIjpbeyJuYW1lIjoiVGV4dCIsInRleHQiOiJuZXh0L2ZvbnQgaXMgbW9yZSBjb21wbGljYXRlZCB0aGFuIHlvdSB0aGluayJ9XX0%3D&sdk=ogcool%400.1.10)

---

Thời gian vừa rồi mình làm việc khá nhiều với **Astro**, và việc Astro không có 1 cái tương tự `next/font` khiến mình khá là đau đầu, website mình vẫn dính CLS khi load trang lần đầu khi font chưa được cache.

Mình đã nghĩ là oh, có thể `next/font` sử dụng 1 API browser hoặc meta tag nào đấy giúp giải quyết vấn đề này, rõ ràng mình có thể làm lại logic tương tự ở Astro.

Nhưng sau khi thử qua các cách `font-display`: `swap`, `fallback`, `optional`, preload font, serve font từ local nhưng vẫn không thể có được kết quả mong muốn.

Cho tới khi mình tìm được **astro-font** thì vấn đề được giải quyết, có vẻ như `astro-font` và `next/font` có 1 cơ chế giống nhau, tò mò thôi thúc mình đọc code để tìm hiểu kĩ hơn, dưới đây là cách nó thực sự hoạt động:

1. **Download font** từ Google (hoặc sử dụng local), lưu vào cache.  
2. **Thêm preload** vào header.  
3. **Sử dụng các font metrics** (`size-adjust`, `ascent-override`, `descent-override`, `line-gap-override`, …), thư viện **fontkit** để:
   - Tạo ra 1 fallback font từ font gốc.  
   - Tạo 1 fallback font từ local font gần giống nhất với dimensions của font gốc.  
   - Mặc định sẽ sử dụng **Arial** cho sans và **Times New Roman** cho serif.

Trình duyệt sẽ sử dụng fallback font để render text, và khi font gốc được load xong, trình duyệt sẽ thay thế font fallback bằng font gốc, và bởi vì font sử dụng và font fallback có dimensions gần giống nhau, nên giảm thiểu tối đa CLS.

Đọc thêm về font metrics và background của font fallback tại đây:  
https://developer.chrome.com/blog/font-fallbacks/#background

## Tham khảo thêm

- https://github.com/rishi-raj-jain/astro-font/blob/master/packages/astro-font/utils.ts#L363  
- https://github.com/vercel/next.js/blob/canary/packages/font/src/local/get-fallback-metrics-from-font-file.ts#L73  
- https://docs.google.com/document/d/e/2PACX-1vRsazeNirATC7lIj2aErSHpK26hZ6dA9GsQ069GEbq5fyzXEhXbvByoftSfhG82aJXmrQ_sJCPBqcx_/pub  
- https://developer.chrome.com/blog/font-fallbacks/  
- https://github.com/nuxt-modules/fontaine  
- https://github.com/unjs/fontaine