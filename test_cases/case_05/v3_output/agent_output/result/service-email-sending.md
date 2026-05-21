# 서비스 이메일 발송 기능

기존 서비스에 이메일을 보내는 기능을 추가한다. Node 환경이므로 nodemailer로 SMTP 기반 발송을 구성한다.

## 참조 지식

- [nodemailer](../knowledge/nodemailer.md)

## 참조 이유

- **nodemailer**: Node에서 이메일 발송을 붙일 때 거의 표준으로 통용되는 라이브러리라 채택. SMTP 정보로 transporter를 만들고 `sendMail`만 호출하면 되는 단순한 사용 흐름이 이 기능에 필요한 수준과 맞다.
