# Multer 스토리지 어댑터

multer에서 파일을 실제로 어디에 저장할지는 스토리지 어댑터가 결정한다. `multer`는 multipart 요청을 파싱하고, 파일 파트를 만나면 파일 스트림을 스토리지에 넘긴다.

## 기본 저장 방식

`multer({ dest: 'uploads/' })`처럼 `dest` 옵션만 주면 디스크의 `uploads/` 폴더에 임시 파일명으로 저장된다. 이 방식은 내부적으로 `diskStorage`를 사용하는 단축 문법이다.

기본 제공 스토리지는 두 가지다.

- `diskStorage`: 디스크에 저장한다.
- `memoryStorage`: 파일을 버퍼로 메모리에 보관한다.

## 스토리지 어댑터의 역할

스토리지는 `_handleFile`과 `_removeFile` 메서드를 가진 객체다.

- `_handleFile`: 파일을 받아 어딘가에 쓴다.
- `_removeFile`: 에러가 났을 때 정리한다.

이 구조 덕분에 파싱 책임과 저장 책임이 분리된다. 저장 위치를 로컬 디스크에서 S3로 옮기고 싶다면 라우터의 `upload.single('image')` 구조는 유지하고 스토리지 인스턴스만 `multer-s3` 같은 것으로 바꿀 수 있다.

## 디스크 저장 파일명 지정 예시

`diskStorage`를 직접 만들면 저장 폴더와 파일명을 함수로 제어할 수 있다.

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

원본 파일명을 그대로 쓰면 한글 깨짐이나 충돌이 생기기 쉬우므로 고유 이름을 만들어 사용할 수 있다.
