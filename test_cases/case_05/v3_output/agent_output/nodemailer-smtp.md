# Nodemailer SMTP 메일 발송

Nodemailer는 Node.js에서 이메일을 보내기 위해 사용할 수 있는 라이브러리다. 대화에서는 서비스에 이메일 발송 기능을 붙일 때 Nodemailer를 사용하기로 했다.

## 기본 사용 흐름

SMTP 정보를 넣어 transporter를 만들고, `sendMail`을 호출해 이메일을 보낸다.

필요한 SMTP 정보는 대화에서 다음 수준으로 언급됐다.

- SMTP 호스트
- 포트
- 계정 정보

## 설치

```bash
npm install nodemailer
```
