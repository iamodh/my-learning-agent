# 환경 변수 관리 (로컬 vs 배포)

## 핵심
- 로컬: 프로젝트 루트에 `.env.local` 파일을 만들고 `API_KEY=...` 형태로 작성.
- 배포(Vercel): 대시보드 Settings → Environment Variables에서 같은 키를 등록.
- 브라우저에 노출돼야 하는 값은 `NEXT_PUBLIC_` 접두사를 붙인다.

## 요점
| 환경 | 위치 |
|------|------|
| 로컬 | `.env.local` |
| Vercel | Settings > Environment Variables |

접두사 규칙(`NEXT_PUBLIC_`)만 기억하면 클라이언트/서버 노출 구분이 끝난다.
