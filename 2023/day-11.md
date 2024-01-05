---
title: An in-comprehensive guide to Local LLM
author: huytd
date: 2023-12-11
---

Như các bạn đã biết thì 2023 là năm của phong trào AI, hay nói chính xác hơn là Large Language Models (LLM).

Để làm việc với LLM, thì giải pháp phổ biến nhất là sử dụng các sản phẩm như ChatGPT, Poe, Phind, Perplexity, ChatUML... Hoặc nếu là developer thì có thể sử dụng API từ các công ty như OpenAI / Microsoft (GPT-3.5, GPT-4,...), Anthropic (Claude), Replicate,... Tất nhiên đây đều là các giải pháp tốn tiền.

Ngoài các giải pháp trên, thì nhờ vào sự mở đường của Facebook (Meta) với việc để lộ weight của model LLaMA, và việc release model Llama 2, cộng đồng công nghệ đã từng bước phát triển được những giải pháp giúp chúng ta có thể run LLM ngay trên các máy tính cá nhân (có hoặc không có GPU), chạy offline không cần kết nối internet, các giải pháp này gọi chung là Local LLM.

Trong bài viết này, mình sẽ không đi sâu vào giải thích LLM là gì và cơ chế hoạt động của nó như thế nào, mà chỉ giới thiệu qua một số giải pháp để sử dụng LLM ở ngay trên máy tính cá nhân, cũng như cách chọn lựa model để sử dụng một cách hiệu quả. Lưu ý là phạm vi bài viết này mình sẽ không đề cập tới vấn đề fine-tuning.

**Một số dòng LLM phổ biến**

- LLaMA: Là model bị leak của Facebook không lâu sau khi OpenAI release ChatGPT. Từ model này, các nhóm nghiên cứu trong cộng đồng đã phát triển thành các model khác như:
    - Alpaca: fine-tune từ model LLaMA 7B, tập trung vào instruction following
    - Vicuna: fine-tune từ nội dung chat với ChatGPT / GPT-4, có phiên bản based trên Llama 2 nữa.
    - WizardML: fine-tune để phản hồi với các instruction phức tạp
- Llama 2: Là model được Facebook release về sau, có performance và khả năng bám sát instruction tốt hơn LLaMA. 
    - Một số model được phát triển từ Llama 2 như Orca, Vicuna, CodeLlama được fine-tuned để generate hoặc trả lời các câu hỏi về code tốt hơn
- Mistral: là các model được phát triển bởi một công ty tên là... Mistral. Theo đánh giá thì Mistral &B có performance ngang ngửa Llama 2 13B.
    - Một số model được phát triển từ Mistral như Zephyr, Neural Chat
- Và rất nhiều model khác.


**Một số khái niệm cần biết**

- 0-shot, 1-shot, N-shot: Ám chỉ số lượng các ví dụ được cung cấp trong prompt để hướng dẫn LLM trả lời, trước khi đặt câu hỏi chính. 0-shot là hỏi luôn không ví dụ gì, 1-shot là có 1 mẫu ví dụ được cung cấp,... càng nhiều ví dụ mẫu thì model càng trả lời đúng với mong muốn của mình, tuy nhiên context window của các model thường có giới hạn. Như của GPT-3.5-Turbo là 4k, GPT-4 mặc định 8k,... các model chạy trên local thường có max context window là 4k.
- 1.3B, 7B, 13B, 70B,...: Có thể hiểu là độ lớn của model, con số này càng cao thì càng cần nhiều bộ nhớ để chạy, tất nhiên tỉ lệ thuận theo đó là độ chính xác, và khả năng follow the instruction của model. Nói một cách chung chung thì các model 7B cần ít nhất 8GB RAM, 13B thì 16GB RAM, 70B thì cần ít nhất 64GB RAM, và model càng lớn, tốc độ generate token càng chậm nếu benchmark trên cùng 1 cấu hình máy tính. Mình test thử thì các model 13B chạy ổn định nhất (không đủ RAM để test các model lớn hơn), còn các model 7B thường gặp chung 1 vấn đề là không hoàn toàn nghe theo instruction từ user.
- Khi đọc thông số của các model, thường chúng ta sẽ thấy các từ khoá như ARC, HellaSwag, MMLU,... đây là tên các bộ benchmark đùng dể đánh giá performance của các model. Dữ liệu dùng để thiết lập các bài đánh giá thường gặp gồm có:
    - **AI2 Reasoning Challenge** (ARC, 25-shot): Tập các câu hỏi khoa học trong trường phổ thông/tiểu học
    - **HellaSwag** (10-shot): bộ test về suy luận cho các kiến thức phổ thông, ai là con người thì thường trả lời đúng ~95%, còn với LLM thì vẫn còn là thách thức.
    - **MMLU** (5-shot): bộ test gồm 57 câu hỏi gồm toán tiểu học, lịch sử Mỹ, khoa học máy tính, luật pháp,..
    - **TruthfulQA** (0-shot): bộ test kiểm tra tỉ lệ đưa ra câu trả lời sai thường thấy trên mạng của model
    - **Winogrande** (5-shot): kiểm tra về các kiến thức suy luận thông thường
    - **GSM8k** (5-shot): tập các bài toán tiểu học để đo khả năng giải toán suy luận nhiều bước của model.
- Quantization là quá trình làm giảm kích thước của các model (ví dụ từ hàng chục GB xuống còn 4-9GB) mà vẫn đảm bảo không ảnh hưởng nhiều đến performance.
    - **GPTQ**: là phương pháp quantization ổn áp nhất, không ảnh hưởng nhiều lắm đến performance, chọn các model có kí hiệu GPTQ nếu bạn đang sử dụng card đồ hoạ của NVIDIA và toàn bộ model có thể fit trong VRAM của bạn.
    - **GGML**: là thư viện được tạo ra bởi tác giả của dự án `llama.cpp`, đây cũng là sự lựa chọn phổ biến cho những ai không có GPU. Có rất nhiều phương pháp quantization khác nhau cho GGML, thường được kí hiệu theo cú pháp `Q<NUMBER>_<LETTERS & NUMBERS>`, với `<NUMBER>` đại diện cho số bits như 4-bit, 5-bit. Càng nhiều bits thì file size càng lớn. Một số phiên bản nên chọn như:
        - Q4_K_M: 4-bit, medium, cân bằng giữa file size và chất lượng
        - Q5_K_M: 5-bit, dung lượng file lớn hơn, chất lượng cân bằng.
- `llama.cpp` là dự án giúp chạy/inference các LLM ở trên máy tính cá nhân, dùng CPU hoặc GPU


Again, nếu các bạn cũng giống như mình, chủ yếu xài Macbook, không có card đồ hoạ rời, và RAM từ 16GB trở xuống thì chỉ nên quanh quẩn trong các model 7B, 13B, chọn GGML Q5_K_M hoặc Q4_K_M.

Sau khi hiểu các thông số trên, các bạn có thể xem qua trang Open LLM Leaderboard ([https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard)) để chọn cho mình một model ưng ý.

**Compile và chạy LLM với llama.cpp**

Để chạy trên máy tính, bạn có thể tải về mã nguồn của `llama.cpp` và compile. Sau khi compile sẽ có rất nhiều file binary nhưng 2 file mà bạn cần quan tâm chính là `main` (dùng để chạy model trực tiếp, và nhận vào input từ bàn phím thông qua giao diện kiểu REPL) và `server` (dùng để tạo một API server để có thể tương tác programmatically). Khi chạy, thì bạn cần chỉ định file model tải về (thường là từ HuggingFace).

Mình không nói sâu vào cách này, các bạn có thể tự tìm hiểu thêm tại github repo: [https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)

**Compile và chạy LLM với Ollama**

Ollama là một chương trình giúp quản lý model và chạy `llama.cpp` một cách tiện lợi, bạn không cần phải quan tâm nhiều về việc đi download model ở đâu các kiểu.

Các bạn có thể tải về tại github repo [https://github.com/jmorganca/ollama](https://github.com/jmorganca/ollama) hoặc trang chủ của dự án ([https://ollama.ai](https://ollama.ai)).

Sau khi cài đặt, thì chúng ta có thể sử dụng được lệnh `ollama` trên terminal. Cách sử dụng khá đơn giản:

**Tải model về máy và chạy**

Có thể tham khảo danh sách các model được Ollama đóng gói sẵn tại [https://ollama.ai/library](https://ollama.ai/library), sau khi đã chọn được model ưng ý thì tiến hành tải về và chạy với lệnh:

```
ollama run <model-name>
```

Ví dụ ở đây mình chạy model CodeLlama phiên bản 13B:

```
ollama run codellama:13b
```

Sau khi chạy lệnh này thì Ollama sẽ đi vào chế độ REPL để bạn prompt trực tiếp với CodeLlama. Các bạn có thể thoát khỏi phiên REPL đó nếu thích, vì Ollama vẫn chạy ngầm và serve một API server ở cổng `:11434`

Các bạn có thể tham khảo API document của Ollama tại đây [https://github.com/jmorganca/ollama/blob/main/docs/api.md](https://github.com/jmorganca/ollama/blob/main/docs/api.md)

Ngoài ra, bạn có thể cài thêm các ứng dụng khác để chat với LLM thông qua Ollama API, ví dụ `oterm` ([https://github.com/ggozad/oterm) ](https://github.com/ggozad/oterm)hoặc nếu xài Raycast thì có addons Ollama cũng khá tiện.

**Sử dụng Ollama làm OpenAI proxy**

Một chức năng cũng khá hữu ích đó là: Sử dụng Ollama để thay thế cho OpenAI API. Nếu bạn nào cần sử dụng OpenAI API trong quá trình phát triển, mà sợ tốn tiền thì có thể thử.

Lưu ý là API mặc định của Ollama không tương thích với API interface của OpenAI, nên chúng ta cần cài thêm một ứng dụng khác là **LiteLLM** để làm proxy:

```
sudo pip3 install litellm
```

Sau đó thì run proxy server như này:

```
litellm --model ollama/codellama:13b --api_base http://localhost:11434
```

LiteLLM sẽ serve một API endpoint tại cổng `:8000`, khi xài các OpenAI API SDK, bạn có thể set giá trị `baseUrl` thành `http://localhost:8000/`

Tất nhiên khi run các model nhỏ ở máy local thì tiện, nhưng đánh đổi là performance không cao, độ chính xác không thể bằng với các model thương mại, ví dụ các model 13B thì mức độ "nghe lời" – follow instruction – tốt hơn, đưa ra câu trả lời chính xác hơn nhưng chậm, các model 7B thì tốc độ trả lời rất nhanh, nhưng follow instruction kém. Tuy nhiên, để sử dụng cho các công việc thường ngày như dịch thuật, sửa grammar, hỏi đáp dựa trên tài liệu sẵn có, summary nội dung trang web hay generate commit message, hỏi đáp code nhanh thì đây là một sự lựa chọn rất tốt.
