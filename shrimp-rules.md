# Development Guidelines

## 프로젝트 개요

- Notion을 CMS로 활용한 개인 기술 블로그
- Next.js 15 App Router + TypeScript + Tailwind CSS + shadcn/ui
- Notion API(`@notionhq/client`)로 글 목록·상세 데이터 조회
- Vercel 배포, ISR 적용

---

## 프로젝트 아키텍처

### 디렉토리 구조 (엄수)

```
src/
├── app/
│   ├── layout.tsx              # 전역 레이아웃
│   ├── page.tsx                # 홈 (글 목록)
│   ├── globals.css
│   └── posts/
│       └── [slug]/
│           └── page.tsx        # 글 상세
├── components/
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── PostCard.tsx
│   ├── PostContent.tsx
│   ├── CategoryFilter.tsx
│   └── SearchBar.tsx
├── lib/
│   └── notion.ts               # Notion API 유틸리티 (유일한 API 호출 지점)
└── types/
    └── notion.ts               # 공통 타입 정의
```

- **새 컴포넌트**: 반드시 `src/components/` 하위에 생성
- **새 타입**: 반드시 `src/types/notion.ts`에 추가
- **Notion API 함수**: 반드시 `src/lib/notion.ts`에만 작성

---

## 언어 규칙

- 변수명·함수명·파일명: **영어**
- 코드 주석: **한국어**
- 커밋 메시지: **한국어**
- 문서(md 파일): **한국어**

---

## Notion API 규칙

### 함수 위치 및 시그니처 (`src/lib/notion.ts`)

```typescript
// Notion 클라이언트 인스턴스 — 파일 최상단에 단일 인스턴스로 생성
const notion = new Client({ auth: process.env.NOTION_API_KEY })

// 글 목록 조회 — Status === '발행됨' 필터 필수
fetchPages(filter?: { category?: string }): Promise<Post[]>

// 글 상세 블록 조회
fetchPageContent(pageId: string): Promise<NotionBlock[]>

// slug(pageId)로 단일 글 조회
fetchPageBySlug(slug: string): Promise<Post | null>
```

- **페이지 컴포넌트에서 직접 `@notionhq/client` import 금지** → 반드시 `src/lib/notion.ts` 함수 경유
- **조회 필터 조건**: `Status === '발행됨'` 항상 적용, 초안 노출 금지
- **정렬 조건**: 발행일(`Published`) 기준 내림차순

### 환경변수

| 변수명 | 위치 |
|--------|------|
| `NOTION_API_KEY` | `.env.local` (커밋 금지) |
| `NOTION_DATABASE_ID` | `.env.local` (커밋 금지) |

- `.env.example`에는 키 값 없이 변수명만 기재
- `.gitignore`에 `.env.local` 포함 확인 필수

---

## 타입 정의 규칙 (`src/types/notion.ts`)

```typescript
type Post = {
  id: string
  title: string
  category: string
  tags: string[]
  publishedAt: string
  status: '초안' | '발행됨'
  slug: string
}

type NotionBlock = {
  type: string
  // 블록 타입별 속성 추가
}
```

- 새 타입 추가 시 `src/types/notion.ts`에만 작성
- `any` 타입 사용 금지

---

## 컴포넌트 규칙

### 서버/클라이언트 컴포넌트 판단 기준

| 조건 | 지정 |
|------|------|
| Notion API 호출 필요 | 서버 컴포넌트 (기본값) |
| `useState`, `useEffect` 필요 | 클라이언트 컴포넌트 (`'use client'`) |
| 이벤트 핸들러 필요 | 클라이언트 컴포넌트 (`'use client'`) |

- **`'use client'` 선언은 최소화** — 필요한 리프 컴포넌트에만 적용
- 카테고리 필터(`CategoryFilter.tsx`), 검색 바(`SearchBar.tsx`)는 클라이언트 컴포넌트
- 페이지 컴포넌트(`page.tsx`)는 서버 컴포넌트 유지

### shadcn/ui 컴포넌트 사용

- 버튼: `Button` (shadcn/ui)
- 카드: `Card`, `CardContent`, `CardHeader` (shadcn/ui)
- 배지(태그): `Badge` (shadcn/ui)
- 입력: `Input` (shadcn/ui)
- 아이콘: `lucide-react`

---

## ISR 규칙

```typescript
// 글 목록 페이지 (src/app/page.tsx)
export const revalidate = 60

// 글 상세 페이지 (src/app/posts/[slug]/page.tsx)
export const revalidate = 3600
```

- 두 값 임의 변경 금지 — PRD 9절 기준값 준수

---

## 라우팅 규칙

| 경로 | 파일 | 설명 |
|------|------|------|
| `/` | `src/app/page.tsx` | 글 목록, 카테고리 필터 |
| `/posts/[slug]` | `src/app/posts/[slug]/page.tsx` | 글 상세 |
| `/?category=Frontend` | URL 파라미터 | 카테고리 필터 상태 |

- slug = Notion 페이지 ID
- 존재하지 않는 slug 접근 시 반드시 `notFound()` 호출
- `generateStaticParams()` → 글 상세 페이지 빌드 시 정적 경로 생성 필수

---

## SEO 규칙

- `src/app/layout.tsx`: 기본 `title`, `description` 메타데이터 설정
- `src/app/posts/[slug]/page.tsx`: `generateMetadata()`로 글 제목 기반 OG 태그 생성
- `src/app/sitemap.ts`: 발행된 글 URL 포함 동적 사이트맵 생성
- `src/app/robots.ts`: robots.txt 설정

---

## 블록 렌더링 규칙 (`src/components/PostContent.tsx`)

| Notion 블록 타입 | 렌더링 결과 |
|-----------------|-------------|
| `paragraph` | `<p>` |
| `heading_1` | `<h1>` |
| `heading_2` | `<h2>` |
| `heading_3` | `<h3>` |
| `code` | `<pre><code>` (언어 표시) |
| `image` | `<Image>` (next/image) |
| `quote` | `<blockquote>` |
| `bulleted_list_item` | `<ul><li>` |
| `numbered_list_item` | `<ol><li>` |

- 이미지 렌더링 시 반드시 `next/image` 사용 (`<img>` 태그 금지)
- 지원하지 않는 블록 타입은 렌더링 생략 (에러 throw 금지)

---

## 다중 파일 수정 규칙

| 작업 | 동시 수정 파일 |
|------|---------------|
| 새 Post 속성 추가 | `src/types/notion.ts` + `src/lib/notion.ts` + 관련 컴포넌트 |
| 새 페이지 라우트 추가 | `src/app/[route]/page.tsx` + `src/app/sitemap.ts` |
| 새 Notion 블록 타입 지원 | `src/lib/notion.ts` + `src/components/PostContent.tsx` + `src/types/notion.ts` |

---

## 금지 사항

- **`.env.local` 커밋** — 절대 금지
- **페이지 컴포넌트에서 `@notionhq/client` 직접 import** — `src/lib/notion.ts` 경유 필수
- **`Status !== '발행됨'` 글 노출** — 필터 조건 제거 금지
- **`any` 타입 사용** — 금지
- **`<img>` 태그로 이미지 렌더링** — `next/image` 사용 필수
- **불필요한 `'use client'` 선언** — 서버 컴포넌트 우선 원칙 위반
- **`src/lib/notion.ts` 외 파일에 Notion API 함수 작성** — 단일 책임 원칙 위반
- **ISR `revalidate` 값 임의 변경** — PRD 기준값 준수 필수
