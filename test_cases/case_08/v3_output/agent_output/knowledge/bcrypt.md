# bcrypt — 비밀번호 해싱

bcrypt는 비밀번호 저장 전용으로 설계된 단방향 해시 알고리즘이다. 평문 비밀번호를 그대로 DB에 저장하면 유출 시 모든 사용자의 비밀번호가 노출되고, 사람들이 여러 사이트에 같은 비밀번호를 쓰기 때문에 피해가 연쇄적으로 번진다. 그래서 비밀번호는 단방향 해시로 변환해 저장하고, 로그인 시엔 입력값을 다시 해싱해 비교한다.

## 일반 해시와의 차이

일반 해시 함수(SHA-256 같은)와 비교한 bcrypt의 핵심 차이는 두 가지다.

- **임의 salt**: 매번 임의의 salt를 생성해 비밀번호에 붙여 해싱한다. 같은 비밀번호도 매번 다른 해시값이 나오므로 미리 계산해둔 해시 테이블(레인보우 테이블) 공격이 무력화된다.
- **의도적인 느림**: work factor 개념을 통해 해시 계산을 수천~수만 번 반복해, 공격자의 무차별 대입 시도에 엄청난 시간이 걸리게 만든다.

salt와 해시는 결과 문자열에 함께 인코딩되므로 salt를 별도 컬럼으로 저장할 필요가 없다.

## 사용 방법 — hash / compare

가입 시 `hash`로 저장하고, 로그인 시 `compare`로 검증한다.

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

`compare`는 저장된 해시 문자열에서 salt를 꺼내 입력값에 다시 적용해 비교하므로, 호출자가 salt를 따로 다룰 필요가 없다.

## saltRounds (work factor)

saltRounds는 work factor를 가리키는 값으로, 1 올릴 때마다 해시 계산 시간이 2배가 된다. 보안과 성능의 트레이드오프다.

- 너무 낮으면 무차별 공격에 약해진다.
- 너무 높으면 로그인 응답이 느려져 사용자 경험이 나빠진다.

현실적인 권장 범위는 10~12 사이다. 일반적인 서버 기준 10은 약 100ms, 12는 약 250~400ms 정도 걸린다. 보안 중시 환경이면 12, 트래픽이 많고 응답 속도가 중요하면 10에서 시작한 뒤, 자기 서버에서 직접 측정해 200~300ms 안에 들어오는 값을 고르는 것이 좋다.
