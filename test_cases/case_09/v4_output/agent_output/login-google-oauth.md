# 구글 OAuth 로그인 + 세션

## 무엇을 만드는가

사이드 프로젝트의 로그인 기능. 이메일/비밀번호를 직접 관리하지 않고, 구글 OAuth로 사용자를 식별한 뒤 서버가 세션을 발급해 HttpOnly 쿠키로 들고 다닌다.

구성 요소는 두 개의 라우트와 세션 발급이다.

- `GET /auth/google` — 사용자를 구글 인증 페이지로 리다이렉트한다.
- `GET /auth/google/callback` — 인가 코드를 access token으로 교환하고 사용자 프로필을 받아온 뒤, 세션에 `userId`를 심고 홈으로 리다이렉트한다.

이후 모든 인증 필요 라우트는 세션의 `userId`로 사용자를 식별한다.

## 어떤 지식을 썼는가 (참조 매핑)

- [`knowledge/oauth.md`](knowledge/oauth.md) — OAuth 3단계 흐름과 구글 기준 Express 구현
- [`knowledge/session.md`](knowledge/session.md) — 세션 발급, sliding + absolute 만료 정책

## 왜 그 지식을 썼는가 (참조 이유)

- **OAuth**: 비밀번호를 우리 서버에서 다루지 않기 위해 외부 인증 제공자에 위임해야 했다. 구글의 인가 코드 → access token → 사용자 프로필 교환 흐름이 그대로 콜백 라우트에 들어간다.
- **세션**: 콜백에서 한 번 식별된 사용자를 이후 요청까지 들고 다녀야 하므로, 콜백 끝에서 `req.session.userId`를 심는다. 만료 정책은 sliding 30분 + absolute 14일을 기준으로 둔다.
