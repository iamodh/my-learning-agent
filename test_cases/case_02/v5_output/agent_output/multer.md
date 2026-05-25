# multer

multer는 Express에서 `multipart/form-data` 형식의 요청을 파싱해 텍스트 필드는 `req.body`에, 파일은 `req.file`(또는 `req.files`)에 꽂아주는 미들웨어다. Express 기본 파서(`express.json`, `express.urlencoded`)는 multipart를 처리하지 못하기 때문에 파일 업로드를 받으려면 multer 같은 별도 미들웨어가 필요하다. busboy, formidable 같은 대안도 있지만 Express 생태계에서는 multer가 사실상 표준이다.

## 왜 기본 파서로는 안 되는가 (multipart/form-data 배경)

`multipart/form-data`는 텍스트와 바이너리가 boundary 문자열로 구분되어 한 요청에 섞여 들어오는 형식이다. JSON처럼 단순히 문자열을 파싱하는 게 아니라, 스트림을 읽으면서 각 파트의 헤더를 읽고 본문을 잘라내는 작업이 필요하다. multer는 이 파싱을 대신 해주고, 그 결과를 Express 친화적인 형태(`req.body` / `req.file`)로 정리해주는 두 가지 일을 한다.

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

`upload.single('image')`에서 `'image'`는 클라이언트가 보낸 form 필드 이름이다. 여러 개를 받을 때는 `.array('images', 5)`나 `.fields([...])`를 쓴다.

## 스토리지 어댑터

파일을 실제로 어디에 저장할지는 multer 본체가 아니라 "스토리지"라는 별도 어댑터가 결정한다. 기본 제공되는 것은 두 가지다.

- `diskStorage`: 디스크에 저장
- `memoryStorage`: 버퍼로 메모리에 보관

위 기본 사용법의 `dest` 옵션은 내부적으로 `diskStorage`를 쓰는 단축 문법이다.

스토리지는 `_handleFile`(파일을 받아서 어딘가에 쓰기)과 `_removeFile`(에러 시 정리) 두 메서드를 가진 객체다. multer는 멀티파트를 파싱하다가 파일 파트를 만나면 그 스트림을 스토리지에 넘기고, 스토리지가 "디스크에 쓸지, 메모리에 담을지, S3에 올릴지"를 결정한다. 이렇게 파싱과 저장의 책임이 분리되어 있어서, `multer-s3`나 `multer-gridfs-storage` 같은 서드파티 스토리지로 갈아끼우기만 하면 라우터 코드를 건드리지 않고 저장 위치를 통째로 바꿀 수 있다.

### diskStorage 직접 구성하기

`diskStorage`를 직접 만들면 폴더와 파일명을 함수로 제어할 수 있다.

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

원본 파일명을 그대로 쓰면 한글 깨짐이나 충돌이 생기기 쉬워서 보통 이렇게 고유 이름을 만들어 준다.

## fileFilter — MIME 타입 필터링

`fileFilter` 옵션에 함수를 넘기면 파일의 `mimetype`을 보고 받을지 거를지 결정할 수 있다.

```js
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) cb(null, true);
    else cb(new Error('이미지만 업로드 가능합니다'), false);
  }
});
```

다만 `mimetype`은 클라이언트가 보낸 값이라 위조될 수 있다. 엄격하게 하려면 업로드 후 파일의 매직 넘버(`file-type` 같은 라이브러리)로 한 번 더 검증하는 것이 좋다.

## limits — 크기 제한과 에러 처리

같은 옵션 객체에 `limits`를 추가해 파일 크기를 제한할 수 있다. `fileSize`는 바이트 단위다.

```js
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});
```

초과 시 multer가 `MulterError('LIMIT_FILE_SIZE')`를 던지므로 에러 핸들러에서 잡아 413 같은 응답을 줄 수 있다. 이 검사는 스트림을 읽으면서 동작하기 때문에 전체를 다 받은 후가 아니라 한도 초과 시점에 즉시 끊는다.
