# OAuth 로그인

OAuth는 외부 인증 제공자(구글, 깃허브 등)에게 사용자 식별을 위임하는 방식이다. 비밀번호가 우리 서버를 거치지 않는다는 점이 핵심이다.

## 동작 흐름

1. 사용자를 구글/깃허브 인증 페이지로 보내 동의를 받는다.
2. 동의하면 인증 제공자가 우리 서버의 콜백 URL로 인가 코드(authorization code)를 붙여 리다이렉트한다.
3. 우리 서버가 그 코드를 들고 인증 제공자에 다시 요청해서 access token으로 교환하고, 그 토큰으로 사용자 프로필(이메일, 이름 등)을 가져온다.

## Express 구현 예시 (구글 기준)

직접 fetch로 토큰 교환하는 최소 버전이다.

```js
// 1) 로그인 시작: 구글 인증 페이지로 보내기
app.get('/auth/google', (req, res) => {
  const params = new URLSearchParams({
    client_id: process.env.GOOGLE_CLIENT_ID,
    redirect_uri: 'https://myapp.com/auth/google/callback',
    response_type: 'code',
    scope: 'openid email profile',
  });
  res.redirect(`https://accounts.google.com/o/oauth2/v2/auth?${params}`);
});

// 2) 콜백: 인가 코드 → access token → 사용자 정보
app.get('/auth/google/callback', async (req, res) => {
  const { code } = req.query;

  const tokenRes = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      code,
      client_id: process.env.GOOGLE_CLIENT_ID,
      client_secret: process.env.GOOGLE_CLIENT_SECRET,
      redirect_uri: 'https://myapp.com/auth/google/callback',
      grant_type: 'authorization_code',
    }),
  });
  const { access_token } = await tokenRes.json();

  const userRes = await fetch('https://www.googleapis.com/oauth2/v2/userinfo', {
    headers: { Authorization: `Bearer ${access_token}` },
  });
  const profile = await userRes.json(); // { email, name, ... }

  // 여기서 DB에 upsert하고 로그인 처리
  res.redirect('/');
});
```

프로덕션에서는 `passport-google-oauth20` 같은 라이브러리가 state 검증, 에러 처리를 대신 해줘서 편하다. 처음 흐름을 익힐 때는 직접 짜보는 편이 이해에 도움된다.
