# Next.js SEO

Next.js에서는 metadata API를 사용해 페이지별 메타 태그를 설정할 수 있다. `layout.tsx`에는 사이트 공통 메타데이터를 두고, 각 `page.tsx`에서 페이지별 메타데이터를 덮어쓰는 흐름으로 사용한다.

## 기본 메타데이터

기본적으로 `title`과 `description`을 챙긴다. 공유 카드 표시를 위해 Open Graph 태그와 Twitter 카드 태그도 언급됐다.

- `og:title`
- `og:description`
- `og:image`
- Twitter 카드 태그
- canonical URL

metadata API에서는 `openGraph`, `twitter`, `alternates` 필드를 사용할 수 있다.

## 글별 메타데이터

글마다 다른 메타 태그가 필요하면 `page.tsx`에서 `generateMetadata` 비동기 함수를 export한다.

```tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.summary,
  };
}
```

`params`로 글 데이터를 불러와 글에 맞는 `title`과 `description`을 반환한다.

## sitemap.xml과 robots.txt

블로그에는 사이트맵과 robots.txt가 있는 것이 좋다고 정리됐다. `app` 디렉토리 루트에 `sitemap.ts`와 `robots.ts`를 만들면 Next.js가 자동으로 `/sitemap.xml`, `/robots.txt` 경로를 만든다.

`sitemap.ts`에서는 모든 글 slug를 순회해 URL 배열을 반환하고, `robots.ts`에서는 크롤링 허용/거부 경로를 적는다.
