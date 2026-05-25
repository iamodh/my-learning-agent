# Next.js metadata API로 SEO 챙기기

Next.js의 metadata API는 각 페이지의 `<head>` 메타 태그를 코드로 선언하면 빌드 시점에 알아서 꽂아주는 기능이다. 페이지 파일에서 `metadata` 객체를 export하면 된다.

```ts
export const metadata = {
  title: "사이트 제목",
  description: "사이트 설명",
};
```

`layout.tsx`에 사이트 공통 메타데이터를 적고, 각 `page.tsx`에서 페이지별로 덮어쓰는 식으로 쓰면 된다.

## 어떤 메타 태그를 챙길까

- `title`, `description`: 기본.
- Open Graph (`og:title`, `og:description`, `og:image`)와 Twitter 카드 태그: 공유했을 때 카드로 예쁘게 뜨게 하려면 필요.
- canonical URL: 같은 글이 여러 URL로 접근될 수 있을 때 지정.

metadata API에서는 `openGraph`, `twitter`, `alternates` 필드에 그대로 적으면 된다.

## 글마다 다른 메타 태그 — generateMetadata

`page.tsx`에서 비동기 함수 `generateMetadata`를 export하면 `params`로 받은 값으로 그 글의 데이터를 불러와 메타데이터를 동적으로 만들 수 있다.

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

`app` 디렉토리 루트에 `sitemap.ts`와 `robots.ts` 파일을 만들면 Next.js가 자동으로 `/sitemap.xml`, `/robots.txt` 경로로 노출해준다.

- `sitemap.ts`: 모든 글 slug를 돌면서 URL 배열을 반환한다.
- `robots.ts`: 어떤 경로의 크롤링을 허용/거부할지 적는다.
