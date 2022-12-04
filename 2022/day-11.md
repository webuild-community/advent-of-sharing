---
title: Make your site talk with Web Speech API
author: ZeroX-DG
date: 12-04-2022
---

[Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API) là các web API cho phép website tạo các chức năng liên quan tới giọng nói như text-to-speech (chị google đọc chữ) hay speech recognition (chị google phân tích xem bạn vừa nói cái gì).

Điểm mạnh của mấy API này là chúng khá đơn giản để dùng. Ví dụ để nói "hello world", thì các bạn chỉ cần dùng một dòng duy nhất:

```js
speechSynthesis.speak(new SpeechSynthesisUtterance("Hello world!"));
```

Tuy vậy, để nhận diện giọng nói thì configuration của Speech Recognition API sẽ hơi phức tạp hơn đôi chút:

```js
const recognition = new SpeechRecognition(); // tạo instance cho speech recognition API
recognition.lang = 'en-US'; // cài đặt ngôn ngữ để nhận diện. ví dụ en-US là tiếng anh, còn Tiếng Việt là 'vi-VN' nhưng không rõ nó có hoạt động không.

// Nhận event khi API nhận dạng được giọng nói.
recognition.onresult = function(event) {
    console.log(event);
}

// Bắt đầu nhận dạng
recognition.start();
```

Khi chạy đoạn code trên, browser sẽ mở một pop up yêu cầu cho phép truy cập vào microphone để bắt đầu nhận dạng. Tuy nhiên, việc nhận dạng sẽ ngay lập tức kết thúc sau khi người dùng ngưng nói. Để ngăn chặn việc này, các bạn cần config thêm một dòng:

```js
recognition.continuous = true;
```

Đặc biệt, mức độ chính xác của việc nhận diện sẽ gia tăng nếu bạn có thể giới hạn nó để chỉ nhận diện cho một số từ. Ví dụ chỉ quan tâm về từ chỉ màu sắc, bạn có thể tạo một gramma list thế này:

```js
const grammar = '#JSGF V1.0; grammar colors; public <color> = aqua | azure | beige | bisque | black | blue | brown | chocolate | coral | crimson | cyan | fuchsia | ghostwhite | gold | goldenrod | gray | green | indigo | ivory | khaki | lavender | lime | linen | magenta | maroon | moccasin | navy | olive | orange | orchid | peru | pink | plum | purple | red | salmon | sienna | silver | snow | tan | teal | thistle | tomato | turquoise | violet | white | yellow ;';
const speechRecognitionList = new SpeechGrammarList();
speechRecognitionList.addFromString(grammar, 1);
recognition.grammars = speechRecognitionList;
```

> Một điều cần lưu ý là grammar list được định nghĩa theo định dạng [JSGF](https://www.w3.org/TR/jsgf/) (nên phần đầu có cái `#JSGF V1.0`).

Chỉ với hai API đơn giản trên là đã đủ xài để bạn bắt đầu xây dựng một con trợ lý ảo ngon ăn. Nhưng đây là 2022 rồi, xài mấy API này thì các bạn có cần phải update privacy policy rồi spam chục cái email cho users không?

Câu trả lời là có. Vì mỗi browser khác nhau sẽ dùng "backend" khác nhau để nhận diện giọng nói. Ví dụ nếu chạy trên Safari, việc nhận diện sẽ được thực hiện bởi Siri (mình đoán thế), [còn trên Firefox](https://wiki.mozilla.org/Web_Speech_API_-_Speech_Recognition#Where_does_the_audio_go.3F) thì audio của bạn sẽ được gửi tới [Google Cloud Speech-to-Text](https://cloud.google.com/speech-to-text), và chrome thì chắc sẽ tuỳ thiết bị bạn đang sử dụng, ví dụ IOS thì Siri, còn Android thì `android.speech` (again mình đoán thế, vì mấy nguồn mình tìm được nó không rõ ràng nên chỉ có thể đoán thế).

Thế nên nếu users của bạn quan tâm về chuyện audio data của họ sẽ đi về đâu và có bị lưu trữ hay không thì mình nghĩ bạn nên viết một dòng cảnh báo trong privacy policy.

=== Skip khúc này cũng được ===

**Extra note 0:** Có một [trang demo](https://www.google.com/intl/en/chrome/demos/speech.html) bên google để các bạn thử khả năng nhận dạng giọng nói.

**Extra note 1:** Có rất nhiều library wrapper cho Web Speech API để abstract mấy phần configuration cho bạn. Mình thì chỉ mới xài [annyang.js](https://www.talater.com/annyang/) nên chỉ có thể recommend nó.
