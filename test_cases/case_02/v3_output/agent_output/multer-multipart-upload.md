# Multer와 multipart/form-data

`multer`는 Express에서 `multipart/form-data` 요청을 파싱해 파일과 텍스트 필드를 Express에서 쓰기 쉬운 형태로 정리해 주는 미들웨어다. Express 기본 파서인 `express.json`이나 `express.urlencoded`는 이미지나 파일 업로드 요청을 처리하지 못한다.

## multipart/form-data가 별도 처리를 요구하는 이유

`multipart/form-data`는 텍스트와 바이너리가 boundary 문자열로 구분되어 한 요청 안에 섞여 들어오는 형식이다. JSON처럼 단순 문자열을 파싱하는 것이 아니라, 스트림을 읽으면서 각 파트의 헤더를 확인하고 본문을 잘라내야 한다.

`multer`는 이 파싱 작업을 수행한 뒤 텍스트 필드는 `req.body`에, 파일은 `req.file` 또는 `req.files`에 넣어준다.

## 기본 사용 예시

```js
const express = require('express');
const multer = require('multer');

const upload = multer({ dest: 'uploads/' });
const app = express();

app.post('/upload', upload.single('image'), (req, res) => {
  console.log(req.file);   // 업로드된 파일 정보
  console.log(req.body);   // 같이 온 텍스트 필드
  res.json({ ok: true });
});
```

`upload.single('image')`의 `'image'`는 클라이언트가 보낸 form 필드 이름이다. 여러 파일을 받을 때는 `.array('images', 5)`나 `.fields([...])`를 사용할 수 있다.

## 파일 제한

이미지만 받으려면 `fileFilter`를 사용해 파일의 `mimetype`을 확인한다.

```js
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) cb(null, true);
    else cb(new Error('이미지만 업로드 가능합니다'), false);
  }
});
```

다만 `mimetype`은 클라이언트가 보낸 값이라 위조될 수 있다. 더 엄격하게 확인하려면 업로드 후 파일의 매직 넘버를 한 번 더 검증할 수 있다.

크기 제한은 `limits.fileSize`로 설정한다. 값은 바이트 단위다.

```js
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});
```

이 제한은 스트림을 읽으면서 동작하므로, 전체 파일을 다 받은 뒤 거르는 것이 아니라 한도 초과 시점에 끊는다.
