# OAuth 기반 로그인

OAuth 로그인은 사용자의 비밀번호를 서비스 서버가 직접 받지 않고, 구글이나 깃허브 같은 외부 인증 제공자를 통해 사용자를 확인하는 방식이다.

대화에서는 이메일/비밀번호를 직접 관리하는 방식은 부담스럽기 때문에 구글이나 깃허브 OAuth를 사용하려는 방향이 확인됐다. 예시는 구글 OAuth 기준으로 다뤘다.

## 기본 흐름

1. 사용자를 구글 인증 페이지로 보낸다.
2. 사용자가 동의하면 구글이 서버의 콜백 URL로 인가 코드를 붙여 리다이렉트한다.
3. 서버는 인가 코드를 구글에 보내 access token으로 교환한다.
4. access token으로 사용자 프로필을 가져온다.
5. 받은 사용자 정보를 DB에 upsert하고 로그인 처리를 한다.

핵심은 사용자의 비밀번호가 서비스 서버를 거치지 않는다는 점이다.

## Express 예시

```js
app.get('/auth/google', (req, res) => {
  const params = new URLSearchParams({
    client_id: process.env.GOOGLE_CLIENT_ID,
    redirect_uri: 'https://myapp.com/auth/google/callback',
    response_type: 'code',
    scope: 'openid email profile',
  });
  res.redirect(`https://accounts.google.com/o/oauth2/v2/auth?${params}`);
});

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
  const profile = await userRes.json();

  res.redirect('/');
});
```

## 라이브러리 언급

프로덕션에서는 `passport-google-oauth20` 같은 라이브러리가 state 검증과 에러 처리를 도와줄 수 있다고 언급됐다. 다만 대화에서는 흐름 이해를 위해 직접 `fetch`로 토큰을 교환하는 최소 예시를 중심으로 다뤘다.
