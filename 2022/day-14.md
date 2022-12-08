---
title: Reverse proxy/API Gateway on Vercel with vercel.json rewrites or Next.js rewrites
author: ducan-ne
date: 12-07-2022
---
Làm việc thực tế, đôi lúc bạn sẽ cần serve blog trên sub path của domain như domain.com/blog hay domain.com/admin.

Với website host theo cách cũ bạn có thể tạo 1 load balancer và trỏ tới *blog* project theo path /blog

Nhưng Vercel (hiện tại) không có chức năng load balancer.

Tuy nhiên chúng ta có thể tận dụng vercel.json rewrites để thực hiện, ví dụ như sau:

```json
{
  "rewrites": [{
    "source": "/blog/:match*",
    "destination": "https://blog.facebook.com/:match*"
  }, {
    "source": "/:match*",
    "destination": "https://landing.facebook.com/:match*"
  }],
  // optional
  "trailingSlash": false
}
```

với config như trên bạn sẽ trỏ request /blog vào project blog, còn lại sẽ trỏ vào landing.

Next.js rewrites và Vercel.json rewrites có cơ chế và config scheme tương tự nhau, bạn có thể tạo 1 project mới để chứa file config này hoặc sử dụng chính Next.js config ở landing project.

Nếu bạn cần một reverse proxy cấu hình flexible hơn thì có thể tham khảo reflare: https://github.com/xiaoyang-sde/reflare

Bonus tips: ở bài [AoF trước](/2022/day-04.md) mình đã nói về việc clone website, trong 1 vài trường hợp bạn clone website nhưng có quá nhiều video nặng ở website đích (vd hơn 100mb), bạn có thể dùng vercel.json để proxy tới trang đích (bỏ qua cors) để có thể streaming resources và tận dụng cache của trình duyệt, ví dụ như sau:

```json
    {
      "source": "/fonts/:match*",
      "destination": "https://facebook.com/fonts/:match*"
    },
```

Nhưng hãy dùng cẩn thận vì nó có thể sẽ tốn nhiều bandwidth gói Vercel của bạn

Tips:

- Vercel's rewrites sử dụng nodejs http-proxy.
- Có thể sử dụng làm api gateway.
- Sử dụng edge function sẽ có performance tốt hơn.

Next.js self-hosted cũng có thể dùng rewrites config làm việc tương tự.

References:

- https://vercel.com/docs/project-configuration
- https://nextjs.org/docs/api-reference/next.config.js/rewrites#rewriting-to-an-external-url
