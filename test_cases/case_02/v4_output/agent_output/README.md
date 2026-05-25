# Express 이미지 업로드 API 학습 정리

대화에서 Express로 이미지 업로드 API를 만들기 위해 multer를 학습했고, 그 지식을 적용한 결과를 정리했다.

## 구조

```
agent_output/
├── README.md
├── multer.md            # 지식: multer와 multipart 파싱, 스토리지 어댑터, fileFilter, limits
└── image-upload-api.md  # 결과: Express 이미지 업로드 엔드포인트
```

## 문서

- [multer.md](./multer.md) — Express에서 multipart/form-data 요청을 파싱해 파일을 받기 위한 미들웨어. 파싱과 저장의 분리, 스토리지 어댑터, fileFilter, limits 옵션을 정리.
- [image-upload-api.md](./image-upload-api.md) — 위 지식을 사용해 만든 Express 이미지 업로드 엔드포인트의 최종 구현.
