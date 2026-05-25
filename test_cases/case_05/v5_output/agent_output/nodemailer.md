# nodemailer

Node.js에서 이메일을 보낼 때 사용하는 라이브러리다. 노드 생태계에서는 이메일 전송의 사실상 표준으로 쓰인다.

## 기본 사용 흐름

SMTP 정보(호스트, 포트, 계정 정보)를 넣어 transporter를 만들고, 그 transporter의 `sendMail`을 호출하면 메일이 전송된다.

## 설치

```
npm install nodemailer
```
