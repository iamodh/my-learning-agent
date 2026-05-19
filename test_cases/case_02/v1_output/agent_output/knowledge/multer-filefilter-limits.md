# multer 파일 검증: fileFilter 와 limits

## 핵심

- `fileFilter`: 파일의 `mimetype`을 보고 받을지 거를지 결정한다 (타입 제한).
- `limits.fileSize`: 허용 최대 크기를 바이트 단위로 지정한다 (크기 제한).
- 두 옵션을 함께 쓰면 타입 + 크기의 두 겹 방어선이 된다.

## fileFilter (타입 제한)

```js
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) cb(null, true);
    else cb(new Error('이미지만 업로드 가능합니다'), false);
  }
});
```

주의: `mimetype`은 클라이언트가 보낸 값이라 위조될 수 있다. 엄격하게 하려면 업로드 후 파일의 매직 넘버(`file-type` 같은 라이브러리)로 한 번 더 검증하는 게 좋다.

## limits (크기 제한)

```js
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB
});
```

- 초과 시 multer가 `MulterError('LIMIT_FILE_SIZE')`를 던지므로, 에러 핸들러에서 잡아 413 같은 응답을 준다.
- 이 검사는 스트림을 읽으면서 동작한다. 전체를 다 받은 뒤 거르는 게 아니라 한도 초과 시점에 연결을 끊는다.
