**Author:** An

---

# Smart Skeleton

[An see u](https://an.cyou/notes/smart-skeleton)

![Favicon](https://an.cyou/favicon.png)

![Smart Skeleton OG](https://ogcool.vercel.app/templates/v1/tpl_TnllPg0CkP?d=eyJtb2RpZmljYXRpb25zIjpbeyJuYW1lIjoiVGV4dCIsInRleHQiOiJTbWFydCBTa2VsZXRvbiJ9XX0%3D&sdk=ogcool%400.1.10)

---

Giải pháp mình tìm được gần đây là sử dụng `-webkit-box-decoration-break: clone`, một CSS property mới được thêm vào từ Chrome 130, giúp tuỳ chỉnh cách hiển thị khi element xuống dòng (đọc thêm [tại đây](https://developer.mozilla.org/en-US/docs/Web/CSS/box-decoration-break)), bạn có thể với button phía trên để thấy sự khác biệt.

## Một số cải tiến có thể thực hiện thêm trong thực tế

- Dùng `TextContext` của `react-aria-components` để đẩy trạng thái loading xuống các Text, bỏ qua việc if else trên từng element.  
- Tương tự với các component khác như Image, Tabs,…  
- Tạo 1 component Skeleton wrap lại toàn bộ content thay vì if else trên từng element, có thể cân nhắn làm thêm `SkeletonContext`.  
- Smart Skeleton: sau khi nhận được response từ API, lưu `totalPosts` vào localStorage và sử dụng `totalPosts` cho render Skeleton trong lần kế tiếp load lại trang. Điều này giúp tăng trải nghiệm người dùng khi dùng số lượng Skeleton giống với thực tế (ví dụ 3) thay vì cố định 1 số như 10/20.

---

Trong hành trình tìm kiếm xây dựng code base tinh gọn nhưng hiệu quả…, mình đã đi qua rất nhiều cách nhưng một trong những thứ khiến mình thích thú là việc tái sử dụng layout từ component gốc thay vì tạo Skeleton riêng biệt.

Mình rất thích việc thêm Skeleton vào app, nhưng việc build 1 component Skeleton chung layout cho mỗi component rất mất thời gian, cũng như công sức để update Skeleton khi có thay đổi từ component gốc.

Qua 1 thời gian, dưới đây là hướng tiếp cận được mình sử dụng nhiều khi cần tới Skeleton, không cần code từng component Skeleton mà thay vào đó tái sử dụng Layout từ component gốc:

## 1. Sử dụng `fallbackData` / `initialData`

Sử dụng `fallbackData` / `initialData` của React Query hoặc SWR tạo fake data cho Skeleton trong lúc chờ dữ liệu thực sự được load.  
Trong trường hợp này mình sử dụng 10 items.

## 2. Thêm các class để chuyển đổi thành Skeleton khi loading

Khi trạng thái `isLoading` là `true`, apply các class sau lên các element hoặc interactive elements:

- `text-transparent`: Ẩn nội dung text.  
- `bg-gray-400` + `animate-pulse`: Thêm background xám để tạo hiệu ứng Skeleton.  
- `opacity-40`: Làm mờ element.  
- `[&>*]:invisible`: Ẩn các children của element.  
- `SkeletonWrapper`: Một wrapper để ẩn các element khi loading, đi cùng [`inert`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inert) attribute để ngăn người dùng tương tác với các element khi loading.

### App.js

![App Skeleton Example](https://prod-files-secure.s3.us-west-2.amazonaws.com/c15e2d47-e034-46ca-8ca6-8717fa9b2844/6b26bdf8-39db-4d21-b094-ad09146da6b2/image.png)

Nhưng cách này không thực sự hoàn hảo, trong một số trường hợp ví dụ như dưới đây, khi đoạn text xuống dòng, background của nó sẽ bị hỏng.

### App.js (Text xuống dòng)

![Broken Background Example](https://prod-files-secure.s3.us-west-2.amazonaws.com/c15e2d47-e034-46ca-8ca6-8717fa9b2844/1dedb27b-e93d-4f95-8895-fbf41e2fe5e9/image.png)

## Sử dụng `-webkit-box-decoration-break: clone`

Giải pháp mình tìm được gần đây là sử dụng `-webkit-box-decoration-break: clone`, một CSS property mới được thêm vào từ Chrome 130, giúp tuỳ chỉnh cách hiển thị khi element xuống dòng (đọc thêm [tại đây](https://developer.mozilla.org/en-US/docs/Web/CSS/box-decoration-break)), bạn có thể với button phía trên để thấy sự khác biệt.

```css
.skeleton-text {
  -webkit-box-decoration-break: clone;
  box-decoration-break: clone;
}
```

## Gợi ý cải tiến thêm

- Dùng `TextContext` của `react-aria-components` để đẩy trạng thái loading xuống các Text, bỏ qua việc if else trên từng element.  
- Tương tự với các component khác như Image, Tabs,…  
- Tạo 1 component `Skeleton` wrap lại toàn bộ content thay vì if else trên từng element, cân nhắc làm thêm `SkeletonContext`.  
- Smart Skeleton: sau khi nhận được response từ API, lưu `totalPosts` vào localStorage và dùng cho lần render Skeleton kế tiếp để số lượng Skeleton khớp với thực tế (ví dụ 3) thay vì cố định 10/20.