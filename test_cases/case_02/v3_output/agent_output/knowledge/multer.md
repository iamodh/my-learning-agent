# multer

`multipart/form-data` 요청을 파싱해서 Express 친화적인 형태로 정리해주는 미들웨어다. 텍스트 필드는 `req.body`에 꽂아주고, 파일은 `req.file`(단일) 또는 `req.files`(다중)에 꽂아준다.

핵심 책임은 두 가지로 좁혀진다.

1. multipart 본문을 스트림으로 읽어 파트 단위로 분해하기.
2. 잘라낸 결과를 Express의 `req`에 꽂아 라우트 핸들러가 평범한 객체로 다룰 수 있게 하기.

실제 저장은 multer 본체가 아니라 스토리지 어댑터가 담당한다 (`knowledge/multer-storage.md`).

## 기본 사용 형태

라우트에 미들웨어로 끼워 넣는다. `upload.single('image')`의 `'image'`는 클라이언트가 보낸 form 필드 이름이다.

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

여러 파일을 받을 때는 `.array('images', 5)`나 `.fields([...])`를 쓴다.

`dest: 'uploads/'`는 내부적으로 `diskStorage`를 쓰는 단축 문법이다 — 저장 방식을 직접 제어하려면 스토리지 어댑터를 명시한다.

## fileFilter — 어떤 파일을 받을지 거르기

옵션 객체에 `fileFilter`를 넘기면 파일별로 받을지 결정할 수 있다. 콜백의 두 번째 인자가 `true`면 통과, `false`면 거부다.

```js
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) cb(null, true);
    else cb(new Error('이미지만 업로드 가능합니다'), false);
  }
});
```

여기서 `file.mimetype`은 클라이언트가 보낸 값이라 위조될 수 있다. 더 엄격하게 검증하려면 업로드 후 파일의 매직 넘버를 보는 라이브러리(예: `file-type`)로 한 번 더 검사한다.

## limits — 크기 등 한도 걸기

옵션 객체에 `limits`를 추가한다. `fileSize`는 바이트 단위다.

```js
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});
```

한도를 넘으면 `MulterError('LIMIT_FILE_SIZE')`가 던져진다. 에러 핸들러에서 잡아 413 같은 응답으로 돌려주면 된다.

중요한 동작 특성: 이 검사는 스트림을 읽는 도중에 일어난다. 전체 본문을 다 받아둔 뒤 거르는 게 아니라, 한도 초과 시점에 바로 끊어버린다.
