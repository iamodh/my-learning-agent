# Next.js SEO와 metadata API

Next.js의 metadata API는 페이지마다 `export const metadata = { ... }`를 적어두면 알아서 `<head>`에 메타 태그를 꽂아준다. `layout.tsx`에 사이트 공통 메타데이터를 적고, 각 `page.tsx`에서 페이지별로 덮어쓰는 식으로 쓴다.

## 챙길 메타 태그

- `title`, `description`: 기본.
- Open Graph(`og:title`, `og:description`, `og:image`): 공유 시 카드로 예쁘게 뜨게 한다.
- Twitter 카드 태그.
- canonical URL: 같은 글이 여러 URL로 접근될 수 있을 때 지정한다.

metadata API의 `openGraph`, `twitter`, `alternates` 필드에 그대로 적으면 된다.

## 동적 메타데이터 (generateMetadata)

글마다 다른 메타 태그가 필요할 때는 `page.tsx`에서 `generateMetadata` 비동기 함수를 export한다. `params`를 받아 글 데이터를 불러온 뒤 `title`/`description`을 글 내용에 맞춰 반환한다.

```tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.summary,
  };
}
```

## 사이트맵과 robots.txt

`app` 디렉토리 루트에 `sitemap.ts`와 `robots.ts`를 만들면 Next.js가 자동으로 `/sitemap.xml`, `/robots.txt` 경로로 노출해준다.

- `sitemap.ts`: 모든 글 slug를 돌며 URL 배열을 반환한다.
- `robots.ts`: 크롤링 허용/거부 경로를 적는다.
