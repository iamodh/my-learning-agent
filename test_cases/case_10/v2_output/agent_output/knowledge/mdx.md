# MDX

MDX는 마크다운에 JSX를 섞어 쓸 수 있게 해주는 포맷이다. 평소처럼 `#` 제목과 `**굵게**`를 쓰다가 중간에 리액트 컴포넌트를 그대로 넣을 수 있어, 블로그에서 인터랙티브 위젯이나 임베드를 끼워 넣기에 유연하다.

## 프론트매터

파일 맨 위에 YAML 헤더를 적어 글의 메타 정보를 표현한다.

```mdx
---
title: 첫 번째 글
date: 2025-05-01
tags: [next, blog]
---

여기부터 본문 내용...
```

빌드 시 이 메타 정보를 파싱해 글 목록을 만들거나 메타 태그에 꽂아 쓸 수 있다. 파싱에는 `gray-matter` 같은 라이브러리를 쓰면 편하다.

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

이러면 `.mdx` 파일도 페이지로 인식된다.

## MDX 안에서 컴포넌트 사용

두 가지 방법이 있다.

- MDX 파일 안에서 직접 `import` 구문으로 컴포넌트를 가져와 쓴다.
- 루트에 `mdx-components.tsx`를 만들어 전역으로 쓸 컴포넌트를 등록해두면, 모든 MDX에서 import 없이 그대로 쓸 수 있다.
