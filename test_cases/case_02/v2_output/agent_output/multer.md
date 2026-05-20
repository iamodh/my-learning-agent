# multer

multer는 Express에서 `multipart/form-data` 요청을 파싱해 파일을 `req.file`(또는 `req.files`)에, 텍스트 필드를 `req.body`에 꽂아주는 미들웨어다. Express 기본 파서(`express.json`, `express.urlencoded`)는 멀티파트를 처리하지 못하기 때문에 파일 업로드에는 별도 미들웨어가 필요하며, Express 생태계에서는 multer가 사실상 표준이다. (busboy, formidable 같은 대안이 있으나 이 학습에서는 multer로 진행하며 채택하지 않았다.)

## 왜 일반 파서로는 안 되는가

`multipart/form-data`는 텍스트와 바이너리가 boundary 문자열로 구분되어 한 요청에 섞여 들어오는 형식이다. JSON처럼 문자열을 단순 파싱하는 것이 아니라, 스트림을 읽으면서 각 파트의 헤더를 읽고 본문을 잘라내야 한다. multer는 이 파싱을 대신 해주고, 결과를 Express 친화적인 형태(`req.file`, `req.body`)로 정리해준다.

## 기본 사용법

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

`upload.single('image')`의 `'image'`는 클라이언트가 보낸 form 필드 이름이다. 여러 개를 받을 때는 `.array('images', 5)`나 `.fields([...])`를 쓴다.

## 스토리지 어댑터

파일을 어디에 저장할지는 multer 본체가 아니라 "스토리지" 어댑터가 결정한다. 기본 제공은 두 가지다.

- `diskStorage`: 디스크에 저장
- `memoryStorage`: 버퍼로 메모리에 보관

위 예시의 `dest` 옵션은 내부적으로 `diskStorage`를 쓰는 단축 문법이다.

스토리지는 `_handleFile`(파일을 받아서 어딘가에 쓰기)과 `_removeFile`(에러 시 정리) 두 메서드를 가진 객체다. multer가 멀티파트를 파싱하다 파일 파트를 만나면 그 스트림을 스토리지에 넘기고, 스토리지가 디스크/메모리/S3 등 어디에 쓸지를 결정한다. 파싱과 저장의 책임이 분리되어 있어 `multer-s3`, `multer-gridfs-storage` 같은 서드파티 스토리지로 갈아끼우면 저장 위치를 통째로 바꿀 수 있다.

### diskStorage로 파일명 직접 정하기

```js
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

const upload = multer({ storage });
```

원본 파일명을 그대로 쓰면 한글 깨짐이나 충돌이 생기기 쉬워서 보통 고유 이름을 만들어준다.

## fileFilter: 파일 종류 제한

mimetype을 보고 받을지 거를지 결정한다.

```js
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) cb(null, true);
    else cb(new Error('이미지만 업로드 가능합니다'), false);
  }
});
```

mimetype은 클라이언트가 보낸 값이라 위조될 수 있다. 엄격하게 하려면 업로드 후 파일의 매직 넘버(예: `file-type` 라이브러리)로 한 번 더 검증한다.

## limits: 크기 제한

`limits.fileSize`는 바이트 단위다.

```js
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});
```

초과 시 multer가 `MulterError('LIMIT_FILE_SIZE')`를 던지므로 에러 핸들러에서 잡아 413 같은 응답을 준다. 이 검사는 스트림을 읽는 도중 동작하므로, 전체를 다 받은 뒤에 거르는 게 아니라 한도 초과 시점에 끊어버린다.
