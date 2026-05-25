# GraphQLError로 의도한 에러 throw

리졸버에서 의도한 에러(예: "유저 없음")를 클라이언트에 그대로 보여주고 싶을 때는 `GraphQLError` 인스턴스를 만들어 throw한다. `extensions.code`에 식별용 코드를 붙이면, 응답 가공 단계에서 그 코드를 보고 통과시킬지 마스킹할지 분기할 수 있다.

```js
import { GraphQLError } from 'graphql';

const resolvers = {
  Query: {
    user: async (_, { id }) => {
      const user = await db.user.findUnique({ where: { id } });
      if (!user) {
        throw new GraphQLError('User not found', {
          extensions: { code: 'NOT_FOUND' },
        });
      }
      return user;
    },
  },
};
```

리졸버는 의도한 에러만 코드를 붙여 throw하고, 예상 못한 에러는 그대로 흘려보낸다. 외부 노출 여부는 응답 가공 레이어에서 코드로 판단한다.
