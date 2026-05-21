# multer 스토리지 어댑터

multer는 multipart를 파싱하는 일과 파일을 실제로 어디에 저장하는 일을 분리한다. 후자를 담당하는 것이 "스토리지" 어댑터다. 파싱과 저장의 책임을 분리해, 라우터 코드를 건드리지 않고 저장 위치만 갈아끼울 수 있게 한 구조다.

스토리지는 두 메서드를 가진 객체다.

- `_handleFile`: 파일을 받아 어딘가에 쓴다.
- `_removeFile`: 에러 시 정리한다.

multer는 multipart를 파싱하다가 파일 파트를 만나면 그 스트림을 스토리지에 넘기고, 스토리지가 "디스크에 쓸지, 메모리에 담을지, S3에 올릴지"를 결정한다. 덕분에 `multer-s3`, `multer-gridfs-storage` 같은 서드파티 스토리지로 교체하면 라우트 코드는 그대로 두고 저장 위치만 바꿀 수 있다.

## 기본 제공 스토리지

- `diskStorage`: 디스크에 파일로 저장한다. `multer({ dest: '...' })`는 이것을 쓰는 단축 문법이다.
- `memoryStorage`: 버퍼로 메모리에 보관한다.

## diskStorage로 폴더·파일명 직접 정하기

`destination`과 `filename`을 함수로 넘기면 저장 폴더와 파일명을 손에 쥘 수 있다.

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

원본 파일명을 그대로 쓰면 한글 깨짐이나 충돌이 생기기 쉬워서 보통 시간 + 랜덤값 + 원본 확장자 같은 고유 이름으로 만들어 준다.
