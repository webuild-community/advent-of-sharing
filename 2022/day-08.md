---
title: Runtime check với Literal Union Types trong TypeScript
author: huytd
date: 12-01-2022
---

Literal Union Types là một cách rất tiện lợi để giới hạn một kiểu vào một tập các giá trị literal (string literal), ví dụ:

```typescript
type AllowedNames = "Cam" | "Xoài" | "Chuối"
```

Nhưng mà mọi thứ feature hay ho trong TypeScript thì chỉ tồn tại ở compile time, sau khi build ra JS code là xong.

Một ví dụ là, ở đây mình muốn tận dụng kiểu `AllowedNames` để viết một hàm chỉ nhận vào tham số có giá trị khớp với một trong những giá trị `Cam`, `Xoài`, `Chuối` của kiểu `AllowedNames` ở trên. Nhưng ngặt một nỗi, những giá trị đó chỉ tồn tại ở type level, không phải ở runtime.

Quèo, thật ra vẫn có cách để giải quyết vấn đề này!

Đầu tiên, thay vì define kiểu `AllowedNames` trực tiếp từ những giá trị string literal như ban đầu, thì chúng ta sẽ tạo một mảng const chứa những giá trị đó, rồi define kiểu `AllowedNames` từ mảng đó, bằng cách dùng từ khóa `typeof` như sau:

```typescript
const allowedNames = ["Cam" | "Xoài" | "Chuối"] as const;
type AllowedNames = typeof allowedNames[number];
//                = "Cam" | "Xoài" | "Chuối"
```

Rồi, bằng cách này, chúng ta vừa có danh sách `allowedNames` ở runtime, vừa có kiểu `AllowedNames` ở type level luôn! Việc cần làm chỉ là viết hàm kiểm tra giá trị theo yêu cầu ban đầu:

```typescript
const isNameAllowed = (input: unknown): input is AllowedNames => {
    return typeof input === 'string' && allowedNames.includes(input as AllowedNames);
};

const checkName = (input: string): AllowedNames => {
    if (isNameAllowed(input)) {
        return input;
    } else {
        throw new Error("Please use one of the allowed name!");
    }
}

console.log(checkName("Chuối")); // Output: "Chuối"
console.log(checkName("Cà Rốt")); // [ERR]: Please use one of the allowed name!
```

Quá xịn!

Ủa, nhưng mà vì sao đoạn code trên lại chạy? `as const` là cái gì? `typeof cái_enum[number]` là cái gì???

Đây là khúc bản chất hại não của TypeScript bắt đầu.

Đầu tiên, sẽ như thế nào nếu ta ko dùng từ khóa `as const`?

```typescript
const allowedNames = ["Cam" | "Xoài" | "Chuối"];
type t = typeof allowedNames; // t == string[]
type t0 = typeof allowedNames[0]; // t0 == string
type t1 = typeof allowedNames[1]; // t1 == string
```

Rất đơn giản, mảng `allowedNames` sẽ là một mảng của các `string`, và mỗi phần tử trong mảng `allowedNames` sẽ có kiểu `string`.

Từ khóa `as const` (hay còn gọi là [const assertion](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)) có tác dụng báo cho TypeScript compiler biết rằng, các phần tử của mảng `allowedNames` là các giá trị `readonly`. Điều này có nghĩa là gì? Có nghĩa là, các phần tử này không còn khả năng thay đổi giá trị nữa, suy ra kiểu của mỗi phần tử trong mảng này chắc chắn sẽ là các literal type với chính giá trị của nó, chứ không nhất thiết phải là một kiểu rộng như `string` nữa.

```typescript
const allowedNames = ["Cam" | "Xoài" | "Chuối"] as const;
type t = typeof allowedNames; // t == readonly ["Cam" | "Xoài" | "Chuối"]
type t0 = typeof allowedNames[0]; // t0 == "Cam"
type t1 = typeof allowedNames[1]; // t1 == "Xoài"
```

Như thế là xong phần `as const`. 

Tiếp, như các bạn thấy ở 2 ví dụ trên, ta có thể lấy ra type của một phần tử bằng cách dùng từ khóa `typeof <Mảng>[index]`. Còn cách dùng `typeof <Mảng>[number]` là một dạng [Indexed Access Type](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html), trả về một union chứa **tất cả các kiểu có thể có** của **tất cả mọi phần tử** có trong mảng. Xài `number` là vì chúng ta truy xuất đến các phần tử của mảng bằng một index kiểu `number`.

Như vậy, `typeof allowedNames[number]` sẽ trả về một union type chứa kiểu của tất cả mọi phần tử trong mảng `allowedNames`, và các kiểu ấy là `"Cam" | "Xoài" | "Chuối"`.

Hy vọng qua bài viết này, các bạn cũng thấy thêm được sự lợi hại của TypeScript. Riêng mình thì đã thấy nó rất chi là hại não rồi.
