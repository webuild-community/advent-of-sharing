---
title: Vì sao bạn nên ngưng sử dụng unload event và event listener?
author: Bần làm vườn 
date: 12-17-2022
---
**Context:** Nhảy thẳng vào kết bài: do Backward/forward cache (viết tắt là bfcache).

**Quick introduction** :point_right: bfcache tồn tại độc lập và không liên quan với HTTP Cache. HTTP Cache chỉ có thể cache http response và về bản chất nó là một disk cache. bfcache trong khi đó là một in-memory cache, đóng vai trò lưu trữ lại toàn bộ snapshot của một page, bao gồm cả javascript heap. Việc này giúp cho các page có utilize bfcache có thể phục hồi một cách tức thời khi user click back button. **Đây là một feature của browser, mọi người không cần phải opt-in để sử dụng nó, nhưng có nhiều caveats khiến feature không được apply.**

Một pattern phổ biến trong số đó là việc sử dụng `unload` event. Khi có tồn tại listener của event này, browser sẽ opt-out hoàn toàn và không đưa page đó và bfcache. Bạn sẽ tự hỏi tại sao nó lại có cái behaviour như vậy. Câu trả lời cũng là câu trả lời chung cho rất nhiều sự tồn tại mang tính chất đau não ở frontend: legacy.

Ở thời xa xưa, một khi page đã unload thì nó sẽ không còn tồn tại nữa, và vì vậy rất nhiều legacy site cũng sẽ hoạt động với một expectation như vậy. Trong khi đó, với bfcache, page thực tế vẫn còn nằm trong memory và chỉ được "hide" đi. Để tránh break các site này, browser vendor bắt buộc phải đưa ra compromise: page nào còn sử dụng unload sẽ tự động opt-out khỏi bfcache.

**Solution:**

Thay cho `unload`, chúng ta sẽ dùng Page Lifecycle event `pagehide`. Và sibling event với nó là `pageshow`

```javascript
window.addEventListener("pagehide", (event) => {
  if (event.persisted) {
    /* the page isn't being discarded, so it can be reused later */
  }
}, false);
```

Ở ví dụ trên, các bạn sẽ thấy `pagehide` event có một thuộc tính là `persisted`. Thuộc tính này sẽ cho ta biết là browser có đang định cache page hiện tại hay không (không guaranteed vì việc cache đó vẫn chưa diễn ra). `pageshow` cũng sẽ có thuộc tính tương tự. Bằng cách check thuộc tính này ở pageshow, ta có thể biết được lần load này có phải từ bfcache hay không và có phương án xử lí tương ứng (ví dụ: update stale data, clearing sensitive fields, force reload when signing out).

**Kết:** Nếu các bạn dùng Lighthouse và thấy warning này và chưa biết tại sao thì this is why. Title post này chỉ là một title giật tít trá hình để nói thêm về bfcache. Caveats thì hằng hà sa số nên muốn biết thêm chi tiết, các bạn đọc thêm ở đây (Warning: it's freaking long). bfcache is "free" performance, but it's not free without some work :pepesad:

_**Extra note:**_ `beforeunload` mình không nhắc đến ở trên vì behaviour của nó với bfcache không đồng nhất giữa các browser. Xem phần caveats trong link trên.

_**Extra note 2:**_ Phần Application > Cache trong Chrome Dev Tools có chỗ để mọi người test bfcache có được apply ở current page hay không.
