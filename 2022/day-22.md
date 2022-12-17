---
title: Có cần sử dụng debounce trong React 18 không?
author: huy
date: 12-15-2022
---

## Mở đầu

Trong bài viết này sẽ mình sẽ đi qua một ví dụ điển hình khi nhắc đến debounce và giới thiệu cho mọi người một cách tiếp cận khác trong **React 18**.

Mọi người lưu ý trong bài viết này assume mọi người sử dụng những công nghệ fetch dữ liệu hỗ trợ `Suspense` nha.

## Sử dụng useDebounce

Khi cài đặt ô tìm kiếm, chúng ta thường sử dụng `useDebounce` để đợi cho sau khi người dùng dừng việc gõ thì mới bắt đầu tiến hành gọi API với nội dung cuối cùng.

Bằng cách này, chỉ khi người dùng dừng việc gõ phím lâu hơn 200ms thì `debouncedQuery` được cập nhật, tiếp theo component `<SearchResult>` fetch kết quả với query mới và trong lúc chờ kết quả thì sẽ được fallback về `Suspense` để hiển thị **"Loading..."** sau đó mới đến kết quả.

![useDebounce](https://user-images.githubusercontent.com/66771508/208147462-462e7fc5-12b1-47f0-8ddb-c8697a38cdc9.gif)

## Sử dụng useDeferredValue

**React 18** cung cấp cho người dùng một hook mới là `useDeferredValue`. Giá trị trả về của hook này là một state và chỉ được cập nhật sau khi **React** đã hoàn thành các render được ưu tiên.

Vẫn là ô tìm kiếm giống bên trên, **React** sẽ ưu tiên render lại ô input ngay lập tức khi người dùng gõ nội dung. Sau khi người dùng kết thúc gõ, việc render ô input không còn nữa, lúc này, **React** sẽ cập nhật lại `deferredQuery` thành giá trị mới nhất của `query` và tiến hành render lại một lần nữa nhưng với giá trị mới của `deferredQuery`

![useDeferredValue](https://user-images.githubusercontent.com/66771508/208147822-6c710c67-0749-458a-8b33-76e9fe80c040.gif)

## Khác nhau giữa useDebounce và useDeferredValue

1. `useDeferredValue` không có thời gian chờ cố định, ngay sau khi **React** hoàn thành hết những render được ưu tiên hơn sẽ tiến hành render lại một lần nữa với giá trị mới nhất của state được truyền vào trong `useDeferredValue`

2. Vì **React** sẽ tiến hành render lại những components liên quan đến state trả về từ `useDeferredValue` trong nền và cập nhật lại nên nếu bạn để ý sẽ thấy khi `<SearchResult>` cập nhật, không hề thấy trạng thái fallback về Suspense, vì vậy cũng **không** hiển thị **"Loading..."** giống như cách trên.

## Tổng kết

Mong là thông qua những thông tin trên đã giúp mọi người nắm bắt được cách tiếp cận mới này, tuỳ vào mục đích sử dụng thì mình để mọi người tự cân nhắc nên sử dụng cái nào khi nào nha.

Để tìm hiểu thêm về hook mới này, mời mọi người đọc tiếp bài viết tại [useDeferredValue | React Docs (BETA)](https://beta.reactjs.org/apis/react/useDeferredValue)

Ngoài ra, mình cũng có tạo một CodeSandbox để mọi người có thể vọc vạch tại [distracted-dew-diu7qe - CodeSandbox](https://codesandbox.io/s/distracted-dew-diu7qe?file=/App.js)