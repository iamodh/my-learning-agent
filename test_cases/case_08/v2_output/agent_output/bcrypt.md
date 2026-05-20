# bcrypt: 비밀번호 해싱

bcrypt는 비밀번호 저장을 위해 설계된 단방향 해시 알고리즘이다. 평문 저장은 DB 유출 시 모든 사용자의 비밀번호가 그대로 노출되고, 같은 비밀번호를 여러 사이트에서 재사용하는 관행 때문에 피해가 연쇄적으로 번지므로 절대 금지된다. 비밀번호는 단방향 해시로 변환해 저장하고, 로그인 시 입력값을 다시 해싱해 비교한다.

bcrypt가 일반 해시 함수(SHA-256 등)와 다른 두 가지 핵심:

- **임의의 salt**: 매번 다른 salt를 생성해 비밀번호에 붙인 뒤 해싱한다. 같은 비밀번호도 매번 다른 해시값이 나오고, 미리 계산된 해시 테이블(레인보우 테이블) 공격이 무력화된다. salt는 결과 문자열에 함께 인코딩되므로 따로 저장할 필요가 없다.
- **의도적으로 느림**: work factor를 통해 해시 계산을 수천~수만 번 반복시켜 무차별 대입의 시간 비용을 키운다.

## 사용 (Node.js)

핵심 함수는 두 개다. 가입 시 `hash`로 저장하고, 로그인 시 `compare`로 검증한다.

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

`compare`는 내부적으로 저장된 해시에서 salt를 꺼내 입력값에 다시 적용해 비교하므로 salt를 별도로 다룰 필요가 없다.

## saltRounds (work factor)

saltRounds는 해시 계산 반복 강도다. 1 올릴 때마다 계산 시간이 2배가 된다. 보안과 성능의 트레이드오프이며, 너무 낮으면 무차별 공격에 약하고 너무 높으면 로그인이 느려져 UX가 나빠진다.

- 권장값: **10~12** 사이.
- 일반적인 서버 기준: 10은 약 100ms, 12는 약 250~400ms.
- 보안 우선이면 12, 트래픽이 많고 응답 속도가 중요하면 10으로 시작.
- 자기 서버에서 직접 측정해 200~300ms 안에 들어오는 값을 고르는 것이 좋다.
