# 서비스 이메일 발송 기능

서비스에 이메일을 보내는 기능을 추가하려고 한다. Node.js 환경에서 Nodemailer를 사용해 구현하는 방향으로 정했다.

## 참조 지식

- `nodemailer-smtp.md`

## 참조 이유

서비스에서 이메일을 발송하려면 SMTP 정보를 바탕으로 transporter를 만들고 `sendMail`을 호출하는 Nodemailer의 기본 흐름이 필요하다.
