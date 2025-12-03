**Author:** Thanh

---

# Một Vài Lưu Ý Khi Lập Lịch Hàng Tháng với EventBridge

TL;DR1
------

Lên lịch hàng tháng vào 0h ngày x mỗi tháng:

```cron
cron(0 0 x * ? *)
```

...và cái kết.

TL;DR2
------

- AWS EventBridge Scheduler giúp tự động hóa các tác vụ định kỳ.
- 3 loại schedule type: Rate-based, Cron-based, và One-time Schedule.
- Thách thức chính là xử lý các tháng có số ngày khác nhau.
- Rate-based kích hoạt theo khoảng thời gian cố định, nhưng có thể lệch với độ dài tháng.
- Cron-based phù hợp nhưng gặp khó khăn với các tháng thiếu ngày cụ thể (ví dụ: ngày 31).
- One-time schedule có thể được dùng cho lịch trình chính xác và linh hoạt.

Đọc chi tiết bài viết [tại đây](https://beautyoncode.com/aws/even-bridge-scheduler/).

![cover](https://beautyoncode.com/assets/images/2024/12/2024-12-10-even-bridge-scheduler-cover.jpg)

---

Trong bài viết hôm nay, mình sẽ chia sẻ về một chủ đề đơn giản nhưng thú vị: **lập lịch công việc định kỳ (scheduling)**. Hãy tưởng tượng bạn cần tự động tặng thêm **$10 hàng tháng** cho khách hàng đã đăng ký gói Premium của dịch vụ bạn cung cấp.

Với AWS, có hai giải pháp phổ biến để giải quyết bài toán này: **SQS Queue** và **EventBridge**. Trong bài viết này, mình sẽ tập trung vào giải pháp sử dụng **Amazon EventBridge Scheduler**.

Để bạn dễ hình dung bài toán, dưới đây là **[Subscription-Based Payment Flow with AWS EventBridge and Stripe](https://drive.google.com/file/d/1zsuc1sh_sLDmxJxjsJRunB42yioDdwdm/view?usp=drive_link)**:

![flow](https://beautyoncode.com/assets/images/2024/12/2024-12-10-even-bridge-scheduler-flow-1.png)

**Mô tả flow:**

1. Người dùng muốn mua gói **subscription Premium** theo hình thức thanh toán **hàng năm**.
2. Ứng dụng Frontend gọi đến Backend để lấy thông tin phiên checkout của Stripe.
3. Ứng dụng FE nhận được URL phiên checkout và chuyển hướng người dùng đến giao diện Stripe để bắt đầu thanh toán.
4. Người dùng điền thông tin và thanh toán thành công.
5. Stripe kích hoạt một Webhook event thông báo thanh toán thành công, một gói đăng ký mới được kích hoạt.
6. Ứng dụng BE lắng nghe sự kiện và tạo một **EventBridge Scheduler** sẽ chạy **hàng tháng**, bắt đầu từ thời gian hiện tại.
7. EventBridge Scheduler kích hoạt sự kiện theo lịch (hàng tháng), hàm **Lambda** được cài đặt làm mục tiêu sẽ chạy.
8. Lambda xử lý logic công việc và cập nhật cơ sở dữ liệu (nếu có).

Bài viết này sẽ tập trung phân tích cách lập lịch hàng tháng với EventBridge Scheduler trong **flow 6 và 7: Schedule Monthly Task with EventBridge**.

---

## Amazon EventBridge là gì?

[Amazon EventBridge](https://aws.amazon.com/eventbridge/) là một dịch vụ serverless của AWS, giúp bạn dễ dàng kết nối các ứng dụng thông qua các sự kiện (event-driven). EventBridge hỗ trợ lập lịch công việc và chuyển tiếp sự kiện từ nhiều nguồn khác nhau (AWS services, SaaS apps, custom applications) đến các mục tiêu như Lambda, SQS, Step Functions…

Bạn có thể xem thêm video giới thiệu ngắn gọn này trên trang chính thức của AWS.

---

## Amazon EventBridge Scheduler là gì?

**Amazon EventBridge Scheduler** là một tính năng được tích hợp sẵn trong EventBridge, cho phép lập lịch và thực thi các công việc định kỳ hoặc vào các thời điểm cụ thể trong tương lai. Nó giúp tự động hóa các tác vụ như:

- Gửi thông báo
- Kích hoạt Lambda function
- Chạy Step Functions
- Gửi sự kiện đến các dịch vụ khác trong AWS theo lịch trình được xác định trước

**Các tính năng chính:**

- Lập lịch công việc định kỳ.
- Hỗ trợ các loại khác nhau: **Rate-based**, **Cron-based**, **One-time**.
- Tích hợp với các dịch vụ AWS như Lambda, Step Functions, SQS, …
- Quản lý lịch trình dễ dàng qua **AWS Management Console, SDK, CLI, API**.

---

## Phân Loại Lịch trong EventBridge Scheduler

EventBridge Scheduler hỗ trợ **3 loại schedule**:

### 1. Rate-based schedule (Định kỳ theo chu kỳ)

Dùng để trigger event theo khoảng thời gian cố định.

**Cú pháp:**

```text
rate(value unit)
```

Với:

- `value` là số dương
- `unit` là `minutes`, `hours`, hoặc `days`

**Ví dụ:**

```text
rate(5 minutes)
```

Sẽ trigger event mỗi 5 phút.

![rate](https://beautyoncode.com/assets/images/2024/12/2024-12-10-even-bridge-scheduler-rate.png)

### 2. Cron-based schedule (Định kỳ theo lịch cụ thể)

Dùng để trigger event vào thời gian cụ thể trong ngày, tuần, tháng hoặc năm.

**Cú pháp:**

```text
cron(minutes hours day-of-month month day-of-week year)
```

**Ví dụ:**

```text
cron(0 0 1 * ? *)
```

Sẽ trigger vào **0h ngày 1 mỗi tháng**.

![cron](https://beautyoncode.com/assets/images/2024/12/2024-12-10-even-bridge-scheduler-cron.png)

### 3. One-time schedule (chỉ định một lần)

Trigger sự kiện duy nhất vào một thời điểm cụ thể.

![one-time](https://beautyoncode.com/assets/images/2024/12/2024-12-10-even-bridge-scheduler-one-time-schedule.png)

---

## Phân Tích Bài Toán Lập Lịch Hàng Tháng

Giả sử bạn cần tự động thực hiện tác vụ trên (tặng $10 vào tài khoản) **hàng tháng** vào đúng ngày mua **“Premium subscription”** của khách hàng, dưới đây là phân tích về hai phương pháp phổ biến:

### Rate-based Schedule

**Rate expression:**

```text
rate(30 days)
```

**Hạn chế:**

- Tháng có thể dài hơn hoặc ngắn hơn 30 ngày (ví dụ tháng 2 có 28 ngày).
- Việc sử dụng `rate(30 days)` sẽ gây ra sự chênh lệch, khiến người dùng nhận thông báo hoặc hành động **không đúng ngày mong muốn**.

### Cron-based Schedule

**Cron expression:**

```text
cron(0 0 x * ? *)
```

Trigger event mỗi tháng vào ngày `x` (ngày mua Premium subscription).

**Hạn chế:**

- Nếu người dùng mua gói vào **ngày 31**, cron sẽ **không thể trigger** trong các tháng không có ngày 31 (tháng 2, 4, 6, 9, 11).
- Tương tự cho các ngày **29, 30** thì tháng 2 sẽ bị thiếu events.

![cron-missing](https://beautyoncode.com/assets/images/2024/12/2024-12-10-even-bridge-scheduler-cron-missing-event.png)

---

## Giải Pháp Xử Lý Cron-based Schedule

### 1. Sử Dụng Ngày Đầu Tháng Tiếp Theo

Đặt cron chuyển các ngày lớn hơn 28 thành **ngày 1 của tháng tiếp theo**.

Cách này đơn giản, đảm bảo **mỗi tháng đều có event trigger**, nhưng có thể gây trễ 1–2 ngày với người dùng.

**Ví dụ:**

- Ngày `≤ 28`:

  ```cron
  cron(0 0 x * ? *)
  ```

- Ngày `> 28`:

  ```cron
  cron(0 0 1 * ? *)
  ```

### 2. Sử Dụng One-time Schedule

Với mỗi lần trigger, Lambda function sẽ **tạo ra lịch trigger tiếp theo** dựa trên logic tính toán ngày cuối cùng của tháng.

**Ví dụ:**

- Nếu user mua gói Premium vào ngày `31/12/2024`, các lần trigger tiếp theo sẽ là:
  - `31/01/2025`
  - `28/02/2025`
  - `31/03/2025`

Viết logic code để chọn đúng ngày tiếp theo và tạo **one-time schedule** tương ứng.

---

## Giải Pháp Xử Lý Online Payment Tương Ứng với Scheduler

Sau khi chọn được loại schedule phù hợp với mình là **“Sử Dụng Ngày Đầu Tháng Tiếp Theo”**, cần xử lý logic tương ứng với sự kiện **Stripe Payment**.

**Ví dụ:**

- Nếu người dùng thanh toán vào ngày **28/02/2024** nhưng bạn chọn:

  ```cron
  cron(0 0 1 * ? *)
  ```

  Sự kiện sẽ trigger vào ngày **01/03/2024** (trễ **1 ngày**).

- Nếu thanh toán vào ngày **30/10/2024** nhưng bạn chọn:

  ```cron
  cron(0 0 1 * ? *)
  ```

  Sự kiện sẽ trigger vào ngày **01/11/2024** (trễ **2 ngày**).

Nếu đây là chương trình khuyến mãi, việc trễ 1–2 ngày có thể chấp nhận được. Tuy nhiên, nếu tính chất công việc quan trọng yêu cầu **chính xác cao**, thì hướng tiếp cận này chưa tối ưu.

Với **Stripe**, có một tính năng là **Prorations** – cài đặt `"proration_behavior": "none"` khi tạo checkout session, người dùng sẽ **không trả chi phí cho 1–2 ngày gap** giữa ngày mua subscription và ngày trigger event quan trọng, vì thanh toán thực tế sẽ diễn ra vào **ngày 1 của tháng tiếp theo**.

Điều này giúp:

- Giữ **chính xác về ngày trigger event**
- Đảm bảo **quyền lợi cho người dùng**

---

## Kết Luận

Khi lập lịch công việc định kỳ với AWS EventBridge, bạn cần cân nhắc kỹ các trường hợp đặc biệt như:

- Các tháng **không có đủ số ngày** trong cron expression.
- Sự **khác biệt về số ngày giữa các tháng**.

Hy vọng nội dung này giúp bạn hiểu rõ hơn cách dùng **EventBridge Scheduler** và cách chọn **schedule type** phù hợp với bài toán của mình.

**Đọc thêm:**

- [Amazon Event Bridge – Schedule types in EventBridge Scheduler](https://docs.aws.amazon.com/)
- [Stripe – Prorations](https://stripe.com/docs/billing/subscriptions/prorations)
- [Stripe – Checkout Session](https://stripe.com/docs/payments/checkout)