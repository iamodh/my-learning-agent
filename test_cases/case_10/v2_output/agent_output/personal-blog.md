# 개인 블로그 (Next.js)

Next.js로 만드는 개인 블로그. 파일 기반 라우팅으로 페이지를 잡고, SSG로 빌드 시점에 글을 미리 생성하고, MDX로 콘텐츠를 작성하고, Vercel에 배포하면서 환경 변수를 관리하고, metadata API로 SEO를 챙기는 흐름으로 구성한다.

배포 플랫폼은 초기에 Netlify도 검토했으나 최종적으로 Vercel로 결정했다.

## 참조한 지식과 이유

| 지식 | 참조 이유 |
|------|----------|
| [Next.js App Router 라우팅](knowledge/nextjs-app-router-routing.md) | 블로그 페이지를 만들기 위한 기반. 글 페이지는 `/posts/[slug]` 동적 경로로 설계한다. |
| [SSG와 ISR](knowledge/ssg-and-isr.md) | 블로그 글은 빌드 시점에 정적으로 생성하는 것이 자연스러워 `generateStaticParams`로 모든 slug를 만든다. |
| [MDX](knowledge/mdx.md) | 글을 마크다운으로 쓰되 필요할 때 리액트 컴포넌트를 끼워 넣기 위해 채택. 프론트매터로 제목·날짜 같은 메타 정보를 표현한다. |
| [Vercel 배포](knowledge/vercel-deployment.md) | Next.js 기능을 별도 설정 없이 그대로 쓰기 위해 선택. 환경 변수는 `.env.local`과 Vercel 대시보드로 분리해 관리한다. |
| [Next.js SEO와 metadata API](knowledge/nextjs-seo-metadata.md) | 검색 노출이 중요한 블로그 특성상 글마다 동적으로 메타 태그를 채우고 사이트맵·robots를 제공한다. |
