---
title: Javascript obfuscator and how it actually works
author: ducan-ne
date: 12-19-2022
---
Là một web developer, hẳn bạn đã thấy qua môt số mã nguồn bị mã hóa như dưới đây, có bao giờ bạn tò mò về cách thực sự mà nó hoạt động :thinkhard:

Nhìn có vẻ rối răm, nhưng thực ra obfuscator lại hoạt động 1 cách rất có logic, cụ thể bao gồm:

- Phần core để parse đoạn code javascript thành ast để thuận tiện cho việc transform, thường là sử dụng lib nào đó như acorn
- Sau đó, sẽ có các bộ transformer sẽ được chạy trên ast tokens:
  - Identifiers transformer: đi qua các variable của bạn và rename nó thành _0x+hex(randomNumber()), mục đích để _0x ban đầu đơn giản chỉ là muốn dùng 1 đoạn số random, nhưng javascript lại ko support variable bằng số, chúng ta có 2 loại identifies transformers: hex, mangled (a, b, c)
  - String array transformer: chuyển các đoạn string của bạn vào một array, encode nó qua 1 dạng khác, và chuyển đoạn code bạn qua sử dụng array đó, ví dụ: `console.log("hello", "world") => var fullStrings = ["hello", "world"] console.log(fullStrings[0], fullStrings[1])`
  - Compact transformer: cách hoạt động giống như UglifyJS, đơn giản là minify code
  - Dead code injector: thêm các đoạn code thừa thãi vào code bạn để nó khó đọc hơn
  - Disable console output injector: override console.log, console.error thành function rỗng để chặn ko log ra devtool
  - Domain lock injector: thêm 1 đoạn check location.hostname và obfuscate nó
  - Debug protection injector: đơn giản là 1 đoạn code `setInterval(() => { debugger }, 1e3)`, nó sẽ chạy debugger liên tục ngay cả sau khi bạn nhấn continue, giải pháp là tắt ở devtool => source
  - Self defending injector: có bao giờ bạn tự hỏi sao mình pretty code đã obfuscate thì nó sẽ chạy quài ko hết thậm chí đơ cả trình duyệt, thực ra có 1 đoạn code bên dưới lưu function().toString().length ban đầu, ví dụ ban đầu là 50, sau khi pretty nó sẽ là 100 (thêm dấu cách, xuống dòng, ...), 1 dòng if và bạn có thể làm bất cứ điều gì sau đấy.
  - Anti devtool injector: 1 vài dòng code có thể detect nếu devtool đang được bật, sẽ crash tab

Sau đấy gửi output ra cho bạn, nghe có vẻ đơn giản nhưng việc implement cũng ko hề đơn giản, bạn có thể đọc qua source code của obfuscator.io

Tham khảo thêm các loại transformer khác tại obfuscator.io, hoặc test ngay với code project của bạn :pepewink:

Vậy trong thực tế chúng ta có nên obfuscate code không:

- Câu trả lời cho hầu hết trường hợp là không, obfuscate ko đảm bảo **100%** và chỉ chặn được các script kiddies, ko cái gì là hoàn hảo và chỉ cần hiểu nguyên lý của nó là có thể dịch ngược lại, obfuscator.io mình có thể deobfuscate ra gần như đầy đủ đoạn code ban đầu.
- Tiếp tục không, vì obfuscate code sẽ làm chậm code đáng kể
- Nhưng sử dụng trong trường hợp mình cần gửi code cho khách nhưng chưa thanh toán, và mình cần gắn 1 feature flag vào để chống bùng :trollface:
- Hay để bảo vệ thông tin nhạy cảm như **publickey** lưu dưới client
