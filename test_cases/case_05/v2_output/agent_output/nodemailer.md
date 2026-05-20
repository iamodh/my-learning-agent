# nodemailer

Node.js 환경에서 이메일 발송에 사용하는 라이브러리. Node에서 거의 표준으로 쓰인다.

## 설치

```
npm install nodemailer
```

설치 후 바로 사용할 수 있다.

## 사용 흐름

1. SMTP 정보(호스트, 포트, 계정 정보)를 넣어 transporter를 만든다.
2. transporter의 `sendMail`을 호출해 이메일을 보낸다.
