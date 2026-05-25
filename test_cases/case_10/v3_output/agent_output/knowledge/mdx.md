# MDX

MDX는 마크다운에 JSX를 섞어 쓸 수 있게 해주는 포맷이다. `# 제목`·`**굵게**` 같은 평범한 마크다운 문법을 그대로 쓰다가 본문 중간에 리액트 컴포넌트를 그냥 넣어 쓸 수 있다. 블로그에서 가끔 인터랙티브한 위젯이나 임베드를 넣고 싶을 때 유연하다.

## 프론트매터

파일 맨 위에 YAML 헤더(프론트매터)로 글의 메타 정보를 적는다.

```mdx
---
title: 첫 번째 글
date: 2025-05-01
tags: [next, blog]
---

여기부터 본문 내용...
```

빌드할 때 이 메타 정보를 파싱해서 글 목록을 만들거나 메타 태그에 꽂아 쓸 수 있다. 프론트매터 파싱은 `gray-matter` 같은 라이브러리를 추가로 쓰면 편하다.

## Next.js에 MDX 붙이기

`@next/mdx` 패키지를 설치하고 `next.config`에서 래핑한다.

```js
// next.config.mjs
import createMDX from '@next/mdx';

const withMDX = createMDX({
  extension: /\.mdx?$/,
});

const nextConfig = {
  pageExtensions: ['ts', 'tsx', 'md', 'mdx'],
};

export default withMDX(nextConfig);
```

이렇게 설정하면 `.mdx` 파일도 페이지로 인식된다.

## MDX 안에서 리액트 컴포넌트 쓰기

두 가지 방법이 있다.

- MDX 파일 안에서 직접 `import` 구문으로 컴포넌트를 가져와 쓴다. 가장 직관적이다.
- 매번 import하기 귀찮으면 루트에 `mdx-components.tsx` 파일을 만들고 전역으로 쓸 컴포넌트를 등록한다. 그러면 모든 MDX에서 import 없이 그냥 쓸 수 있다.
