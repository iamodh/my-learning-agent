# Next.js 개인 블로그

Next.js로 개인 블로그를 만든다. 구현 흐름은 파일 기반 라우팅으로 페이지를 만들고, 블로그 글은 SSG로 빌드 시점에 생성하며, 콘텐츠는 MDX로 작성하고, Vercel에 배포한 뒤 metadata API와 사이트맵, robots.txt로 SEO를 챙기는 방향이다.

## 참조 지식

- `knowledge/nextjs-app-routing-and-ssg.md`: 블로그의 URL 구조와 글 상세 페이지를 만들고, 글을 빌드 시점에 정적 페이지로 생성하기 위해 필요하다.
- `knowledge/mdx-content.md`: 블로그 글을 마크다운 기반으로 작성하고, 필요하면 React 컴포넌트를 섞어 쓰기 위해 필요하다.
- `knowledge/vercel-deployment.md`: Next.js 블로그를 배포하고 로컬/배포 환경 변수를 분리해 관리하기 위해 필요하다.
- `knowledge/nextjs-seo.md`: 글별 메타데이터, 공유 카드, sitemap.xml, robots.txt를 구성하기 위해 필요하다.

## 결정 사항

초기에는 Netlify도 검토했으나 최종적으로 Vercel로 배포하기로 했다.
