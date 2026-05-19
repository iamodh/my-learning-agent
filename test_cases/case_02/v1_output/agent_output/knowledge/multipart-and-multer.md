# multipart/form-data 와 multer

## 핵심

- 파일 업로드는 `multipart/form-data` 형식으로 오며, Express 기본 파서(`express.json`, `express.urlencoded`)로는 처리되지 않는다. 그래서 `req.body`로 파일이 잡히지 않는다.
- `multer` 미들웨어가 multipart 요청을 파싱해서 텍스트 필드는 `req.body`, 파일은 `req.file`(또는 `req.files`)에 꽂아준다.
- multer의 역할은 두 가지: **파싱** + **Express 친화적 형태로 정리**.

## 왜 기본 미들웨어로는 안 되나

`multipart/form-data`는 텍스트와 바이너리가 `boundary` 문자열로 구분되어 한 요청에 섞여 들어온다. JSON처럼 문자열을 통째로 파싱하는 게 아니라, 스트림을 읽으면서 각 파트의 헤더를 읽고 본문을 잘라내는 작업이 필요하다. multer가 이 작업을 대신 처리한다. (busboy, formidable도 있으나 Express 생태계에서는 multer가 사실상 표준)

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

- `upload.single('image')`의 `'image'`는 클라이언트가 보낸 form 필드 이름이다.
- 여러 파일이면 `.array('images', 5)` 또는 `.fields([...])`를 쓴다.
- `dest` 옵션은 내부적으로 diskStorage를 쓰는 단축 문법이다.
