# 학습 정리: Express 이미지 업로드 API

이 폴더는 Express에서 이미지 업로드 API를 만들기 위해 학습한 내용을 정리한 것이다.

## 배치 구조

```
.
├── README.md
├── image-upload-api.md          # 결과 문서: 만든 API와 사용한 지식
└── knowledge/
    ├── multipart-form-data.md   # 파일 업로드 요청 형식
    ├── multer.md                # multipart 파서 미들웨어 (옵션 포함)
    └── multer-storage.md        # multer의 스토리지 어댑터 패턴
```

## 인과 관계

- `image-upload-api.md`가 만들고자 한 결과물이다.
- `knowledge/`는 그 결과물을 만들기 위해 학습한 개념들이다.
  - `multipart-form-data`는 왜 기본 파서로 파일을 못 받는지의 이유.
  - `multer`는 그 형식을 파싱해 Express에 꽂아주는 도구.
  - `multer-storage`는 파싱과 저장이 분리된 구조 — 저장 위치를 갈아끼울 수 있는 축.
