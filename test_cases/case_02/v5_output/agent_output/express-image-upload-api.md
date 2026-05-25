# Express 이미지 업로드 API

Express 라우터에 multer 미들웨어를 붙여 이미지 파일 업로드를 받는 `POST /upload` API. 파싱은 multer가 담당하고 저장 위치는 `diskStorage`로 결정하며, MIME 필터(`fileFilter`)와 크기 제한(`limits`) 두 겹의 방어선을 둔다.

## 구성

- 파싱: multer (`multipart/form-data` → `req.file` / `req.body`)
- 저장: `multer.diskStorage` — `uploads/` 폴더에 시간 기반 고유 파일명으로 저장
- 필터: `fileFilter` — `image/*` MIME만 허용
- 보호: `limits.fileSize` — 5MB 초과 시 `MulterError('LIMIT_FILE_SIZE')`

파싱(multer)과 저장(스토리지)이 분리돼 있어, 나중에 로컬 디스크에서 S3로 옮기고 싶을 때 스토리지 인스턴스만 `multer-s3` 등으로 교체하면 라우터 코드는 그대로 둘 수 있다.

## 구현

```js
const express = require('express');
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const unique = Date.now() + '-' + Math.round(Math.random() * 1e9);
    cb(null, unique + path.extname(file.originalname));
  }
});

const fileFilter = (req, file, cb) => {
  if (file.mimetype.startsWith('image/')) cb(null, true);
  else cb(new Error('이미지만 업로드 가능합니다'), false);
};

const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});

const app = express();

app.post('/upload', upload.single('image'), (req, res) => {
  console.log(req.file);
  console.log(req.body);
  res.json({ ok: true });
});
```

## 참조 매핑

- [multer](./multer.md) — 요청 파싱, 스토리지 어댑터를 통한 저장, `fileFilter`로 MIME 거르기, `limits`로 크기 제한과 `MulterError` 발생까지 이 API에서 쓰는 multer 동작 전반.

## 참조 이유

- **multer**: `multipart/form-data` 요청을 파싱해 `req.file`로 꽂아주는 핵심 미들웨어이고, 이 API의 파싱·저장·필터·크기 제한이 모두 multer의 옵션과 스토리지 어댑터 구조 위에서 구성된다.
