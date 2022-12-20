---
title: Tips to improve canvas performance
author: ZeroX-DG
date: 12-17-2022
---

Cải thiện performance cho canvas rendering là một công chuyện vô cùng thú vị của một front-end developer, vì nó bắt bạn ngụp lội đọc spec, học cách browser hoạt động và phát hiện ra nhiều tool cộ xịn bị giấu trong web developer tools.

Data & visual càng phức tạp, thì công việc optimise performance càng phải thực hiện chuyên sâu, nhưng bài này sẽ chỉ đi vào một vài cách cơ bản và “mì ăn liền”.

## Tip 1: Đừng xài canvas

Browser là phần mềm đã tồn tại từ xa xưa, trải qua nhiều phong ba bão táp và được nhiều lãnh tụ tiên phong trong ngành dành nhiều thời gian công sức để optimise performance. Khi chọn dùng canvas, bạn chọn hi sinh những optimisations của những bậc đi trước để có thể tuỳ ý customise app của mình.

Thế nên hãy suy nghĩ kĩ và chỉ dùng canvas khi DOM hay SVG trở nên quá chậm chạp (có thể do số lượng data lớn và visual phức tạp).


## Tip 2: Hãy dùng requestAnimationFrame đúng cách

Khi làm animation với canvas, bạn sẽ phải redraw mỗi frame khi có sự thay đổi. Điều này từng được thực hiện với `setInterval` hay `setTimeout`:

```
setInterval(animationFunction, 1000 / 60);
// or
const doAnimation = () => {
  animationFunction();
  setTimeout(doAnimation, 1000 / 60);
};
doAnimation();
```

Nhưng kết quả khi sử dụng 2 hàm đó là animation rất hay bị giật/lag. Thế nên best practice hiện tại mà mọi người sử dụng là `requestAnimationFrame`:

```
const doAnimation = () => {
  animationFunction();
  requestAnimationFrame(doAnimation);
};
doAnimation();
```

Tuy vậy, mọi người thường cho rằng `requestAnimationFrame` sẽ chạy 60 lần mỗi giây (60 FPS), nhưng thực chất `requestAnimationFrame` [sẽ match với refresh rate của monitor](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame). Nên refresh rate càng cao, số lượng frame redraw càng nhiều:

> You should call this method whenever you’re ready to update your animation onscreen. This will request that your animation function be called before the browser performs the next repaint. The number of callbacks is usually 60 times per second, but will generally match the display refresh rate in most web browsers as per W3C recommendation.

Thế nên người dùng phổ thông với 60hz monitor có thể sẽ thấy animation của app khá mượt, nhưng khi upgrade lên 120hz monitor, CPU của họ có thể sẽ bắt đầu kêu gào và tới 240hz thì hỡi ôi mùa đông này họ sẽ không còn phải chịu lạnh nữa.

Cách xử lý cho tình huống này là:
- Giảm thiểu số lần redraw không cần thiết. (Không có gì đổi thì thôi đừng redraw).
- Chỉ redraw với lần requestAnimationFrame thứ N để phân tán số lượng redraw:

```
const doAnimation = () => {
  if (timePassedMs > 1000 / 60) { // 60 FPS
    animationFunction();
  }
  requestAnimationFrame(doAnimation);
};
doAnimation();
```

## Tip 3: Reuse text rendering càng nhiều càng tốt

Text rendering là một [công chiện cực kì tốn kém](https://faultlore.com/blah/text-hates-you/). Thế nên bạn nên hạn chế gọi `context.fillText()` càng nhiều càng tốt.
Giả dụ bạn đang làm game sudoku bằng canvas và cần render số từ 1 tới 9 cho tất cả 9x9 = 81 ô. Thì thay vì gọi `fillText()` hết 81 lần, bạn có thể gọi `fillText()` 9 lần để render các số từ 1 tới 9 và cache vào một canvas riêng, sau đó sử dụng `drawImage()` để render số vào ô thay cho `fillText()` vì `drawImage` chỉ đơn giản là copy pixel từ canvas này đè lên canvas kia nên nhanh hơn gọi `fillText()`:

```
const numbersCache = new Map();
// draw & cache 1 - 9
for (const i = 1; i <= 9; i++) {
  const canvas = document.createElement('canvas');
  canvas.getContext('2d').fillText(i.toString());
  numbersCache.set(i, canvas);
}
// draw the number from cache instead of using fillText
mainCanvasContext.drawImage(numbersCache.get(1), 0, 0);
```

## Tip 4: Hạn chế thay đổi context càng nhiều càng tốt

Mình cũng không rõ tại sao nhưng nếu có càng nhiều sự thay đổi vào state của canvas context thì rendering sẽ càng chậm. Ví dụ đổi font càng nhiều, render sẽ càng chậm (`context.font = '…'`). Thế nên mình khuyến khích render theo batch càng nhiều các tốt. Thay vì đổi font liên tục để render text, thì bạn có thể batch các text có cùng font lại với nhau để render cùng một lúc.

## Tip 5: Đừng render mấy thứ không cần thiết

Trước khi render web page, browser thường lọc ra những element nào [below the fold](https://www.optimizely.com/optimization-glossary/below-the-fold/) hay `display: none` để giảm bớt số lượng render calls. Bạn cũng có thể chiêu thức này vào canvas của bạn và giảm số lượng render calls, hạn chế thay đổi context, redraw, etc. Chỉ cần trước khi render, filter ra những thứ gì nằm ngoài canvas hay nằm dưới cùng và bị nhiều thức khác che lấp.

## Tip 6: Chồng chất nhiều canvas lên nhau khi cần thiết

Tưởng tượng bạn làm ra một ứng dụng vẽ bằng canvas. Bạn gọi nó là Excalidraw. Bạn gặp vấn đề là bạn muốn di chuyển được những thứ bạn vừa vẽ. Implement thì không khó, nhưng performance của app giảm đáng sợ khi bạn có nhiều element trên canvas. Lý do là mỗi lần di chuyển element, canvas của bạn cần:

- Clear frame
- Loop qua mỗi element và vẽ lại dù element đó có thay đổi hay không.
- Canvas đang có 1000 elements

Để khắc phục vấn đề này, bạn có thể tạo ra 3 canvas cùng kích thước và nằm chồng lên nhau.

- Canvas đáy: Chứa những element đang không di chuyển và nằm dưới element đang di chuyển. Canvas này chỉ render một lần khi có một element bắt đầu di chuyển.
- Canvas giữa: Liên tục redraw và chỉ vẽ element đang bị di chuyển.
- Canvas trên cùng: Y chang canvas đáy, khác mỗi việc nó chứa những element không di chuyển và nằm trên element đang di chuyển.

Với cấu trúc này, khi drag và di chuyển một element, thì tất cả các element khác không bị thay đổi sẽ nằm ở canvas đáy hoặc trên cùng và chỉ render một lần, còn element đang bị di chuyển sẽ nằm ở canvas giữa và liên tục được redraw. Tức là 999 elements không di chuyển sẽ chỉ render một lần, và 1 element bị di chuyển sẽ render liên tục nên ít tốn kém.
Sử dụng một canvas, độ phức tạp để render cho mỗi frame là `O(n)` , còn dùng 3 canvas độ phức tạp sẽ là `O(1)`!

![image](https://user-images.githubusercontent.com/12984316/208667323-b0f5afde-1a00-4c60-bbcc-e34d9e373339.png)


hết gòi!

