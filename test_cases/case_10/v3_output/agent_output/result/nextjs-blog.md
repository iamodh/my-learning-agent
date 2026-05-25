# Next.js 개인 블로그

처음으로 Next.js를 사용해 만들고자 한 개인 블로그. 파일 기반 라우팅으로 페이지를 만들고, 빌드 시점에 글을 미리 생성하며, 마크다운으로 글을 쓰고, Vercel에 배포한 뒤 검색에 잘 노출되도록 메타데이터까지 챙기는 흐름으로 구성한다.

## 전체 흐름

라우팅 이해 → SSG로 글 페이지 생성 → MDX로 글 작성 → Vercel 배포 → SEO 메타데이터 정비.

처음에는 페이지 한두 개와 글 한 편만 가지고 끝까지 배포해본 뒤, 그 위에 MDX 컴포넌트나 사이트맵 같은 디테일을 더해 가는 방향으로 진행한다.

## 구성 요소

- 글 페이지: `app/posts/[slug]/page.tsx` 동적 경로로 글 하나를 보여주는 페이지.
- 글 콘텐츠: `.mdx` 파일로 작성, 프론트매터에 제목·날짜·태그 등을 적는다.
- 배포: GitHub에 푸시하면 Vercel이 자동 배포.
- SEO: 페이지별 metadata와 `sitemap.ts`, `robots.ts`.

## 사용된 지식과 참조 이유

- [Next.js App Router와 파일 기반 라우팅](../knowledge/nextjs-app-router.md) — 블로그 페이지(`/`, `/posts/[slug]` 등)를 폴더 구조만으로 만들기 위해 사용. 동적 경로 `[slug]`로 글 페이지를 표현한다.
- [SSG와 ISR](../knowledge/ssg-and-isr.md) — 블로그 글은 자주 바뀌지 않으므로 빌드 시점에 정적 HTML로 미리 만들어두기 위해 SSG를 채택. `generateStaticParams`로 모든 글의 slug를 미리 등록한다. ISR은 옵션으로 알아두되, 개인 블로그에서는 글 올릴 때마다 푸시해 재빌드하는 쪽을 기본으로 한다.
- [MDX](../knowledge/mdx.md) — 글을 마크다운으로 쓰면서도 필요할 때 리액트 컴포넌트를 본문에 끼워 넣을 수 있도록 채택. 프론트매터로 글 메타 정보를 관리한다.
- [Vercel 배포와 환경 변수](../knowledge/vercel-deploy.md) — Next.js를 만든 회사라 기능 호환이 가장 좋고 GitHub 연동 자동 배포가 가능해서 채택. 환경 변수는 로컬 `.env.local`과 Vercel 대시보드 양쪽에서 관리한다. (초기에는 Netlify도 검토했으나 Next.js 신규 기능 지원이 한 박자 늦을 수 있어 Vercel로 결정.)
- [Next.js SEO와 metadata API](../knowledge/nextjs-seo-metadata.md) — 검색 노출과 공유 카드 표시를 위해 페이지별 메타 태그, Open Graph, Twitter 카드, canonical URL을 관리. 글마다 다른 메타 태그는 `generateMetadata`로, 전체 사이트의 사이트맵과 robots는 `sitemap.ts`·`robots.ts`로 처리한다.
