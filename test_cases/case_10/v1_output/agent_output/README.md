# Next.js 개인 블로그 학습 정리

대화를 기반으로 학습한 지식과 그 지식으로 만들려던 결과물을 인과 관계에 따라 정리했다.
`knowledge/`의 개별 개념들이 `result/`의 블로그 제작에 사용된 구조다.

## 폴더 구조

```
.
├── README.md                          이 파일 (문서 배치 안내)
├── knowledge/                         학습한 지식 (개념 단위)
│   ├── routing/                       페이지 구성 / 글 생성
│   │   ├── file-based-routing.md      파일 기반 라우팅, 동적 경로
│   │   └── ssg-and-isr.md             SSG / ISR 정적 생성
│   ├── content/                       콘텐츠 작성 / 검색 노출
│   │   ├── mdx.md                     MDX 포맷, 프론트매터, 연동
│   │   └── seo.md                     metadata API, OG/Twitter, 사이트맵
│   └── delivery/                      배포 / 운영
│       ├── deployment.md              Vercel vs Netlify 비교 및 결정
│       └── env-variables.md           로컬 vs 배포 환경 변수
└── result/                            만들려던 결과물
    └── personal-blog.md               Next.js 개인 블로그 (지식 참조 + 이유)
```

## 인과 관계

- `result/personal-blog.md` 가 최종 목표(블로그)이며, 위 6개 지식 문서를 단계별로 참조한다.
- `knowledge/`는 주제별로 묶었다: 라우팅·정적 생성(routing), 콘텐츠·SEO(content), 배포·설정(delivery).

## 읽는 순서 (권장)

1. `result/personal-blog.md` — 무엇을 만들고 어떤 지식을 왜 썼는지 한눈에
2. `knowledge/routing/` → `knowledge/content/` → `knowledge/delivery/` 순으로 세부 개념
