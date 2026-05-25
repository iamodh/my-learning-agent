# Vercel 배포와 환경 변수

Vercel은 Next.js를 만든 회사의 호스팅 서비스로, Next.js 기능들이 별도 설정 없이 거의 다 그대로 동작한다. GitHub 저장소를 연동해두면 푸시할 때마다 자동 배포된다. 개인 블로그 규모면 무료 플랜으로 충분하다.

## 환경 변수

로컬과 배포 환경에서 같은 키를 두고 값만 다르게 관리한다.

- 로컬: 프로젝트 루트에 `.env.local` 파일을 만들고 `API_KEY=...` 같은 식으로 적는다.
- 배포: Vercel 대시보드의 Settings → Environment Variables에서 같은 키를 등록한다.

브라우저에 노출돼야 하는 값은 키 이름에 `NEXT_PUBLIC_` 접두사를 붙여야 한다.

> 초기에는 Netlify도 검토했으나 최종적으로 Vercel로 결정했다. 탈락 사유와 비교 메모는 `netlify-alternative.md` 참고.
