**Author:** vthang

---

# vthang.dev | How we migrate a large codebase with module federation

Team mình vừa trải qua một quãng thời gian khó khăn để hoàn thành deadline cho một khách hàng lớn và sau đó không khó để nhận ra những phần break things cả mới cả cũ của dự án. Khi ngồi lại team đã thảo luận các vấn đề và mình chịu trách nhiệm lên plan cho việc refactor lại codebase frontend của dự án. Và sau đây là những kinh nghiệm rút ra được sau khi hoàn thành những bước đầu tiên của việc refactoring này.

> Bài gốc:  
> https://vthang.dev/articles/how_we_migrate_large_codebase_with_module_federation

---

Team mình vừa trải qua một quãng thời gian khó khăn để hoàn thành deadline cho một khách hàng lớn và sau đó không khó để nhận ra những phần *break things* cả mới cả cũ của dự án. Khi ngồi lại team đã thảo luận các vấn đề và mình chịu trách nhiệm lên plan cho việc refactor lại codebase frontend của dự án. Và sau đây là những kinh nghiệm rút ra được sau khi hoàn thành những bước đầu tiên của việc refactoring này.

### Motivation

Động lực lớn nhất để refactor lại toàn bộ codebase là sau khi team nhận ra việc onboarding cho người mới bắt đầu trở nên khó khăn. Cộng với việc dự án đã chạy gần 3 năm mà chưa có giai đoạn nào ngừng nghỉ để update hay xử lý các technical debts:

- Hiện tại project đang sử dụng NextJS phiên bản khá cũ (12.2.0) và nặng nề nên trong quá trình upgrade, mình phải nâng version tới phiên bản 13.5.1 mới có thể sử dụng ổn định module federation với `@module-federation/nextjs-mf` (deprecated) và NextJS đang không còn hợp hay cần thiết với dự án nữa.
- Team đang chưa có thống nhất chung về formatter cũng như linter.
- Styling cực loạn với rất nhiều kiểu khác nhau dẫn đến implement design system bị sai rất nhiều cũng như khó sửa đổi.
- Dependencies được sử dụng chưa hợp lý.
- Team đang sử dụng Typescript không được chặt chẽ.
- Đang có hai dự án lớn nhưng sử dụng chung 1 codebase.
- Và nhiều vấn đề khác nữa.

### Plan

Lúc đầu nghĩ tới việc refactor team thực sự không rõ phải bắt đầu từ đâu. Nếu upgrade từ codebase hiện tại, team sẽ phải sửa từng view, xây lại base components, design system,… dựa trên hệ thống cũ chẳng khác nào đập đi xây lại cả. Đối với những thành viên mới (chiếm đa số trong team) thì làm việc trên codebase mới lúc nào cũng dễ chịu hơn nên đa số đều thoải mái với quyết định này.

Tuy nhiên đập đi xây lại là một bài toán rất lớn về cost mà rất khó để nó có thể được thông qua. Mình đã quyết định làm POC để migrate từng phần của dự án nhưng trên một codebase mới hoàn toàn và sử dụng stack hoàn toàn mới. Để làm điều đó thì chúng ta cần tới **module federation**.

### Module federation

**Module federation** (MF) ra mắt trong phiên bản webpack 5 - là tính năng được coi là game changer trong frontend architecture. MF hỗ trợ load và code sharing at runtime giúp cho việc triển khai micro-frontend cũng trở nên dễ dàng hơn. Tuy nhiên mình chỉ sử dụng MF để migrate codebase thành 2 app nhỏ hơn cho 2 business khác nhau chứ không có ý định xây dựng đến mức micro cho dự án.

Về cơ bản chúng ta sẽ coi app hiện tại như **host app** và codebase mới sẽ nằm trên **remote app**, **remote app** sẽ được mount trên **host app**:

![Module Federation Diagram](https://vthang.dev/mf.png)

Khi này host app (Shell app) sử dụng nextjs (react 18) chạy trên `localhost:3002` còn remote app chạy độc lập với stack tùy chọn, ở đây mình sử dụng react (version 19) với [rspack](https://rspack.dev/).

Khi cài đặt trên remote app, để đơn giản chúng ta sẽ bundle luôn các dependencies vào cùng 1 file:

```js
// Example remote app federation config (minh họa)
module.exports = {
  output: {
    publicPath: 'auto',
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'remoteApp',
      filename: 'remoteEntry.js',
      exposes: {
        './App': './src/App',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
};
```

Còn trên host app với nextjs:

```js
// Example host app webpack config (minh họa)
const { NextFederationPlugin } = require('@module-federation/nextjs-mf');

module.exports = {
  webpack(config) {
    config.plugins.push(
      new NextFederationPlugin({
        name: 'host',
        remotes: {
          remoteApp: 'remoteApp@http://localhost:8080/remoteEntry.js',
        },
      })
    );

    return config;
  },
};
```

Lúc này trên host app chúng ta có thể import và sử dụng module được expose từ remote app:

```tsx
// pages/feature-a.tsx (minh họa)
import dynamic from 'next/dynamic';

const FeatureA = dynamic(() => import('remoteApp/App'), {
  ssr: false,
});

export default function FeatureAPage() {
  return <FeatureA />;
}
```

Vậy là chúng ta đã có thể render remote app trên host app.

### Problems

Tuy nhiên không chỉ đơn giản là render được vài component với text là xong. Chúng ta phải xây dựng cả 1 app mới với đầy đủ các thành phần khác nhau từ đơn giản đến phức tạp. Khi đi sâu vào implement mình bắt đầu nhận ra vài vấn đề:

- CSS conflict
- Routing
- State & Type sharing
- Component library

### CSS isolation

Khi mount remote app lên host app, chúng ta sẽ bắt đầu nhận ra css sẽ break tè le. Vì sao lại vậy? Khi host app và remote app sử dụng component library khác nhau và styling method khác nhau, sẽ có các phần CSS bị conflict:

```css
/* Example conflict (minh họa) */
.button {
  padding: 4px 8px;
  font-size: 12px;
}

/* Remote app cũng có */
.button {
  padding: 12px 16px;
  font-size: 16px;
}
```

Khi này remote app sẽ leak phần CSS có thể gây ảnh hưởng tới style host app của bạn. Mình đã từng nghĩ tới dùng `postcss-prefix` plugin để thêm prefix cho các class của remote app. Nhưng có vẻ somehow vẫn có những style bị leak xuống host app và mình thực sự muốn isolate hoàn toàn style của remote app. Và khi đó mình quyết định sử dụng **shadow DOM** để giải quyết vấn đề này.

Về shadow DOM các bạn có thể xem tại [đây](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM). Nhưng đại ý thì:

> A set of JavaScript APIs for attaching an encapsulated "shadow" DOM tree to an element — which is rendered separately from the main document DOM — and controlling associated functionality. In this way, you can keep an element's features private, so they can be scripted and styled without the fear of collision with other parts of the document.

Lúc này mình sẽ phải custom lại việc mounting remote app.

#### Expose `inject` và `cleanup` functions (remote app)

```tsx
// remote/AppEntry.tsx (minh họa)
import React from 'react';
import { createRoot, Root } from 'react-dom/client';
import { App } from './App';

let root: Root | null = null;

export function inject(container: HTMLElement) {
  if (!root) {
    root = createRoot(container);
  }
  root.render(<App />);
}

export function cleanup() {
  if (root) {
    root.unmount();
    root = null;
  }
}
```

#### Mount trên host app với shadow DOM

```tsx
// host/components/RemoteWrapper.tsx (minh họa)
import React, { useEffect, useRef } from 'react';

// @ts-expect-error remote types
import { inject, cleanup } from 'remoteApp/AppEntry';

export function RemoteWrapper() {
  const hostRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    if (!hostRef.current) return;

    // Tạo shadow root
    const shadowRoot = hostRef.current.attachShadow({ mode: 'open' });

    // Container mà remote app sẽ render vào
    const container = document.createElement('div');
    shadowRoot.appendChild(container);

    inject(container);

    return () => {
      cleanup();
      shadowRoot.innerHTML = '';
    };
  }, []);

  return <div ref={hostRef} />;
}
```

Về lý thuyết thì việc tạo isolated dom tree đã được giải quyết. Nhưng lại phát sinh ra một vấn đề. Tại runtime thì `style-loader` sẽ load CSS lên thẻ `<style></style>` và đẩy chúng lên thẻ `<head>` tại host app. Mà điều này lại hoàn toàn không như kì vọng ban đầu đó là bỏ hết CSS của remote app vào trong **shadow DOM**.

Do vậy mình đã viết custom `style-loader` để inject thẻ `<style>` được sinh ra tại remote app vào trong **shadow DOM** là xong:

```js
// custom-style-loader.js (minh họa ý tưởng)
module.exports = function customStyleLoader(source) {
  const shadowRootVar = 'window.__REMOTE_SHADOW_ROOT__';

  return `
    const style = document.createElement('style');
    style.textContent = ${JSON.stringify(source)};
    if (${shadowRootVar}) {
      ${shadowRootVar}.appendChild(style);
    } else {
      document.head.appendChild(style);
    }
    module.exports = {};
  `;
};
```

Trong khi đó việc dev phiên bản standalone trên remote app vẫn diễn ra bình thường (chỉ cần set `window.__REMOTE_SHADOW_ROOT__` tương ứng hoặc fallback sang `document.head`).

### Routing

Về routing chúng ta có thể tiếp cận theo 2 hướng: để host app lo hoặc routing trên remote app.

Và mình thì chọn theo hướng routing với `@tanstack/react-router` trên remote app. Vì remote app dần dần sẽ trở thành app chính nên việc setup routing chuẩn ngay từ đầu là cực kì quan trọng. Nếu chúng ta chọn routing trên host app thì chắc chắn việc routing trên remote app (standalone) sẽ trở nên phức tạp hơn đồng nghĩa với việc sharing state giữa host app và remote cũng phức tạp hơn.

- Remote app URL: `localhost:8080/feature-a/view-a`  
  → sẽ được mount lên host app thông qua page `localhost:3002/feature-a`

Từ đó tất cả subroute của `feature-a` trên remote app sẽ chỉ cần inject vào 1 page route duy nhất trên host app thông qua cài đặt của nextjs:

```tsx
// host/pages/feature-a.tsx (minh họa)
import dynamic from 'next/dynamic';

const FeatureARemote = dynamic(
  () => import('remoteApp/FeatureARouter'),
  { ssr: false }
);

export default function FeatureAPage() {
  return <FeatureARemote />;
}
```

### Component library

> Không phải tất cả các component library đều hoạt động tốt với shadow DOM.

Thời điểm đầu mình chọn `react-aria-components` để làm component library nhưng ngay sau đó mình gặp phải vấn đề khi thao tác với các component trong shadow dom. Do RAC sử dụng custom events nên một số event như click hay focus đều đang gặp vấn đề nên mình đã chuyển sang Mantine.

Default Mantine sử dụng selector `:root` trong file output nên khi mount vào shadow dom chúng ta sẽ sử dụng custom style-loader để convert thành `:host` hoặc có thể sử dụng `cssVariablesSelector`:

```tsx
import { MantineProvider } from '@mantine/core';

export function App() {
  return (
    <MantineProvider
      cssVariablesSelector=":host"
      theme={{ /* ... */ }}
    >
      {/* ... */}
    </MantineProvider>
  );
}
```

Mantine cũng dễ dàng cho phép thay đổi nơi nào portal children sẽ được render thông qua `portalProps` nếu không chúng sẽ bị render ra ngoài host app chứ không còn nằm trong **shadow DOM** nữa.

Nhờ những cài đặt này mà các component như select, combobox,... sử dụng portal sẽ không bị tính toán sai về position khi render.

Đến giờ, mình tạm yên tâm về **shadow DOM** và vẫn chưa gặp thêm bất kỳ vấn đề khó chịu nào với nó.

### State & Type sharing

Thật ra do port từng page qua một nên vấn đề share state không quá phức tạp. Ví dụ auth, retrieve account info host app đã làm hết và khi đó remote app chỉ cần nhận duy nhất 1 hook là `useAccount` được expose từ host app là có thể hoạt động được như bình thường rồi.

```tsx
// host/hooks/useAccount.ts (minh họa)
export function useAccount() {
  // logic: call API, get user from context, ...
  return { id: '123', name: 'John Doe' };
}

// Expose qua module federation
// host/webpack (hoặc nextjs-mf) exposes: { './useAccount': './hooks/useAccount' }
```

```tsx
// remote/app.tsx (minh họa)
// @ts-expect-error mf remote
import { useAccount } from 'host/useAccount';

export function App() {
  const account = useAccount();
  return <div>Hello {account.name}</div>;
}
```

Lúc đầu mình chưa rõ type sharing như nào sẽ là ổn nhất. Nhưng ngẫm lại việc expose không quá nhiều cũng khiến việc type sharing trở nên không quá quan trọng. Nếu muốn thì `@module-federation/typescript` có thể support vụ này:

```jsonc
// mf.config.json (minh họa)
{
  "compilerOptions": {
    "remoteTypes": {
      "host": "http://localhost:3002/remoteTypes.json"
    }
  }
}
```

### Conclusion

Việc sử dụng Module federation cùng một vài kỹ thuật trong micro-frontend bước đầu đã giúp team trong việc migrate từng phần của codebase cũ sang một codebase mới tốt hơn. Tuy nhiên vẫn có những challenge cần phải được research kỹ trước khi áp dụng vào project bởi những hạn chế hiện tại của **shadow DOM**.

Hy vọng bài viết sẽ giúp ích cho những ai đang có nhu cầu migrate codebase như team mình!