# 산출물 구조

Next.js 개인 블로그 제작 과정에서 학습한 지식과 만들려는 결과를 정리한 문서들이다.

```
.
├── README.md
├── personal-blog.md                       # 결과 문서: 만들려는 블로그
└── knowledge/
    ├── nextjs-app-router-routing.md       # 파일 기반/동적 라우팅
    ├── ssg-and-isr.md                     # 빌드 시점 정적 생성과 ISR
    ├── mdx.md                             # MDX 포맷·프론트매터·Next.js 설정
    ├── vercel-deployment.md               # Vercel 배포와 환경 변수 (Netlify는 탈락)
    └── nextjs-seo-metadata.md             # metadata API·generateMetadata·sitemap/robots
```

- `personal-blog.md`가 결과 문서로, `knowledge/` 아래 각 지식 문서를 참조한다.
- 각 지식 문서는 서로 독립적으로 재사용 가능한 단위로 묶었고, 한 문서 내에서만 쓰이는 하위 측면(예: MDX 프론트매터, generateMetadata, 환경 변수)은 별도 파일로 쪼개지 않고 해당 문서의 섹션으로 두었다.
