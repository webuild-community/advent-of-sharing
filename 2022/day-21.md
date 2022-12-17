---
title: Debugging vấn đề về performance với PerformanceObserver
author: Bần làm vườn
date: 12-14-2022
---

(Disclaimer: Đây không phải là post khuyên bạn từ bỏ console.log đi và xài những công cụ debug advance hơn. In fact, với API này, console.log càng trở nên lợi hại)

**Context**: `PerformanceObserver` là một API trong nhóm gọi chung là Performance API. Nếu mọi người cảm thấy nó hơi xa lạ thì here's a tidbit: hàm `performance.now()` mà mọi người thường dùng để measure task run time là được gọi trên một cái global `Performance` object. Nếu chưa có dịp xem qua thì highly recommend mọi người dành một buổi để nghía qua hết [docs](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API) của tụi này.
Back to the topic. Các API khác trong nhóm này đóng vài trò record, measure và tạo ra performance events/metrics. Với `PerformanceObserver`, chúng ta sẽ được notify khi các performance events đó xảy ra. Ngoài việc có thể gửi các metrics đó đến logging service thì chúng ta còn có thể dùng nó để test rằng một interaction nào đó có đang trigger quá nhiều performance event (ví dụ như layout shift) ngoài mong muốn hay không.

**Mechanism:**

```ts
// Create new observer instance
const observer = new PerformanceObserver(callback)
// observe the type of performance metrics we care about
observer.observe(performanceEntryTypes)
```

Mình sẽ không đi sâu về cách sử dụng chi tiết. Các bạn có thể xem qua [docs](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API) của `PerformanceObserver` để biết thêm.

**Here's a use case example:**

Test for layout shift

```ts
let shiftScore = 0;

new PerformanceObserver((entryList) => {
  // Loop through all the layout shift event
  for (const entry of entryList.getEntries()) {
    // Only count if the layout shift occur without immediate user action, the bad kind
    if (!entry.hadRecentInput) {
      // value for the layout entry is sort of like a point of how severe the shift is
      shiftScore += entry.value;
      // shiftScore will increase overtime
      console.log('Current shiftScore:', shiftScore, entry);
    }
 }}).observe({type: 'layout-shift', buffered: true});
```

Xong thì mọi người đi interact với chỗ mà mọi người đang nghi là tạo ra unnecessary layout shift and watch your console.

**Extra note**: Có một event type khá juicy là `longtask`, mình chưa có dịp xài, [but you might find a need for it](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongTaskTiming)

**Extra note 2**: For a quick list of supported event (entry) type, run `PerformanceObserver.supportedEntryTypes`

**Extra note 3**: Có trong docs nhưng mà mình lôi luôn ra đây. This API can run in Web Worker. If you're gathering real time metrics, consider putting it in here.
