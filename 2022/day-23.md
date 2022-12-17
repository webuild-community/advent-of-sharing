---
title: Tối ưu Docker Image Size cho Next.js App
author: hai
date: 12-16-2022
---

Không phải công ty nào cũng sẵn sàng bỏ tiền để dùng Vercel, hoặc có đội ngũ DevOps trợ giúp bạn tận răng trong việc triển khai dự án.

Dưới đây là cách bạn có thể tối ưu Docker Image Size khi dùng Next.js.

## Quản lý package dependencies

Đây là nơi khởi đầu cho vấn đề phình to bundle size, cần phân chia rạch ròi những package nào thuộc `dependencies` hoặc `devDependencies`.

Cách của mình:

- Run-time => dependencies
- Build-time | Compiled-time => devDependencies

Bonus: Dùng Must match `version` exactly thay cho Compatible with `^version`

```json
{
  "dependencies": {
    "@next/font": "13.0.6",
    "@vercel/analytics": "0.1.5",
    "classcat": "5.0.4",
    "crypto-js": "4.1.1",
    "gray-matter": "4.0.3",
    "next": "13.0.6",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "react-markdown": "8.0.4",
    "remark-frontmatter": "4.0.1",
    "remark-gfm": "3.0.1"
  },
  "devDependencies": {
    "@tailwindcss/typography": "0.5.8",
    "@types/crypto-js": "4.1.1",
    "@types/node": "18.11.11",
    "@types/react": "18.0.26",
    "@types/react-dom": "18.0.9",
    "autoprefixer": "10.4.13",
    "eslint": "8.29.0",
    "eslint-config-next": "13.0.6",
    "postcss": "8.4.19",
    "tailwindcss": "3.2.4",
    "typescript": "4.9.3"
  }
}
```

## .dockerignore

Để tránh copy những file không cần thiết ta nên dùng `.dockerignore` để quản lý, cách hoạt động cũng tương tự như `.gitignore`

```.dockerignore
node_modules
.git
```

## Base Image

Dùng `alpine` tag cho Node.js để giảm dung lượng base image

```diff
# ~1.5GB vs ~0.3GB
- FROM node:18
+ FROM node:18-alpine
```

Lưu ý không bao giờ dùng `latest` tag.

## Multi-stage builds

Ví dụ Dockerfile không dùng multi-stage, đây là cách đơn giản nhất để dockerize một Next.js app

```Dockerfile
FROM node:18-alpine

WORKDIR app

COPY yarn.lock .
COPY package.json .
# No need to add `--frozen-lockfile` flag
# Because I use Must match version exactly
RUN yarn

COPY . .
RUN yarn build

ENTRYPOINT ["yarn", "start"]
```

Vấn đề với cách trên là chúng ta đang gộp chung phần `devDependencies` khi install package, điều này khiến cho image đang bị phình to vì dung lượng `node_modules` phải chứa những package không cần thiết cho production.

Nếu bạn nghĩ giải pháp là thay `RUN yarn` bằng `RUN yarn --production` thì app của bạn sẽ crash vì thiếu `devDependencies`, một số package như Tailwind cần phải được output ở build-time.

Để khắc phục ta sẽ dùng đến multi-stage builds như sau:

```Dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR app

COPY yarn.lock .
COPY package.json .

RUN yarn

COPY . .
RUN yarn build

# Run stage
FROM node:18-alpine AS runner

WORKDIR app

COPY --from=builder /app/yarn.lock .
COPY --from=builder /app/package.json .

RUN yarn --production

COPY --from=builder /app/next.config.js .
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./.next

ENTRYPOINT ["yarn", "start"]
```

Như vậy là ta đã loại bỏ được `devDependencies` đi theo `node_modules`. Tuy nhiên cách này vẫn chưa thực sự giảm dung lượng hiệu quả, với một Next.js app cơ bản cũng sẽ chiếm khoảng ~500MB docker image size.

## Standalone output mode

Next.js hỗ trợ [Automatically Copy Traced Files](https://nextjs.org/docs/advanced-features/output-file-tracing#automatically-copying-traced-files) để tạo ra một `standalone` folder chỉ chứa những files cần thiết cho production deployment, bao gồm chọn lọc cả các files trong `node_modules`.

Trước tiên cần enable thông qua option trong `next.config.js`

```js
module.exports = {
  output: "standalone",
};
```

Sau đó đổi lại Dockerfile config như sau:

```Dockerfile
# Build stage
# ...
# Run stage
FROM node:18-alpine AS runner

WORKDIR app

COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

ENTRYPOINT ["node", "server.js"]
```

Với `standalone`, dung lượng Docker image size giảm xuống khoảng ~180MB.

Đọc thêm docs để biết thêm trade-offs và caveats nhé.
