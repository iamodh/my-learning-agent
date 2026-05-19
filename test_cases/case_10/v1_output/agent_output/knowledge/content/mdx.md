# MDX로 콘텐츠 작성

## 핵심
- MDX는 마크다운에 JSX를 섞어 쓸 수 있는 포맷. `# 제목`, `**굵게**` 사이에 리액트 컴포넌트를 그대로 넣을 수 있다.
- 인터랙티브 위젯이나 임베드를 글 안에 넣고 싶을 때 유연하다.

## 프론트매터
파일 맨 위에 YAML 헤더로 메타 정보를 적는다. 빌드 시 파싱해 글 목록이나 메타 태그에 사용한다.

```mdx
---
title: 첫 번째 글
date: 2025-05-01
tags: [next, blog]
---

여기부터 본문 내용...
```

## Next.js 연동
`@next/mdx`를 설치하고 `next.config`에서 래핑한다.

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

- 프론트매터 파싱은 `gray-matter` 같은 라이브러리를 추가로 쓰면 편하다.

## MDX 안에서 컴포넌트 사용
- MDX 파일 안에서 직접 `import` 구문으로 가져와 쓰는 방법 (가장 직관적).
- 루트에 `mdx-components.tsx`를 만들어 전역 컴포넌트를 등록하면 모든 MDX에서 import 없이 사용 가능.
