# multer 스토리지 어댑터

## 핵심

- 파일을 **어디에 저장할지**는 multer 본체가 아니라 별도의 "스토리지" 어댑터가 결정한다.
- 기본 제공: `diskStorage`(디스크에 저장), `memoryStorage`(버퍼로 메모리에 보관).
- 파싱(multer)과 저장(스토리지)의 책임이 분리돼 있어, 스토리지만 교체하면 저장 위치를 통째로 바꿀 수 있다 (예: `multer-s3`, `multer-gridfs-storage`).

## 동작 원리

스토리지는 두 메서드를 가진 객체다.

- `_handleFile`: 파일을 받아서 어딘가에 쓰기
- `_removeFile`: 에러 시 정리

multer는 멀티파트를 파싱하다가 파일 파트를 만나면 그 스트림을 스토리지에 넘긴다. 스토리지가 디스크/메모리/S3 중 어디에 쓸지 결정한다. 이 분리 덕분에 라우터의 `upload.single('image')`는 그대로 두고 스토리지 인스턴스만 교체하면 된다.

## diskStorage 로 파일명 직접 제어

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

원본 파일명을 그대로 쓰면 한글 깨짐이나 충돌이 생기기 쉬우므로 보통 고유 이름을 생성한다.
