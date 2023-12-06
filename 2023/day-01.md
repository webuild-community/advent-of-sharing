---
title: Function as useEffect's dependency
author: huytd
date: 2023-12-01
---

Khi sử dụng ESLint trong một dự án React, kiểu gì bạn cũng sẽ gặp tình huống mà bạn cho là vớ vẩn như sau: Bạn có một đoạn code xài `useEffect`, trong đó bạn gọi một function, thường thì function này được tạo ra ngay trong component hiện tại luôn, ví dụ:

```js
const doSomething = (someValue) => {
    // ...do something with someValue...
}

useEffect(() => {
   if (someValue) {
      doSomething(someValue)
   }
}, [someValue])
```

Và bạn sẽ bị ESLint quăng ngay một câu complain vô mặt:

```
React Hook useEffect has a missing dependency: 'doSomething'. Either include it or remove the dependency array
```

Đây là warning từ plugin `eslint-plugin-react-hooks`, đại ý của nó là: "Ê sao mày xài hàm doSomething trong useEffect, mà không cho doSomething vào danh sách dependencies?"

Tào lao hết sức! – bạn nghĩ. Nó là một cái hàm chứ có phải value quái gì đâu mà phải bỏ vô dependency list???

Vậy đây có phải là bug của `eslint-plugin-react-hooks` không? Đáng ra chỉ nên áp dụng rule này lên các giá trị thông thường thôi, sao còn áp dụng với cả function?

Lý do là: Function cũng là các object, và trong React (functional component), chúng cũng được khởi tạo lại sau mỗi lần render giống như mọi object khác. Vậy nên, dù function của bạn không đổi, thì trên thực tế sau mỗi lần render, chúng ta đều có một instance mới của cái function đó được tạo ra.

Để ví dụ rõ hơn, có thể xem qua trường hợp chúng ta dùng giá trị của một state ở trong function:

```js
const [name, setName] = useState("")

const sayHello = () => {
   console.log(`Hello, ${name}`)
}

useEffect(() => {
   sayHello()
}, [name])
```

Ở ví dụ trên, vì `name` là một state value, nó có thể thay đổi bất cứ lúc nào và khiến cho component bị re-render. Hàm `sayHello` sử dụng giá trị của name, nên đương nhiên nó là một đối tượng có khả năng thay đổi giống như state.

Chính vì thế mà các function cũng được đưa vào diện cần đưa vào dependency list của `useEffect`.

Rồi, đã hiểu vấn đề, vậy khi gặp tình huống này thì làm sao để fix? Có nhiều cách để giải quyết vấn đề này, tuỳ theo implementation của function cần xử lý.

**Nếu function không sử dụng value hay state nào của component hiện tại** thì có thể move nó ra khỏi component:

```js
const sayHello = (name: string) => {
   console.log("Hello")
}

const Component = () => {
   const [name, setName] = useState("")
      
   useEffect(() => {
      sayHello(name)
   }, [name])
}
```

**Nếu function có sử dụng value hay state thuộc component hiện tại**, thì có thể define function này bên trong khối `useEffect`:

```js
const Component = () => {
   const [name, setName] = useState("")
   const [age, setAge] = useState("")
      
   useEffect(() => {
      const sayHello = () => {
         console.log(`I am ${name}, I am ${age} yo`)
      }
   
      sayHello()
   }, [name, age])
}
```

Lưu ý là, bạn phải đưa luôn toàn bộ những state mà function đó sử dụng vào dependency list luôn (vì function của bạn phụ thuộc vào những state đó).

Một cách khác là **sử dụng useCallback hook để memoize function này sau mỗi lần render**, tuỳ thuộc vào giá trị của các state mà function này sử dụng:

```js
const Component = () => {
   const sayHello = useCallback(() => {
      ...
   }, [/* danh sách state mà function cần */])
      
   useEffect(() => {   
      if (age > 0) {
          sayHello()
      }
   }, [age, sayHello])
}
```
Xài cách này, bạn vẫn phải đưa function này vào danh sách dependency như cách trước, nhưng có sự phân chia trong danh sách dependency rõ ràng hơn.
