# Next.js SEO와 metadata API

Next.js는 metadata API를 제공해서 페이지마다 `export const metadata = { ... }`를 적어두면 `<head>`에 메타 태그를 자동으로 꽂아준다. `layout.tsx`에 사이트 공통 메타데이터를 두고, 각 `page.tsx`에서 페이지별로 덮어쓰는 식으로 쓴다.

## 챙겨야 할 메타 태그

- `title`, `description`: 기본.
- Open Graph 태그(`og:title`, `og:description`, `og:image`): 공유 시 카드로 예쁘게 뜨도록.
- Twitter 카드 태그: 같은 이유.
- canonical URL: 같은 글이 여러 URL로 접근될 수 있으면 지정해둔다.

metadata API에서는 `openGraph`, `twitter`, `alternates` 필드에 그대로 적는다.

## 글마다 다른 메타 태그 — generateMetadata

페이지에서 `generateMetadata` 비동기 함수를 export하면 `params`를 받아 글 데이터를 불러온 뒤 메타데이터를 동적으로 반환할 수 있다.

```tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.summary,
  };
}
```

## sitemap.ts와 robots.ts

`app` 디렉토리 루트에 `sitemap.ts`와 `robots.ts` 파일을 만들면 Next.js가 자동으로 `/sitemap.xml`, `/robots.txt` 경로로 만들어준다.

- `sitemap.ts`: 모든 글 slug를 돌면서 URL 배열을 반환.
- `robots.ts`: 어떤 경로를 크롤링 허용/거부할지 적는다.
