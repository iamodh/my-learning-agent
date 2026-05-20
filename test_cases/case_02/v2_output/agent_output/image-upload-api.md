# Express 이미지 업로드 API

Express로 이미지를 업로드 받는 API를 만든다. 클라이언트가 `multipart/form-data`로 보낸 이미지 파일을 받아 서버 디스크에 저장하고, 이미지가 아닌 파일이나 너무 큰 파일은 거른다.

## 윤곽

- 엔드포인트: `POST /upload`, 필드명 `image`로 단일 파일 수신
- 저장: 로컬 디스크 `uploads/` 폴더, 충돌·한글 깨짐 방지를 위해 파일명은 `Date.now()-random + 확장자`로 재생성
- 허용 종류: `image/*` MIME만 통과
- 크기 한도: 5MB, 초과 시 `MulterError('LIMIT_FILE_SIZE')`로 차단
- 구조: 라우터는 `upload.single('image')`만 유지하고 저장 책임은 스토리지 인스턴스로 분리 — 추후 S3 등으로 옮길 때 라우터를 건드리지 않는다.

## 사용된 지식

| 지식 | 참조 이유 |
|------|-----------|
| [multer](./multer.md) | Express 기본 파서는 `multipart/form-data`를 처리하지 못하므로, 멀티파트 파싱과 `req.file` 노출을 위해 채택. `diskStorage`로 저장 위치와 파일명을 제어하고, `fileFilter`로 MIME(`image/*`)을 거르고, `limits.fileSize`로 5MB 한도를 강제한다. 파싱과 저장이 분리된 구조 덕분에 스토리지만 교체하면 저장 위치를 바꿀 수 있다는 점이 설계상의 핵심이다. |
