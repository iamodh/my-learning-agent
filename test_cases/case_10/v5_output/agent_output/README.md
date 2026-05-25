# Next.js 개인 블로그 학습 정리

Next.js로 개인 블로그를 만들기 위해 학습한 내용과, 그 과정에서 결정한 블로그 구성안을 정리했다.

## 폴더 구조

```
agent_output/
├── README.md
├── personal-blog.md           # 결과 문서: 개인 블로그 구성
└── knowledge/
    ├── app-router-routing.md  # App Router 파일 기반 라우팅 + 동적 경로
    ├── ssg-and-isr.md         # SSG(generateStaticParams) + ISR
    ├── mdx.md                 # MDX, 프론트매터, mdx-components
    ├── vercel-deployment.md   # Vercel 배포 + 환경 변수
    ├── seo-metadata.md        # metadata API + generateMetadata + sitemap/robots
    └── netlify-alternative.md # 검토 후 탈락한 대안
```

## 문서 간 관계

- `personal-blog.md`는 위 지식 문서들을 참조해 블로그를 어떻게 구성할지 정리한 결과 문서다.
- `knowledge/` 하위는 각각 독립된 개념 단위의 지식 문서다.
- `netlify-alternative.md`는 결과에 채택되지 않은 대안으로, 결과 문서의 참조 매핑에는 포함되지 않는다.
