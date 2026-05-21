# nodemailer

nodemailer는 Node 환경에서 이메일을 보낼 때 거의 표준으로 쓰이는 라이브러리다.

## 사용 흐름

SMTP 호스트, 포트, 계정 정보를 넣어 transporter를 만들고, `sendMail`을 호출하면 메일이 발송된다.

## 설치

`npm install nodemailer`로 설치하면 바로 쓸 수 있다.
