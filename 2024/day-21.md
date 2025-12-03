**Author:** huytd

---

# Deploy Go lên Vercel và làm ứng dụng hóng hớt | Huy's Blog

Có thể bạn biết rồi, ngoài JavaScript, TypeScript, NodeJS, Next.js các kiểu ra thì Vercel còn hỗ trợ deploy cả Go nữa.

![](https://notes.huy.rocks/apple-touch-icon.png)

https://notes.huy.rocks/posts/go-on-vercel.html

![](https://notes.huy.rocks/img/default.jpg)

---

Trước tiên, là nội dung chính đúng như tiêu đề:

Có thể bạn biết rồi, ngoài JavaScript, TypeScript, NodeJS, Next.js các kiểu ra thì Vercel còn hỗ trợ [deploy cả Go nữa](https://vercel.com/docs/functions/runtimes/go).

Mặc định code được deploy trên Vercel sẽ sử dụng môi trường Node.js, nhưng nếu Vercel detect được file `go.mod` trong project root của bạn, thì nó sẽ chuyển qua Go runtime. Mỗi một file `*.go` trong thư mục `/api` của bạn sẽ là một API endpoint (với điều kiện là file đó có export một function kiểu [`HandlerFunc`](https://golang.org/pkg/net/http/#HandlerFunc)).

## Cấu trúc thư mục ví dụ

Ví dụ với cấu trúc thư mục sau:

```bash
.
├── api
│   └── feed.go
├── go.mod
└── index.html
```

Lưu ý là các endpoint này có thể được request với bất kỳ method gì, nếu muốn giới hạn từng method với từng endpoint cụ thể thì bạn phải kiểm tra trong handler function:

```go
func handler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }

    // handle GET...
}
```

Rồi thế là xong phần nội dung chính. Ngắn gọn dễ hiểu, tuy nhiên bài ngắn quá, không vui tí nào. Hãy thử ứng dụng kiến thức này để làm một cái gì đó vui vẻ tí xem sao.

---

## Làm app “Hacker Live”

Chúng ta sẽ build một trang web tự động hiển thị các comment mới nhất trên Hacker News, vì nếu như Giáng sinh này, bạn chỉ có một mình và không đi đâu (không như mình, lúc các bạn đọc được bài này thì mình đang trên đường lái xe đi nghỉ lễ rồi 😏), thì có thể dùng nó để xem thiên hạ đang bàn tán gì, mình đặt tên cho nó là **Hacker Live** cho dễ hình dung.

Tạo một thư mục mới, init Go module rồi tạo file với cấu trúc như sau:

```bash
mkdir hacker-live
cd hacker-live
go mod init hacker-live

.
├── api
│   └── feed.go
└── index.html
```

## Cơ chế hoạt động

Cơ chế hoạt động của Hacker Live được mô tả qua sơ đồ sau:

![](https://notes.huy.rocks/posts/img/hacker-live-diagram.png)

Chúng ta sẽ có một API endpoint tên là `/api/feed` có nhiệm vụ fetch các comment mới nhất trên Hacker News về, dữ liệu comment chúng ta lấy từ RSS feed (<https://hnrss.org/newcomments>).

---

## Phần UI với htmx

Phần UI, để cho tiện thì chúng ta sẽ dùng [htmx](https://htmx.org/docs/), giới thiệu nhanh gọn thì đây là một thư viện cho phép chúng ta sử dụng rất nhiều những tính năng của trình duyệt như AJAX, WebSocket, Form Validation,... mà không cần viết tí JavaScript nào.

Cụ thể, ở đây chúng ta chỉ cần dùng `hx-get` để fetch nội dung từ `/api/feed`, và hiện nó lên trang.

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Hacker Live</title>
    <script src="https://unpkg.com/htmx.org@1.9.2"></script>
  </head>
  <body>
    <div
      id="feed"
      hx-get="/api/feed"
      hx-trigger="load, every 30s"
      hx-swap="innerHTML"
    >
      Loading...
    </div>
  </body>
</html>
```

Trong đoạn HTML trên, `hx-trigger="load, every 30s"` nghĩa là chúng ta bắt đầu fetch dữ liệu khi trang web vừa được load lên, sau đó cứ mỗi 30 giây thì fetch lại một lần. Dòng chữ "Loading..." sẽ được thay thế bằng nội dung trả về từ API.

---

## API trả về HTML (code Go nhưng tâm hồn PHP)

Ở phía API, thay vì trả về JSON thì chúng ta chỉ cần trả về HTML trực tiếp:

```go
package main

import (
    "encoding/xml"
    "fmt"
    "html/template"
    "net/http"
)

type RSS struct {
    Channel struct {
        Items []struct {
            Title string `xml:"title"`
            Link  string `xml:"link"`
        } `xml:"item"`
    } `xml:"channel"`
}

func Feed(w http.ResponseWriter, r *http.Request) {
    resp, err := http.Get("https://hnrss.org/newcomments")
    if err != nil {
        w.WriteHeader(http.StatusInternalServerError)
        return
    }
    defer resp.Body.Close()

    var rss RSS
    if err := xml.NewDecoder(resp.Body).Decode(&rss); err != nil {
        w.WriteHeader(http.StatusInternalServerError)
        return
    }

    tmpl := `
<ul>
  {{ range . }}
    <li><a href="{{ .Link }}" target="_blank">{{ .Title }}</a></li>
  {{ end }}
</ul>
`
    t := template.Must(template.New("feed").Parse(tmpl))
    w.Header().Set("Content-Type", "text/html; charset=utf-8")
    _ = t.Execute(w, rss.Channel.Items)
}
```

---

## Deploy lên Vercel

Rồi, giờ thì chỉ việc deploy lên Vercel:

```bash
vercel
# hoặc
vc
```

Và thế là, tèn ten, có ngay một ứng dụng để ngồi FOMO: <https://hackerlive.vercel.app/>

![](https://notes.huy.rocks/posts/img/hacker-live-demo.png)

Trong bài có lược bỏ đi một phần code, các bạn có thể tham khảo source code đầy đủ tại <https://github.com/huytd/hacker-live>