# Express 회원가입/로그인 인증 시스템

Express 서버에 회원가입과 로그인 인증을 붙였다. 가입 시점엔 bcrypt로 비밀번호를 안전하게 단방향 저장하고, 로그인 이후엔 JWT로 stateless하게 사용자 상태를 증명하는 구조다. 두 기술이 시점도 책임도 분리돼 있다.

## 사용한 지식

- [bcrypt](../knowledge/bcrypt.md) — 가입 시점에 비밀번호를 안전하게 보관해야 했기 때문. 평문 저장은 DB 유출 시 사용자가 여러 사이트에서 쓰는 같은 비밀번호까지 함께 노출시키므로, 단방향 해시로 저장하고 로그인 시 입력값을 다시 해시해 비교하는 흐름이 필요했다. 가입에선 `bcrypt.hash`, 로그인에선 `bcrypt.compare`를 사용한다.
- [JWT](../knowledge/jwt.md) — 로그인 이후 사용자 상태를 매 요청마다 다시 검증해야 하는데, 비밀번호를 매번 입력시킬 수 없다. JWT는 서명만 검증하면 되므로 별도 저장소 없이 stateless하게 동작해 SPA/모바일 친화적이라 선택했다.

## 흐름

1. **회원가입 요청** — 클라이언트가 이메일/비밀번호 전송 → 서버가 `bcrypt.hash`로 해시 → 이메일+해시를 DB에 저장.
2. **로그인 요청** — 클라이언트가 이메일/비밀번호 전송 → 서버가 이메일로 사용자 조회 → `bcrypt.compare`로 입력 비밀번호와 저장된 해시 비교 → 일치하면 `jwt.sign`으로 토큰 발급 → 토큰을 응답으로 전달.
3. **인증된 요청** — 클라이언트가 `Authorization` 헤더에 토큰을 실어 보냄 → 서버의 `verify` 미들웨어가 토큰 서명 검증 → `req.user`에 사용자 정보를 꽂아 다음 핸들러로 넘김.

## 구현

```js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const saltRounds = 12;
const SECRET = process.env.JWT_SECRET;

// 회원가입
app.post('/signup', async (req, res) => {
  const { email, password } = req.body;
  const hashed = await bcrypt.hash(password, saltRounds);
  await User.create({ email, password: hashed });
  res.json({ ok: true });
});

// 로그인
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ where: { email } });
  const valid = await bcrypt.compare(password, user.password);
  if (!valid) return res.status(401).json({ error: 'invalid' });

  const token = jwt.sign(
    { userId: user.id, email: user.email },
    SECRET,
    { expiresIn: '1h' }
  );
  res.json({ token });
});

// 인증 미들웨어 (이후 보호된 라우트에 적용)
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  try {
    const payload = jwt.verify(token, SECRET);
    req.user = payload;
    next();
  } catch (e) {
    res.status(401).json({ error: 'invalid token' });
  }
}
```
