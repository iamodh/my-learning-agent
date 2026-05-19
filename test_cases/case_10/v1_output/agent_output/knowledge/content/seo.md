# Next.js SEO (metadata API)

## 핵심
- `export const metadata = { ... }`를 적으면 Next.js가 `<head>`에 메타 태그를 자동으로 꽂는다.
- `layout.tsx`에 사이트 공통 메타데이터, 각 `page.tsx`에서 페이지별로 덮어쓴다.

## 챙겨야 할 메타 태그
- 기본: `title`, `description`.
- 공유 카드: Open Graph(`og:title`, `og:description`, `og:image`), Twitter 카드.
- 중복 URL 대응: canonical URL.
- metadata API의 `openGraph`, `twitter`, `alternates` 필드에 그대로 작성.

## 글마다 다른 메타 태그
`page.tsx`에서 `generateMetadata` 비동기 함수를 export한다. `params`로 글 데이터를 불러와 동적으로 반환.

```tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.summary,
  };
}
```

## 사이트맵 / robots
- `app` 루트에 `sitemap.ts`, `robots.ts` 파일을 만들면 `/sitemap.xml`, `/robots.txt`로 자동 노출된다.
- `sitemap.ts`: 모든 글 slug를 돌며 URL 배열 반환.
- `robots.ts`: 크롤링 허용/거부 경로 지정.
