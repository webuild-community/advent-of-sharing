# Backup App's local storage for testing on Android

## Các rắc rối với local storage

1 app mobile thường có 2 thành phần lưu trữ chính, **_local storage_** và **_remote storage_** (server side). Local storage có thể chỉ là phần phản ánh dữ liệu ở remote, nhưng cũng có thể chứa các config chỉ dành cho client.

Ngoài ra, dữ liệu ở local còn có thể chứa session của user từ lúc đăng nhập. Khác với web app, người dùng ở client thường chỉ đăng nhập 1 lần và sử dụng. Quá trình đăng nhập với app cũng phiền phức hơn so với web app trên PC.

Bên cạnh đó, khi làm việc với database, developer thường gặp một điều phiền phức là version của database thay đổi, dẫn đến việc phải clean data trước rồi mới có thể cài đặt (rất hay gặp khi vừa phải fix bug trên `release` branch, vừa phải dev feature mới trên `main` branch).
Việc xoá dữ liệu local dẫn đến việc phải thực hiện lại hầu như từ đầu các thao tác để tái thiết lập dữ liệu, hoặc thậm chí dữ liệu test từ năm ngoái biến mất không thể khôi phục.

Test migration cũng là một việc phức tạp và cần phải lặp lại nhiều lần để có độ tự tin cao. Tuy nhiên, một khi đã migrate lên version mới, dữ liệu cũ không thể lấy lại được. Muốn có dữ liệu cũ phải cài lại app version cũ và thực hiện lại các bước để tái tạo dữ liệu.

Giải pháp cho những rắc rối trên chính là backup local storage để phục vụ cho việc test hoặc debug hoặc chỉ để tiết kiệm thời gian thiết lập môi trường dev cho tuỳ chỉnh khác, cũng giúp giảm chi phí cần phải có nhiều thiết bị để test cho nhiều môi trường khác nhau (ví dụ, 1 device để test cho account ở US, 1 device để test cho account ở VN, etc.).

## Vậy làm sao để backup local storage?

Với Android, Device Explorer trên Android Studio có thể giúp chúng ta làm việc này khá dễ dàng, nhưng lại khá mất thời gian.
Backup:
1. Tìm app
2. Tìm folder
3. Save
4. Chọn save location

Khi restore, chúng ta cần làm lại các bước tương tự.

Có cách nào để làm nhanh hơn không?
Thật may mắn, câu trả lời là có. `adb` kết hợp với `dd` là một công cụ mạnh mẽ để làm được việc này. Xem thêm chi tiết chém gió [ở đây](https://iamtuna.org/2023-10-08/use-adb-backup-restore-local-data-2).

### Ý tưởng cơ bản:
**Backup**
1. Nén các thư mục cần backup vào 1 file `.gz`
2. Copy file `.gz` vào `sdcard` (dù tên là sdcard nhưng Android sẽ trỏ tới external storage) bằng Unix pipe và `dd`
    ```bash
    adb shell "run-as ..." | adb shell "dd of=/sdcard/..."
    ```
**Restore**
1. Copy file từ sdcard vào app's dir bằng dd và Unix pipe
    ```bash
    adb shell "dd if=/sdcard/..." | adb shell "run-as ..."
    ```
2. Giải nén và thay các thư mục tương ứng của app bằng nội dung trong backup.

Xem script chi tiết ở repo này: https://github.com/tuanchauict/android-localstorage-backup/ (mirror from my scripts at work, not tested for general debuggable apps).

