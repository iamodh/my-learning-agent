# Express 이미지 업로드 API

Express에서 이미지를 업로드 받는 API를 만들려는 것이 목표다. 파일은 `req.body`로 직접 받는 것이 아니라 `multipart/form-data` 요청을 처리할 수 있는 `multer` 미들웨어로 받아야 한다.

## 참조 지식

- `multer-multipart-upload.md`: 이미지 파일과 텍스트 필드가 섞인 multipart 요청을 파싱하고, 파일을 `req.file` 또는 `req.files`에 연결하기 위해 필요하다.
- `multer-storage-adapter.md`: 업로드된 파일을 디스크에 저장할지, 메모리에 둘지, 나중에 S3 같은 다른 저장소로 옮길지 결정하기 위해 필요하다.

## 구현 방향

기본 라우터는 `upload.single('image')` 형태로 구성한다. 여기서 `'image'`는 클라이언트가 보내는 form 필드 이름이다.

```js
const express = require('express');
const multer = require('multer');

const upload = multer({ dest: 'uploads/' });
const app = express();

app.post('/upload', upload.single('image'), (req, res) => {
  console.log(req.file);
  console.log(req.body);
  res.json({ ok: true });
});
```

이미지만 받기 위해 `fileFilter`로 `mimetype`이 `image/`로 시작하는지 확인한다. 너무 큰 파일을 막기 위해 `limits.fileSize`를 설정한다.

```js
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }
});
```

파일 크기 초과 시 multer는 `MulterError('LIMIT_FILE_SIZE')`를 던지므로 에러 핸들러에서 413 같은 응답으로 처리할 수 있다.
