# Express 이미지 업로드 API

Express로 이미지를 받는 업로드 엔드포인트. multer로 multipart 요청을 파싱하고, `diskStorage`로 `uploads/` 폴더에 고유 파일명으로 저장한다. `fileFilter`로 이미지 MIME만 받고, `limits`로 파일 크기를 5MB로 제한한다.

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

const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) cb(null, true);
    else cb(new Error('이미지만 업로드 가능합니다'), false);
  },
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});

const app = express();

app.post('/upload', upload.single('image'), (req, res) => {
  console.log(req.file);
  console.log(req.body);
  res.json({ ok: true });
});
```

라우터는 `upload.single('image')` 미들웨어 하나로 multipart 파싱 → 검증 → 저장이 묶여서 처리되고, 핸들러에서는 `req.file`만 보면 된다.

## 참조 매핑

- [multer.md](./multer.md) — multipart 요청을 파싱해 파일을 `req.file`에 꽂아 라우터에서 다루기 위해 사용. 같은 문서의 `diskStorage`로 파일명/저장 위치를 직접 정하고, `fileFilter`로 이미지 MIME만 받고, `limits`로 5MB 상한을 두는 두 겹의 방어선을 구성한다. 나중에 저장 위치를 로컬에서 S3로 바꿀 때 스토리지 인스턴스만 교체하면 되도록 파싱과 저장의 분리 구조를 그대로 활용한다.
