# 결과물: Express 이미지 업로드 API

## 만들려고 했던 것

Express로 이미지 파일을 업로드받는 API. 텍스트 필드는 `req.body`, 파일은 `req.file`로 받고, 저장 위치를 나중에 로컬 디스크 → S3로 바꿀 수 있는 구조. 이미지만 허용하고 크기 한도를 두어 서버를 보호한다.

## 최종 형태

```js
const express = require('express');
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, 'uploads/'),
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
  res.json({ ok: true, file: req.file });
});
```

## 사용된 지식과 이유

| 지식 문서 | 사용 이유 |
|-----------|-----------|
| [multipart/form-data 와 multer](../knowledge/multipart-and-multer.md) | 파일은 multipart로 와서 기본 파서로 못 받음 → multer로 파싱해 `req.file`에 꽂기 위해 필요했다. |
| [multer 스토리지 어댑터](../knowledge/multer-storage-adapter.md) | "나중에 디스크 → S3로 옮기고 싶다"는 요구 때문. 파싱/저장이 분리돼 있어 스토리지만 교체하면 라우터 코드를 안 건드려도 되므로 diskStorage를 명시적으로 구성했다. 파일명 충돌·한글 깨짐 방지를 위해 고유 파일명도 직접 지정했다. |
| [multer 파일 검증: fileFilter 와 limits](../knowledge/multer-filefilter-limits.md) | "이미지만 받고 싶다", "큰 파일에 서버가 죽을 것 같다"는 우려 때문. fileFilter로 MIME 타입을 거르고 limits로 크기를 막아 두 겹 방어선을 두었다. |

## 핵심 설계 결정

파싱(multer)과 저장(스토리지)의 책임 분리가 이 API 구조의 핵심이다. 라우터에선 `upload.single('image')`만 유지하고, 저장 위치 변경 시 스토리지 인스턴스만 교체한다. 거기에 fileFilter(타입) + limits(크기)의 두 겹 방어선으로 안정성을 확보한다.
