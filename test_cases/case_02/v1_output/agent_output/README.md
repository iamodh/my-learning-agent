# 문서 배치 구조

이 저장소는 "Express 이미지 업로드 API"를 만들기 위해 학습한 지식과, 그 지식으로 만든 결과물을 인과 관계에 따라 정리한 것이다.

## 인과 관계

`knowledge/`의 개념들을 학습 → `result/`의 API를 제작.
결과 문서는 어떤 지식을 왜 사용했는지 knowledge 문서를 참조한다.

## 폴더 구조

```
.
├── README.md
├── knowledge/                       # 학습한 지식 (개념 단위)
│   ├── multipart-and-multer.md      # multipart 형식과 multer의 파싱 역할
│   ├── multer-storage-adapter.md    # 저장 위치를 결정하는 스토리지 어댑터
│   └── multer-filefilter-limits.md  # 타입/크기 검증 (fileFilter, limits)
└── result/                          # 만들려고 했던 결과물
    └── express-image-upload-api.md  # Express 이미지 업로드 API + 사용 지식 참조
```

## 문서 목록

### knowledge/
- **multipart-and-multer.md** — 파일이 multipart로 오는 이유와 multer가 파싱해 `req.file`에 꽂는 원리
- **multer-storage-adapter.md** — 파싱/저장 책임 분리, diskStorage·memoryStorage, 스토리지 교체 가능성
- **multer-filefilter-limits.md** — MIME 타입 제한과 파일 크기 제한으로 만드는 두 겹 방어선

### result/
- **express-image-upload-api.md** — 최종 API 코드, 사용된 지식과 그 이유
