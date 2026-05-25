# 개인 블로그 (Next.js)

Next.js로 처음 만드는 개인 블로그. 글은 마크다운으로 쓰고, 빌드 시점에 정적으로 생성한 뒤 Vercel에 배포해서 검색 엔진이 잘 인식하도록 SEO 메타 태그까지 챙기는 것을 목표로 한다.

## 구성

- **라우팅**: App Router 기반. 글 페이지는 `app/posts/[slug]/page.tsx`로 만들어 동적 경로를 받는다.
- **글 생성 방식**: SSG. `generateStaticParams`에서 모든 slug 목록을 반환해 빌드 시점에 정적 HTML을 생성한다. 새 글이 추가될 때마다 푸시해서 재빌드하는 흐름.
- **콘텐츠 포맷**: MDX. 프론트매터(YAML 헤더)로 제목·날짜·태그 같은 메타 정보를 적고, 필요한 곳에 리액트 컴포넌트를 섞어 쓴다.
- **배포**: Vercel. GitHub 연동으로 푸시할 때마다 자동 배포. 환경 변수는 로컬은 `.env.local`, 배포는 Vercel 대시보드의 Environment Variables에 등록.
- **SEO**: metadata API. `layout.tsx`에 사이트 공통 메타데이터를 두고 각 글의 `page.tsx`에서 `generateMetadata`로 글마다 동적 메타 태그를 채운다. `sitemap.ts`, `robots.ts`도 함께 둔다.

초기 검토 단계에서 Netlify도 검토했으나, Next.js 신기능 1급 지원이 Vercel 쪽이라는 점을 고려해 Vercel로 결정했다.

## 진행 순서

1. 페이지 한두 개와 글 한 편만 가지고 끝까지 배포까지 한 번 굴려본다.
2. 이후 MDX 전역 컴포넌트, 사이트맵, OG 태그 같은 디테일을 붙인다.

## 참조 매핑

| 지식 문서 | 참조 이유 |
|---|---|
| `knowledge/app-router-routing.md` | 글 페이지를 `app/posts/[slug]/page.tsx`라는 동적 경로로 만들기 위해. |
| `knowledge/ssg-and-isr.md` | 블로그 글을 빌드 시점에 정적 HTML로 미리 생성하기 위해 `generateStaticParams`를 사용한다. |
| `knowledge/mdx.md` | 글 본문을 마크다운으로 작성하면서도 리액트 컴포넌트를 끼워 넣기 위해 MDX 포맷을 채택. 프론트매터로 글 메타 정보를 관리한다. |
| `knowledge/vercel-deployment.md` | GitHub 푸시 기반 자동 배포와 환경 변수 관리를 Vercel로 처리하기 위해. |
| `knowledge/seo-metadata.md` | 글마다 다른 메타 태그(title/description/OG)를 동적으로 채우고, 사이트맵과 robots를 자동 생성하기 위해. |
