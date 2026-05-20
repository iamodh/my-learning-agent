# MDX 콘텐츠

MDX는 마크다운에 JSX를 섞어 쓸 수 있게 해주는 포맷이다. 일반 마크다운처럼 제목이나 굵은 글씨를 쓰면서, 중간에 React 컴포넌트를 넣을 수 있다.

## 프론트매터

MDX 파일 맨 위에는 제목, 날짜, 태그 같은 메타 정보를 YAML 헤더로 적을 수 있다.

```mdx
---
title: 첫 번째 글
date: 2025-05-01
tags: [next, blog]
---

여기부터 본문 내용...
```

이 메타 정보는 빌드할 때 파싱해서 글 목록이나 메타 태그에 사용할 수 있다.

## Next.js MDX 설정

`@next/mdx` 패키지를 사용하고 `next.config`를 래핑한다.

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

이렇게 하면 `.mdx` 파일도 페이지로 인식된다. 프론트매터 파싱에는 `gray-matter` 같은 라이브러리를 추가로 쓸 수 있다.

## 컴포넌트 사용

MDX 안에서 React 컴포넌트를 쓰는 방법은 두 가지가 언급됐다.

- MDX 파일 안에서 직접 `import`해서 사용한다.
- 루트에 `mdx-components.tsx`를 만들고 전역 컴포넌트를 등록해 모든 MDX에서 import 없이 사용한다.
