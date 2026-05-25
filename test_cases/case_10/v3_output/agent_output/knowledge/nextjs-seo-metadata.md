# Next.js SEO와 metadata API

Next.js의 metadata API는 페이지마다 `export const metadata = { ... }`를 적어두면 알아서 `<head>`에 메타 태그를 꽂아준다. `layout.tsx`에 사이트 공통 메타데이터를 적고, 각 `page.tsx`에서 페이지별로 덮어쓰는 식으로 쓴다.

## 챙겨야 할 메타 태그

- `title`, `description` — 기본.
- Open Graph 태그 (`og:title`, `og:description`, `og:image`) — 공유했을 때 카드로 예쁘게 뜨도록.
- Twitter 카드 태그 — 트위터에서의 공유 카드.
- canonical URL — 같은 글이 여러 URL로 접근될 수 있을 때 정본 URL을 지정.

metadata API에서는 각각 `openGraph`, `twitter`, `alternates` 필드에 그대로 적는다.

## 글마다 다른 메타데이터 — generateMetadata

글 페이지처럼 페이지마다 메타 태그가 달라야 하면, `page.tsx`에서 `generateMetadata` 비동기 함수를 export한다. `params`를 받아 그 글의 데이터를 불러오고, 그에 맞게 메타데이터를 반환한다.

```tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.summary,
  };
}
```

이렇게 해두면 글마다 메타 태그가 동적으로 들어간다.

## 사이트맵과 robots.txt

`app` 디렉토리 루트에 `sitemap.ts`와 `robots.ts` 파일을 만들면 Next.js가 자동으로 `/sitemap.xml`, `/robots.txt` 경로로 만들어준다.

- `sitemap.ts` — 모든 글의 slug를 돌면서 URL 배열을 반환.
- `robots.ts` — 어떤 경로를 크롤링 허용/거부할지 지정.
