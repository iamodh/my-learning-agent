# Next.js 파일 기반 라우팅

## 핵심
- Next.js(app 디렉토리)는 폴더 구조가 그대로 URL이 된다. 별도 라우터 설정 파일이 없다.
- 약속된 파일명에 정해진 역할이 있다: `page.tsx`(페이지), `layout.tsx`(공통 레이아웃).
- 동적 경로는 폴더 이름을 대괄호로 감싼다: `app/posts/[slug]/page.tsx`.

## 동작 방식
- `app/about/page.tsx` → `/about` 경로 자동 생성.
- `app/posts/[slug]/page.tsx` → `/posts/hello` 요청 시 `slug`가 `"hello"`.
- 컴포넌트는 `params`로 동적 세그먼트 값을 받는다.

## 예시
```tsx
// app/posts/[slug]/page.tsx
export default async function PostPage({
  params,
}: {
  params: { slug: string };
}) {
  const { slug } = params;
  // slug로 글 데이터 가져오기
  return (
    <article>
      <h1>{slug}</h1>
    </article>
  );
}
```

`slug`를 키로 글 내용을 찾아오는 함수만 붙이면 글 페이지가 완성된다.
