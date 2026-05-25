# Next.js 개인 블로그

처음 Next.js를 써서 만드는 개인 블로그. 글은 마크다운(MDX)으로 쓰고, 빌드 시점에 정적으로 생성한 뒤 Vercel에 배포한다. 검색 유입을 위해 SEO 메타데이터까지 챙기는 것을 목표로 한다.

## 무엇을 만드는가

- `/posts/[slug]` 형태의 동적 경로로 글 페이지를 만든다.
- 글은 빌드 시점에 모든 slug에 대해 정적 HTML로 미리 생성한다.
- 본문은 MDX로 작성하고, 프론트매터로 제목·날짜·태그 같은 메타 정보를 담는다.
- GitHub에 푸시하면 Vercel이 자동 배포한다. 환경 변수는 Vercel 대시보드에서 관리한다.
- 페이지마다 metadata API로 SEO 메타 태그를 꽂고, sitemap·robots도 생성한다.

## 구축 순서

1. app 디렉토리 라우팅으로 페이지 구조 잡기 (`app/posts/[slug]/page.tsx`).
2. `generateStaticParams`로 모든 글의 slug를 반환해 빌드 시점에 정적 생성.
3. MDX 패키지를 붙이고 `.mdx` 파일을 페이지로 인식하도록 설정.
4. GitHub 연동 → Vercel로 자동 배포. 환경 변수는 Vercel Settings에 등록.
5. layout.tsx에 공통 메타데이터, 각 글 페이지에 `generateMetadata`로 동적 메타 태그, `sitemap.ts`/`robots.ts` 추가.

배포 후보로 Netlify도 검토했으나, Next.js 신규 기능 1급 지원이 Vercel 쪽이라 Vercel로 결정.

## 참조 지식과 참조 이유

- `knowledge/nextjs-app-routing-and-ssg.md` — `/posts/[slug]` 동적 경로와 `generateStaticParams`로 빌드 시점 정적 생성을 구현하는 데 직접 사용.
- `knowledge/mdx-in-nextjs.md` — 본문을 마크다운으로 쓰고 프론트매터로 메타 정보를 담기 위해 MDX를 도입하고 `@next/mdx`로 설정하기 위해 사용.
- `knowledge/vercel-deployment.md` — GitHub 연동 자동 배포와 로컬/배포 환경의 환경 변수 분리를 위해 사용.
- `knowledge/nextjs-seo-metadata.md` — 글마다 다른 메타 태그를 동적으로 생성하고 sitemap·robots로 검색 노출을 챙기기 위해 사용.
