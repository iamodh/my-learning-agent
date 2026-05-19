# nodemailer로 이메일 보내기

## 핵심 요약

- Node.js에서 이메일을 보낼 때 사실상 표준으로 쓰이는 라이브러리가 `nodemailer`다.
- 설치: `npm install nodemailer`
- SMTP 호스트, 포트, 계정 정보로 transporter를 만들고 `sendMail`을 호출하면 메일이 발송된다.

## 사용 절차

1. 패키지 설치

```bash
npm install nodemailer
```

2. SMTP 정보로 transporter 생성

```js
const nodemailer = require("nodemailer");

const transporter = nodemailer.createTransport({
  host: "smtp.example.com", // SMTP 호스트
  port: 587,                 // SMTP 포트
  auth: {
    user: "account@example.com", // 계정
    pass: "app-password",        // 비밀번호
  },
});
```

3. 메일 발송

```js
await transporter.sendMail({
  from: "account@example.com",
  to: "receiver@example.com",
  subject: "안녕하세요",
  text: "본문 내용",
});
```

## 정리

필요한 입력은 SMTP 호스트 / 포트 / 계정 정보 세 가지뿐이다. 이 값으로
transporter를 한 번 만들어 두고, 보낼 때마다 `sendMail`을 호출하는 흐름이다.
