# 결과물: Next.js 개인 블로그

## 만들려고 한 것
Next.js로 개인 블로그를 처음부터 만들어 배포한다. 글은 마크다운으로 작성하고, 검색 노출까지 챙긴다.

## 전체 흐름
파일 기반 라우팅으로 페이지 구성 → SSG로 빌드 시점에 글 생성 → MDX로 콘텐츠 작성 → Vercel 배포 + 환경 변수 관리 → metadata API로 SEO.

처음엔 페이지 한두 개와 글 한 편으로 끝까지 배포해본 뒤, MDX 컴포넌트·사이트맵 같은 디테일을 채워나가는 순서로 진행한다.

## 사용된 지식과 이유

| 단계 | 사용 지식 | 왜 사용했는가 |
|------|-----------|----------------|
| 페이지 구성 | [파일 기반 라우팅](../knowledge/routing/file-based-routing.md) | 글 페이지가 `/posts/[slug]` 형태의 동적 경로라 폴더 기반 라우팅과 `params` 처리가 필요했다 |
| 글 생성 방식 | [SSG와 ISR](../knowledge/routing/ssg-and-isr.md) | 블로그 글은 자주 안 바뀌므로 빌드 시점에 정적 생성하는 게 빠르고 단순했다. 갱신 자동화 옵션으로 ISR도 검토 |
| 콘텐츠 작성 | [MDX](../knowledge/content/mdx.md) | 마크다운으로 편하게 글을 쓰되 가끔 인터랙티브 컴포넌트를 넣고 싶어 JSX를 섞을 수 있는 MDX를 선택 |
| 배포 | [배포: Vercel vs Netlify](../knowledge/delivery/deployment.md) | 어디에 올릴지 결정해야 했고, Next.js 1급 지원과 학습 단순화를 위해 Vercel로 결정 |
| 배포 설정 | [환경 변수 관리](../knowledge/delivery/env-variables.md) | 로컬과 배포 환경의 설정 값이 달라, 환경별 환경 변수 주입 방법이 필요했다 |
| 검색 노출 | [Next.js SEO](../knowledge/content/seo.md) | 블로그는 검색 유입이 중요해, 글마다 동적 메타 태그와 사이트맵/robots가 필요했다 |

## 의사결정 메모
배포 플랫폼은 가격 때문에 Netlify를 검토했으나, Next.js 호환성과 학습 단순화를 우선해 Vercel로 번복·확정했다.
