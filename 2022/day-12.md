---
title: Avoid extra useless function in React with useReducer
author: monodyle
date: 12-05-2022
---
Có một mẹo nhỏ khi code react đó là bạn có thể dùng `useReducer` để tránh việc viết một cái state và một cái callback function chỉ để toggle cái state đó.

Ví dụ, thông thường làm việc với field password, đôi lúc bạn sẽ cần phải code cái icon con mắt ẩn/hiện password theo UI của design system thay vì dùng cái native của browser, hay nói cách khác là trigger việc toggle một state boolean. Vậy thì bình thường (mình đã tham khảo rất nhiều trang [trên mạng](https://melvingeorge.me/blog/show-or-hide-password-ability-reactjs) <- ví dụ) mọi người thường code như thế này:

```typescript
const [passwordShown, setPasswordShown] = useState(false)

const togglePasswordShown = () => {
  setPasswordShown(!passwordShown)
} // thậm chí nhiều lúc còn phải wrap nó lại trong một cái useCallback :(

return (
  <div>
    <input type={passwordShown ? "text" : "password"} />
    <button onClick={togglePasswordShown}>Show Password</button>
  </div>
)
```

Nhưng đôi khi chúng ta có thể đơn giản hóa nó lại nếu bạn biết dùng reducer, dĩ nhiên không phải kiểu dispatch siêu phức tạp kiểu redux mà [document của react](https://beta.reactjs.org/apis/react/useReducer#usereducer) hướng dẫn.

Bạn có thể làm gọn cái component trên như sau:

```typescript
const [passwordShown, togglePasswordShown] = useReducer(value => !value, false)

return (
  <div>
    <input type={passwordShown ? "text" : "password"} />
    <button onClick={togglePasswordShown}>Show Password</button>
  </div>
)
```

Đôi khi mình còn dùng reducer cho nhiều mục đích khác nhau, ví dụ việc force reload một component:

```typescript
const [lastUpdate, refresh] = useReducer(() => Date.now(), Date.now())
```

Thế thôi.
