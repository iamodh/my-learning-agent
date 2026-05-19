# bcrypt 비밀번호 해싱

## 핵심

비밀번호는 평문으로 저장하면 안 되고, 단방향 해시로 변환해 저장한다. 로그인 시에는 입력값을 다시 해싱해 저장된 해시와 비교한다. Node에서는 비밀번호 저장 전용으로 설계된 `bcrypt` 라이브러리를 쓴다.

## 왜 평문 저장이 금지인가

DB가 유출되면 모든 사용자의 비밀번호가 그대로 노출된다. 사람들이 여러 사이트에 같은 비밀번호를 재사용하기 때문에 피해가 연쇄적으로 번진다.

## bcrypt가 일반 해시와 다른 점

- **임의 salt 자동 부착**: 매번 다른 salt를 생성해 비밀번호에 붙인 뒤 해싱한다. 같은 비밀번호도 매번 다른 해시값이 나오므로 미리 계산된 해시 테이블(레인보우 테이블) 공격이 무력화된다. salt는 결과 문자열에 함께 인코딩되어 따로 저장할 필요가 없다.
- **의도적으로 느림 (work factor)**: 해시 계산을 수천~수만 번 반복시켜, 무차별 대입 공격의 시간 비용을 크게 올린다.

## saltRounds (work factor)

1 올릴 때마다 해시 계산 시간이 2배가 된다. 보안과 성능의 트레이드오프다.

- 너무 낮으면 무차별 공격에 약하고, 너무 높으면 로그인이 느려진다.
- 권장값은 10~12. 일반 서버 기준 10은 약 100ms, 12는 약 250~400ms.
- 보안 우선이면 12, 트래픽이 많고 응답 속도가 중요하면 10으로 시작.
- 자기 서버에서 직접 시간을 재서 200~300ms 안에 드는 값을 고르는 것이 좋다.

## 사용법

핵심은 두 함수다. 가입 시 `hash`로 저장, 로그인 시 `compare`로 검증한다.

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

`compare`는 저장된 해시에서 salt를 꺼내 입력값에 다시 적용해 비교하므로, salt를 별도로 다룰 필요가 없다.
