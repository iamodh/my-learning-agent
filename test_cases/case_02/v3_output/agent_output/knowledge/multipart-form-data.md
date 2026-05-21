# multipart/form-data

파일 업로드 같이 텍스트와 바이너리를 한 HTTP 요청에 함께 실어 보낼 때 쓰는 요청 형식이다. 여러 "파트"가 boundary 문자열로 구분돼 한 본문에 섞여 들어온다.

이 형식은 JSON처럼 문자열을 통째로 파싱하는 게 아니라, 스트림을 읽으면서 각 파트의 헤더를 읽고 본문을 잘라내는 별도 처리가 필요하다. 그래서 Express 기본 파서인 `express.json`, `express.urlencoded`로는 파일이 잡히지 않는다 — `req.body`에 아무것도 안 들어온다.

파일을 받으려면 multipart를 이해하는 별도 파서가 필요하다. Node/Express 생태계에는 `busboy`, `formidable`, `multer` 등이 있고, Express에서는 `multer`가 사실상 표준으로 쓰인다.
