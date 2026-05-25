# Express 이미지 업로드 학습 기록

Express에서 multer를 사용해 이미지 업로드 API를 만들기 위해 학습한 내용과 그 결과물을 정리한다.

## 문서 배치

```
agent_output/
├── README.md
├── multer.md                     # 지식 문서: multer와 그 구성 요소
└── express-image-upload-api.md   # 결과 문서: Express 이미지 업로드 API
```

## 인과 관계

- `multer.md` — multipart/form-data 파싱, 스토리지 어댑터, fileFilter, limits 등 multer 사용에 필요한 개념을 한 문서에 묶었다.
- `express-image-upload-api.md` — 위 지식을 사용해 Express 라우터에 붙인 업로드 API 구현.
