**Author:** An

---

# Understanding Streaming by Building a Simple One

[**Understanding Streaming by Building a Simple One**  
Learn how streaming works by building a simple streaming server  
![](https://an.cyou/favicon.png)  
https://an.cyou/blog/understanding-streaming-by-building-a-simple-one  

![](https://ogcool.vercel.app/templates/v1/tpl_TnllPg0CkP?d=eyJtb2RpZmljYXRpb25zIjpbeyJuYW1lIjoiVGV4dCIsInRleHQiOiJVbmRlcnN0YW5kaW5nIFN0cmVhbWluZyBieSBCdWlsZGluZyBhIFNpbXBsZSBPbmUifV19&sdk=ogcool%400.1.10)  
](https://an.cyou/blog/understanding-streaming-by-building-a-simple-one)

You’ve probably heard about streaming before. With the rise of AI, React RSC, and other new technologies, streaming is becoming more and more popular.

But what exactly is streaming? How does it work? Many developers use it when building applications, but not everyone understands how it actually works under the hood.

In this article, I’ll build a simple streaming server that works similarly to RSC.

---

### What is Streaming?

According to the [Streams Standard](https://streams.spec.whatwg.org/#intro):

> Large swathes of the web platform are built on streaming data: that is, data that is created, processed, and consumed in an incremental fashion, without ever reading all of it into memory.

Simply put, streaming is a way to handle data by sending it in chunks, instead of all at once, like this:

> Data is processed and delivered incrementally, rather than buffering the entire payload first.

Not only is there less data to load at a time, but it also provides a way to get new data from the server in real time in the same request. The most common case we all know is text completion from AI.

---

### `Transfer-Encoding: chunked`

This header is the key to making streaming work. By setting this header, the server sends data in chunks instead of all at once.

> Data is sent in a series of chunks. Content can be sent in streams of unknown size to be transferred as a sequence of length-delimited buffers, so the sender can keep a connection open, and let the recipient know when it has received the entire message.

Read more about `Transfer-Encoding: chunked` at [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding#chunked).

---

### How does RSC use streaming? How does `<Suspense>` work?

RSC uses streaming to progressively render components and send them to the client. When a component is wrapped in `<Suspense>`, React can start streaming the HTML for other components while waiting for the suspended component to load.

**How it works under the hood:**

1. When React encounters a `<Suspense>` boundary, it renders a loading fallback initially.
2. The suspended component (e.g. one fetching data) continues processing on the server.
3. Once the component is ready, React streams the HTML for that component as a new chunk.
4. Special JavaScript instructions are included to replace the loading state (the fallback) with the new content.

**Example (simplified):**

_First chunk (sent immediately):_

```html
<div id="user-profile">
  <p>Loading data…</p>
</div>
```

_Second chunk (sent later):_

```html
<div data-rsc-chunk="user-profile">
  <h1>John Doe</h1>
  <p>Software engineer</p>
</div>
<script>
  // Simplified: replace loading state with streamed content
  const target = document.getElementById('user-profile');
  const chunk = document.querySelector('[data-rsc-chunk="user-profile"]');
  target.replaceWith(chunk);
</script>
```

The second chunk runs a script to replace the loading state with the new content. This is how RSC works under the hood.

The real RSC implementation is more complex, but the core idea is the same. You can look at the [Playground](https://demystifying-rsc.vercel.app/server-components/streaming/) for more details.

---

### Build a Simple Streaming Server

If the previous section feels a bit abstract, let’s build a simple streaming server to understand how it works.

We already know how RSC works under the hood, so we can build a simple streaming server to simulate RSC-like behavior.

#### 1. A simple “Hello World” streaming server

Using [Hono](https://hono.dev/) (which supports running in many environments, including edge/browsers):

```ts
import { Hono } from 'hono';

const app = new Hono();

app.get('/', (c) => {
  const stream = new ReadableStream({
    start(controller) {
      controller.enqueue(new TextEncoder().encode('Hello World'));
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/html; charset=utf-8',
      'Transfer-Encoding': 'chunked',
    },
  });
});

export default app;
```

#### 2. Add a `Loading...` state

```ts
app.get('/loading', (c) => {
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();

      // Send initial loading UI
      controller.enqueue(encoder.encode('<p>Loading...</p>'));

      // Simulate async work
      await new Promise((resolve) => setTimeout(resolve, 2000));

      // Send final content
      controller.enqueue(encoder.encode('<p>Data loaded!</p>'));

      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/html; charset=utf-8',
      'Transfer-Encoding': 'chunked',
    },
  });
});
```

#### 3. Multiple components & skeleton loading

```ts
app.get('/page', (c) => {
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();

      // Initial shell + skeleton
      controller.enqueue(
        encoder.encode(`
          <html>
            <body>
              <h1>My Page</h1>
              <div id="user">
                <p class="skeleton">Loading user...</p>
              </div>
              <div id="posts">
                <p class="skeleton">Loading posts...</p>
              </div>
            </body>
          </html>
        `),
      );

      // Simulate user loading
      await new Promise((r) => setTimeout(r, 1000));
      controller.enqueue(
        encoder.encode(`
          <script>
            document.getElementById('user').innerHTML =
              '<h2>User: John Doe</h2>';
          </script>
        `),
      );

      // Simulate posts loading
      await new Promise((r) => setTimeout(r, 1500));
      controller.enqueue(
        encoder.encode(`
          <script>
            document.getElementById('posts').innerHTML =
              '<ul><li>Post 1</li><li>Post 2</li></ul>';
          </script>
        `),
      );

      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/html; charset=utf-8',
      'Transfer-Encoding': 'chunked',
    },
  });
});
```

In real-world implementations, we need to handle many additional concerns (error handling, hydration, ordering, security, etc.), but this example demonstrates the core concepts of streaming responses.

---

### Hono tip: Streaming JSX

Hono provides built-in support for streaming JSX components. Under the hood it uses the same technique as the previous example, making it easier to write streaming HTML responses.

```tsx
/** @jsx jsx */
import { Hono } from 'hono';
import { jsx } from 'hono/jsx';

const app = new Hono();

const Page = () => (
  <html>
    <body>
      <h1>Streaming JSX with Hono</h1>
      <SuspenseSection />
    </body>
  </html>
);

const SuspenseSection = () => (
  <div>
    <p>Loading...</p>
  </div>
);

app.get('/', (c) => c.html(<Page />));

export default app;
```

---

### Conclusion

In this article, we built a simple streaming server to understand how streaming works.

We also learned how RSC leverages streaming to progressively render components and send them to the client.

The future is bright with AI, and streaming makes it even more powerful.

Happy coding!