# bcrypt 비밀번호 해싱

bcrypt는 비밀번호를 평문으로 저장하지 않기 위해 사용하는 단방향 해시 방식이다. 가입 시 비밀번호를 해시로 바꿔 저장하고, 로그인 시 입력값을 저장된 해시와 비교한다.

## 평문 저장 금지

비밀번호를 그대로 DB에 저장하면 DB 유출 시 모든 사용자의 비밀번호가 노출된다. 사용자가 여러 사이트에서 같은 비밀번호를 쓰는 경우 피해가 다른 서비스로 번질 수 있다.

## bcrypt의 특징

bcrypt는 매번 임의의 salt를 만들어 비밀번호에 붙인 뒤 해싱한다. 그래서 같은 비밀번호라도 매번 다른 해시값이 나온다. salt와 해시는 결과 문자열 안에 함께 들어가므로 따로 저장할 필요가 없다.

bcrypt는 의도적으로 느리게 동작한다. `saltRounds`는 work factor이며, 값이 1 증가할 때마다 해시 계산 시간이 2배가 된다. 대화에서는 현실적인 범위로 10~12가 언급됐다. 보안을 중시하면 12, 트래픽과 응답 속도를 중시하면 10으로 시작하고, 서버에서 직접 시간을 재서 200~300ms 안에 들어오는 값을 고르는 방식이 언급됐다.

## 사용 예시

```js
const bcrypt = require('bcrypt');
const saltRounds = 12;

// 회원가입
app.post('/signup', async (req, res) => {
  const { email, password } = req.body;
  const hashed = await bcrypt.hash(password, saltRounds);
  await User.create({ email, password: hashed });
  res.json({ ok: true });
});

// 로그인 검증
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ where: { email } });
  const valid = await bcrypt.compare(password, user.password);
  if (!valid) return res.status(401).json({ error: 'invalid' });
  res.json({ ok: true });
});
```

`compare`는 저장된 해시에서 salt를 꺼내 입력값에 다시 적용해 비교한다.
