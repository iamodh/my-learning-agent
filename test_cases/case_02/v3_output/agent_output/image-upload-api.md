# Express 이미지 업로드 API

## 무엇을 만들었는가

Express에서 클라이언트가 올리는 이미지 파일을 받는 `POST /upload` 엔드포인트.

- 요청은 `multipart/form-data`로 받는다.
- 파싱은 `multer` 미들웨어가 담당하고, 라우트 핸들러에서는 `req.file`로 업로드된 파일 정보를 읽는다.
- 파일은 `diskStorage`로 `uploads/` 폴더에 저장하며, 파일명은 시간 + 랜덤값 + 원본 확장자로 직접 만들어 충돌·한글 깨짐을 피한다.
- 이미지가 아닌 파일은 `fileFilter`로 거부한다 (MIME `image/`로 시작 검사).
- 파일 크기는 `limits.fileSize`로 5MB로 제한한다.

라우터 코드는 `upload.single('image')` 한 줄로 끝나고, 저장 방식 변경(예: 나중에 S3로 이전)은 스토리지 인스턴스만 교체하는 방식으로 흡수한다.

## 사용한 지식과 참조 이유

| 참조 | 참조 이유 |
|------|-----------|
| `knowledge/multipart-form-data.md` | 클라이언트가 보내는 형식이 multipart라서, `express.json`/`express.urlencoded`로는 파일을 받을 수 없다는 점을 이해해야 별도 파서를 도입하는 결정이 정당화된다. |
| `knowledge/multer.md` | multipart 파싱과 `req.file` 주입을 담당하는 미들웨어. `fileFilter`로 MIME 제한, `limits`로 크기 제한을 거는 두 겹의 방어선을 이 도구의 옵션으로 구성했다. |
| `knowledge/multer-storage.md` | 저장 위치를 라우터와 분리하기 위해 스토리지 어댑터를 직접 구성했다. 파일명 규칙을 손에 쥐고, 나중에 S3 같은 외부 스토리지로 옮길 때 라우터를 안 건드리는 구조를 얻는다. |
