# MDX

MDX는 마크다운에 JSX를 섞어 쓸 수 있게 해주는 포맷이다. 평소처럼 `#` 제목과 `**굵게**` 같은 마크다운 문법을 쓰다가, 중간에 리액트 컴포넌트를 그냥 넣어 쓸 수 있다. 블로그에서 가끔 인터랙티브한 위젯이나 임베드를 넣고 싶을 때 유연하다.

## 프론트매터

글의 제목·날짜·태그 같은 메타 정보는 파일 맨 위의 YAML 헤더(프론트매터)에 적는다.

```mdx
---
title: 첫 번째 글
date: 2025-05-01
tags: [next, blog]
---

여기부터 본문 내용...
```

빌드할 때 이 메타 정보를 파싱해서 글 목록을 만들거나 메타 태그에 꽂아 쓸 수 있다. 프론트매터 파싱은 `gray-matter` 같은 라이브러리를 함께 쓰면 편하다.

## Next.js에 MDX 붙이기

`@next/mdx`를 설치하고 `next.config`를 래핑하면 `.mdx` 파일도 페이지로 인식된다.

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

## MDX 안에서 리액트 컴포넌트 쓰기

두 가지 방법이 있다.

- MDX 파일 안에서 직접 `import` 구문으로 가져와 쓰는 방법이 가장 직관적이다.
- 매번 import하기 번거로우면 루트에 `mdx-components.tsx` 파일을 만들어 전역으로 쓸 컴포넌트를 등록해두면 된다. 그러면 모든 MDX에서 import 없이 그냥 쓸 수 있다.
