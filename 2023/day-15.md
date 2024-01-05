---
title: Sử dụng useSyncExternalStore để subscribe external data changes
author: ducan-ne
date: 2023-12-23
---

# Sử dụng useSyncExternalStore để subscribe external data changes

useSyncExternalStore là 1 api khá mới của React giúp chúng ta subscribe tới external data, Cùng với trend chuyển state ra khỏi ứng dụng React, sử dụng hook này giúp code chúng ta ngắn gọn hơn rất nhiều, và cũng hạn chế được việc sử dụng useEffect

useSyncExternalStore nhận vào 2 params:
- subscribe function subscribe vào store và trả về 1 function để unsubscribe
- getSnapshot function để đọc snapshot từ store

Một số use cases của useSyncExternalStore
- Subscribe tới browser API
- Subscribe tới external data store (jotai, nanostore, redux, …)
- Micro FE communication ở component level
- Third-party libraries

Một số ví dụ sử dụng useSyncExternalStore
- Online indicator từ official docs
```javascript
function getSnapshot() {
return navigator.onLine;
}

function subscribe(callback) {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
      window.removeEventListener('online', callback);
      window.removeEventListener('offline', callback);
    };
}

const isOnline = useSyncExternalStore(subscribe, getSnapshot);
```

- Viết một hook mới implement lại react query từ query core, tên là useQuerySelector
```javascript
const queryClient = new QueryClient()

const query = queryOptions({
  queryFn() {
    return {
      nested: {
        todos: [],
      },
      anotherField: {},
    }
  },
  queryKey: ['todos'],
})
queryClient.ensureQueryData(query)
const useQuerySelector = ({ queryKey }, selector) => {
  return useSyncExternalStore(queryClient.getMutationCache().subscribe, () => selector(queryClient.getQueryData(queryKey)))
}

const data = useQuerySelector(query, (data) => data.nested.todos)
```
Ở ví dụ này useQuerySelector subscribe vào api của queryClient để biết khi nào cache thay đổi thì gọi vào queryData để lấy data mới nhất.

Fun fact: react query thực tế cũng sử dụng useSyncExternalStore https://github.com/TanStack/query/blob/main/packages/react-query/src/useBaseQuery.ts#L67

Ko chỉ queryClient mà bất kì store nào đều có thể áp dụng với cách tương tự như trên, ví dụ như jotai, nanostore, hoặc browser api như localStorage, fetch,….

Mọi người có thể đọc thêm tại https://react.dev/reference/react/useSyncExternalStore