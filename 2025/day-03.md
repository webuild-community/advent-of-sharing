Chào mọi người!!!

Đã chuẩn bị hết năm 2025 rồi, nhưng những bài viết của Advent Of Sharing 2024 vẫn chưa được tạo Pull Request lên [Github](https://github.com/webuild-community/advent-of-sharing) của Webuild. Chắc có một thế lực nào đó trong Webuild ngăn cản mọi người không đăng bài trên [Github](https://github.com/webuild-community/advent-of-sharing)!!!

<img width="811" height="609" alt="Image" src="https://github.com/user-attachments/assets/378052ac-d476-47c7-8ade-6cf978ebdaff" />

Nhân dịp cuối năm nay, mình muốn sao chép hết những bài viết năm 2024 đang ở [Notion](https://we-build-vn.notion.site/13ad61cdcc3580478202fbc845cfa009?v=13ad61cdcc3581be9d9e000cc6720809&p=13ad61cdcc3580159552eab040a9fa11&pm=s&pvs=31) lên [Github](https://github.com/webuild-community/advent-of-sharing). Lý do cho việc này là vì mình thích [Github](https://github.com/webuild-community/advent-of-sharing) hơn là [Notion](https://we-build-vn.notion.site/13ad61cdcc3580478202fbc845cfa009?v=13ad61cdcc3581be9d9e000cc6720809&p=13ad61cdcc3580159552eab040a9fa11&pm=s&pvs=31).

Bài viết này sẽ là hành trình để mình thực hiện dự án nhỏ này.

Vậy là xong việc trình bày lý do!!! Phần tiếp theo mình sẽ chia bài viết ra từng phần nhỏ dựa trên các bước thực hiện ý tưởng này.

## Xác định mục tiêu

Mục tiêu chính của mình sẽ là sao chép hết tất cả các bài viết có trên [Notion](https://we-build-vn.notion.site/13ad61cdcc3580478202fbc845cfa009?v=13ad61cdcc3581be9d9e000cc6720809&p=13ad61cdcc3580159552eab040a9fa11&pm=s&pvs=31) và chuyển đổi nó sang dạng Markdown, rồi commit lên [Github](https://github.com/webuild-community/advent-of-sharing)

Người thực hiện sẽ là mình, mình sẽ dùng bất cứ công cụ nào đấy để có thể chuyển đổi Notion page sang định dạng Markdown, sau đó tạo Pull Request. Mình chấp nhận đây là tool có thể dùng một lần, không cần maintain.

Về chất lượng Markdown file output, mình sẽ cho phép nó ở mức chấp nhận được bao gồm các tiêu chí: không mất nội dung, không mù mắt, không mất hình.

## Phát thảo ý tưởng

Có nhiều hướng để giải quyết bài toán này, có thể dùng no-code tools, có thể làm bằng tay, nhưng vì là một cộng đồng công nghệ, mình ưu tiên viết một công cụ CLI để làm việc đấy thay mình.

<img width="1301" height="237" alt="Image" src="https://github.com/user-attachments/assets/84fb9393-b0a4-46b9-88b5-e96447443ccf" />

Ý tưởng chính của công cụ này là một cái Web Scraper, đi duyệt qua các bài viết Advent Of Sharing 2024 trên Notion và lưu ở máy tính dưới dạng file HTML. Sau đó convert file HTML đã sang Markdown file. Cuối cùng mình commit files đó tạo Pull Request, ý tưởng chính là thế.

## Những vấn đề và các hướng giải quyết

Với ý tưởng như vậy, những vấn đề cần giải quyết bao gồm:

### Dùng techstack nào

Để chọn techstack phù hợp, dự án nhỏ này mình dựa trên các yếu tố sau đây:

 - Side projects, fun projects
 - Không cần maintain
 - Support tốt công nghệ web Scraper, converter
 - Cảm thấy thoải mái để có thể hoàn thành dự án, không drop ngang

Có vài ứng cử viên sáng giá bao gồm: Go, Typescript, Python

### Dùng Web Scraper nào

Do mình muốn hoàn thành nhanh dự án, nên mình sẽ ưu tiên chọn những Web Scraper có tiếng, stable trong cộng đồng:
 - Selenium, Scrapy, Apify, Puppeteer, Playwright, Cypress

Mình cũng có test nhanh qua thì Notion yêu cầu phải có Javascript engine để render chứ không chỉ static HTML. Vì vậy với những công cụ không hỗ trợ Javascript engine để render page sẽ bị lược bỏ. Notion cũng chặn một vào scraper (có cách custom để vượt qua, nhưng mình lười quá) nên danh sách thu gọn lại chỉ còn hai cái tên: Selenium và Playwright

Mình đã có 5 năm sử dụng Selenium rồi, nên mình quyết định chọn Playwright. Try new something new in the end of the year!

Do sử dụng Playwright, nên khả năng cao mình sẽ chọn luôn Typescript cho dễ.

### Dùng converter nào

Mình có tìm thử qua những giải pháp thì python và typescript đều support tốt mảng này. Một trong những thư viện mình có cơ hội sử dụng là https://github.com/mixmark-io/turndown

Một ý tưởng khác khi chọn converter là có thể sử dụng LLM để convert HTML sang Markdown format. Vừa thời thượng, lại vừa bắt trend.

Mình có làm vài thử nghiệm và có kế quả như sau:
 - Turndown break format nhiều, nhưng content thì vẫn giữ được
 - LLM thì tốn tokens quá (tiền). HTML tags consumes nhiều tokens quá.

Dựa trên kết quả như vậy, mình quyết định chọn một hướng vừa dùng library, vừa dùng LLM như sau:
 - Sử dụng Turndown để convert HTML sang Markdown trước. Chấp nhận việc formats/styles bị broken.
 - Sau đó dùng LLM đọc Markdown và chỉnh lại styles, formats cho đẹp, phù hợp với Github Markdown, như vậy đỡ tốn tokens hơn.

LLM mình chọn luôn dùng OpenAI, model ChatGPT 5.1. Lý do vì mình lười chọn, nên chọn đứa to nhất thôi.

Sau khi thay đổi 1 tí thì ý tưởng sẽ trông như thế này

<img width="821" height="407" alt="Image" src="https://github.com/user-attachments/assets/cc32f251-a3a3-44f0-a433-04e7b1d035c3" />

## Hiện thực ý tưởng

Chỉ là code thôi, không có gì đặc biệt ở step này.

Mình nghĩ mọi người đọc tới đây đều có thể tự code được một cái ứng dụng như vầy. Công việc chỉ bao gồm:
 - Start project
 - Implement flow structure
 - Add library
 - Đọc library documents
 - Add vào code
 - Test thử

Nên mình xin phép không đi chi tiết ở bước này. Source code mình xin để ở đây nhé https://github.com/ledongthuc/advent-of-sharing-notion-2-github

## Các vấn đề phát sinh

### Notion elements are dynamic

Các block trong Notion elements đa phần dynamic và không được đặt tên class/id chỉnh chu lắm. Cách này mình chỉ còn cách đọc relative dựa trên những cái cố định thôi

<img width="1076" height="394" alt="Image" src="https://github.com/user-attachments/assets/0e1aadd0-e465-44f3-a2ce-b3dbbe3205c3" />

<img width="1190" height="470" alt="Image" src="https://github.com/user-attachments/assets/6ff044f0-93a7-4ca1-ab98-9268bfce42df" />

### LLM lèm bèm thông tin không liên quan

Vấn đề này cũng không có gì đặc biệt, chủ yếu là output của LLM thường sẽ có những thông tin mình không mong muốn.
Cách giải quyết có 2 cách:
 - Dùng structured Output response để lấy đúng phần mình muốn lấy, tránh những thông tin không liên quan
 - Dùng những keywords kiểu như: "No explains, no additional data, just content", nói chung là kêu LLM nói ít lại.

### Notion lazy-render page

Notion sẽ không render hết tất cả các block cùng 1 lúc, mà dựa trên viewport của browser để render blocks tương ứng.
Cách giải quyết là sau khi xử lý xong item cuối cùng thì nên scroll thử để check lại xem còn item nào khác không.

## Marketing nhẹ

Bài post này là 1 cái marketing nè, nếu bạn đọc tới đây và thấy công sức mình bỏ ra xứng đá cho cộng đồng, thì ngại gì không cho mình 1 star vào repo https://github.com/ledongthuc/advent-of-sharing-notion-2-github

## Tổng kết

Bài viết này của mình mục đích chính muốn chia sẻ mọi người quy trình khi mình bắt đầu làm một dự án nhỏ hoặc một tính năng sẽ như thế nào.

Tổng kết là mình đã tạo được Pull Request https://github.com/webuild-community/advent-of-sharing/pull/72 rồi, nhưng không biết có thế lực nào đó trong WeBuild không merge giúp mình không thôi. Bonus thêm tí update README cho đẹp.

Source code mình cũng để ở đây nhé: https://github.com/ledongthuc/advent-of-sharing-notion-2-github

Chúc mọi người một mùa sáng sinh tốt lành!!!