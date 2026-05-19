# 문서 배치 구조

이 대화에서는 "서비스에 이메일 발송 기능을 추가"하는 것이 목표였고,
이를 위해 "nodemailer로 이메일 보내기" 지식을 학습했다.

인과 관계: `knowledge`(학습한 개념) → `result`(그 개념으로 만들려던 것)

```
agent_output/
├── README.md
├── knowledge/
│   └── nodemailer-기초.md            # 학습한 지식: nodemailer 사용법
└── result/
    └── 서비스-이메일-발송-기능.md     # 만들려던 것 (knowledge 참조)
```

## 문서 목록

- [knowledge/nodemailer-기초.md](knowledge/nodemailer-기초.md): Node.js에서
  nodemailer로 이메일을 보내는 방법 (설치, transporter, sendMail)
- [result/서비스-이메일-발송-기능.md](result/서비스-이메일-발송-기능.md):
  서비스에 이메일 발송 기능을 붙이는 작업과 위 지식이 사용된 이유

## 참고

대화에는 등산, 커스텀 키보드, 카페 등 잡담이 섞여 있었으나 학습/제작과
무관하여 문서화 대상에서 제외했다.
