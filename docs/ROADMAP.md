# ROADMAP: 개인 개발 블로그 (Notion CMS 기반)

> PRD 참고: [docs/PRD.md](./PRD.md)  
> 총 예상 기간: **9~14일**  
> 작성일: 2026-04-21

---

## 전체 일정 요약

| Phase | 내용 | 예상 소요 | 누적 기간 |
|-------|------|-----------|-----------|
| Phase 1 | 프로젝트 골격 | 1~2일 | 1~2일 |
| Phase 2 | 공통 모듈 | 2~3일 | 3~5일 |
| Phase 3 | 핵심 기능 | 3~4일 | 6~9일 |
| Phase 4 | 추가 기능 | 2~3일 | 8~12일 |
| Phase 5 | 최적화 및 배포 | 1~2일 | 9~14일 |

---

## Phase 1: 프로젝트 골격

**예상 소요 기간**: 1~2일

**왜 이 순서인가?**
골격이 없으면 어디에도 코드를 붙일 수 없다. 라우팅 구조, 환경변수, 폴더 컨벤션이 먼저 확정되어야 이후 모든 작업이 방향을 잃지 않는다. 특히 Notion API 키 없이는 어떤 기능도 실제 데이터로 검증할 수 없으므로, 환경 설정이 가장 먼저 와야 한다.

### 작업 목록

#### 1-1. Next.js 프로젝트 구조 설정
- [ ] Next.js 15 프로젝트 초기화 (App Router)
- [ ] TypeScript 설정 (`tsconfig.json` 검토)
- [ ] Tailwind CSS 설치 및 설정
- [ ] shadcn/ui 초기화 및 기본 컴포넌트 설치 (Button, Card, Badge, Input)
- [ ] Lucide React 설치
- [ ] ESLint / Prettier 설정

#### 1-2. Notion API 연동 환경 구축
- [ ] `@notionhq/client` 패키지 설치
- [ ] Notion Integration 생성 및 API 키 발급
- [ ] Notion 블로그 데이터베이스 생성 (PRD 3절 속성 구조 반영)
- [ ] `.env.local` 파일 생성 및 환경변수 설정
  ```env
  NOTION_API_KEY=secret_xxxx
  NOTION_DATABASE_ID=xxxx
  ```
- [ ] `.env.example` 파일 작성 (키 값 제외)
- [ ] `.gitignore`에 `.env.local` 포함 확인

#### 1-3. 기본 레이아웃 구조 생성
- [ ] `src/app/layout.tsx` — 전역 레이아웃 (폰트, 메타 기본값)
- [ ] `src/app/globals.css` — 전역 스타일 기본값
- [ ] 기본 폴더 구조 생성 (`components/`, `lib/`, `types/`)

### 완료 기준

- `npm run dev` 실행 시 로컬에서 Next.js 앱이 정상 구동됨
- Notion API 키로 간단한 테스트 호출이 성공함 (콘솔 로그로 확인)
- `src/` 하위 기본 폴더 구조가 PRD 8절 디렉토리 구조와 일치함

---

## Phase 2: 공통 모듈

**예상 소요 기간**: 2~3일

**왜 이 순서인가?**
글 목록 페이지와 상세 페이지는 둘 다 `fetchPages`, `Post` 타입, `PostCard` 컴포넌트를 공유한다. 공통 모듈 없이 페이지부터 만들면 나중에 같은 코드를 두 번 작성하거나, 이미 만든 페이지를 다시 고쳐야 하는 역행이 발생한다. 타입 정의를 이 단계에서 확정해야 이후 TypeScript 오류 없이 페이지 개발을 진행할 수 있다.

### 작업 목록

#### 2-1. Notion API 공통 함수
- [ ] `src/lib/notion.ts` — Notion 클라이언트 인스턴스 생성
- [ ] `fetchPages(filter?)` — 데이터베이스에서 글 목록 조회
  - `Status === '발행됨'` 필터 적용
  - 발행일 기준 내림차순 정렬
  - 카테고리 필터 파라미터 지원
- [ ] `fetchPageContent(pageId)` — 개별 페이지 블록 조회
- [ ] `fetchPageBySlug(slug)` — slug(페이지 ID)로 단일 글 조회

#### 2-2. 공통 타입 정의
- [ ] `src/types/notion.ts` 작성
  ```typescript
  type Post {
    id: string
    title: string
    category: string
    tags: string[]
    publishedAt: string
    status: '초안' | '발행됨'
    slug: string
  }

  type NotionBlock {
    type: string
    // 블록 타입별 속성
  }
  ```

#### 2-3. 공통 컴포넌트
- [ ] `src/components/Header.tsx` — 로고/블로그명, 검색 바 포함
- [ ] `src/components/Footer.tsx` — 기본 푸터
- [ ] `src/components/PostCard.tsx` — 글 카드 (제목, 카테고리, 태그, 날짜)

### 완료 기준

- `fetchPages()` 호출 시 Notion 데이터베이스의 발행된 글 목록이 `Post[]` 타입으로 반환됨
- `fetchPageContent(id)` 호출 시 페이지 블록 배열이 반환됨
- `Header`, `Footer`, `PostCard` 컴포넌트가 Storybook 없이 임시 페이지에서 시각적으로 확인됨
- TypeScript 컴파일 오류 없음 (`tsc --noEmit` 통과)

---

## Phase 3: 핵심 기능

**예상 소요 기간**: 3~4일

**왜 이 순서인가?**
글 목록과 글 상세, 이 두 페이지가 동작해야 비로소 블로그라고 부를 수 있다. 카테고리 필터나 검색은 이 두 페이지가 없으면 붙일 곳 자체가 없다. 핵심 기능을 먼저 완성하고 안정화해야, 이후 추가 기능이 기반을 흔들지 않는다.

### 작업 목록

#### 3-1. 블로그 글 목록 페이지 (`/`)
- [ ] `src/app/page.tsx` — 홈 페이지 서버 컴포넌트
- [ ] `fetchPages()` 호출로 글 목록 데이터 조회
- [ ] `PostCard` 컴포넌트로 카드 그리드 렌더링
- [ ] ISR 적용: `export const revalidate = 60`
- [ ] 글이 없을 때 빈 상태(empty state) 처리

#### 3-2. 블로그 글 상세 페이지 (`/posts/[slug]`)
- [ ] `src/app/posts/[slug]/page.tsx` — 상세 페이지 서버 컴포넌트
- [ ] `generateStaticParams()` — 빌드 시 정적 경로 생성
- [ ] `generateMetadata()` — 글 제목 기반 동적 메타 태그 생성
- [ ] 글 제목, 카테고리, 태그, 발행일 메타 정보 표시
- [ ] ISR 적용: `export const revalidate = 3600`
- [ ] 존재하지 않는 slug 접근 시 `notFound()` 처리

#### 3-3. Notion 컨텐츠 렌더링
- [ ] `src/components/PostContent.tsx` — 블록 렌더러 컴포넌트
- [ ] 블록 타입별 렌더링 구현
  | 블록 타입 | 렌더링 결과 |
  |-----------|-------------|
  | `paragraph` | `<p>` |
  | `heading_1` | `<h1>` |
  | `heading_2` | `<h2>` |
  | `heading_3` | `<h3>` |
  | `code` | `<pre><code>` (언어 표시) |
  | `image` | `<Image>` (Next.js) |
  | `quote` | `<blockquote>` |
  | `bulleted_list_item` | `<ul><li>` |
  | `numbered_list_item` | `<ol><li>` |

### 완료 기준

- 홈(`/`)에서 Notion 데이터베이스의 발행된 글이 카드 형태로 렌더링됨
- 카드 클릭 시 `/posts/[slug]` 상세 페이지로 이동함
- 상세 페이지에서 Notion 본문 블록이 올바르게 렌더링됨 (단락, 제목, 코드, 이미지 포함)
- 존재하지 않는 URL 접근 시 404 페이지가 표시됨
- `npm run build` 성공 (빌드 오류 없음)

---

## Phase 4: 추가 기능

**예상 소요 기간**: 2~3일

**왜 이 순서인가?**
카테고리 필터와 검색은 글 목록 페이지 위에 올라타는 기능이다. 핵심 기능이 안정된 후에 추가해야 의존 관계가 단방향으로 유지된다. 반대로 했을 경우, 핵심 기능을 수정할 때 필터·검색 로직이 함께 깨지는 연쇄 오류가 발생할 수 있다. SEO는 배포 전 이 단계에서 처리해야 초기 색인부터 올바르게 잡힌다.

### 작업 목록

#### 4-1. 카테고리 필터링
- [ ] `src/components/CategoryFilter.tsx` — 카테고리 탭 컴포넌트
- [ ] Notion 데이터베이스에서 카테고리 목록 동적 추출
- [ ] URL 파라미터 기반 필터 상태 관리 (`?category=Frontend`)
- [ ] 선택된 카테고리 탭 하이라이트 처리
- [ ] `fetchPages({ category })` 필터 파라미터 연동
- [ ] `[전체]` 선택 시 필터 해제

#### 4-2. 검색 기능
- [ ] `src/components/SearchBar.tsx` — 검색 입력 컴포넌트
- [ ] 글 제목 기준 클라이언트 사이드 실시간 필터링
- [ ] 검색어 입력 시 결과가 즉시 업데이트됨
- [ ] 검색 결과 없음 상태 처리
- [ ] Header 내 검색 바 통합

#### 4-3. SEO 최적화
- [ ] `src/app/layout.tsx` — 기본 메타데이터 설정 (`title`, `description`)
- [ ] 글 상세 페이지 `generateMetadata()` — OG 태그 추가 (`og:title`, `og:description`)
- [ ] `src/app/sitemap.ts` — 동적 사이트맵 생성
- [ ] `src/app/robots.ts` — robots.txt 설정

### 완료 기준

- 카테고리 탭 클릭 시 URL이 `/?category=Frontend` 형태로 변경되고 해당 글만 표시됨
- 페이지 새로고침 후에도 선택한 카테고리 필터 상태가 유지됨
- 검색어 입력 시 제목 기준으로 글 목록이 실시간 필터링됨
- `/sitemap.xml` 접근 시 발행된 글 URL이 포함된 사이트맵이 반환됨
- 브라우저 소셜 미리보기 툴에서 OG 태그가 올바르게 노출됨

---

## Phase 5: 최적화 및 배포

**예상 소요 기간**: 1~2일

**왜 이 순서인가?**
모든 기능이 완성된 후에 최적화해야 실측 데이터를 보고 어디를 개선할지 판단할 수 있다. 개발 중 섣부른 최적화는 아직 바뀔 수 있는 코드에 시간을 쏟는 낭비다. 반응형도 UI가 확정된 시점에 한 번에 점검하는 것이 수정 비용이 가장 적다.

### 작업 목록

#### 5-1. 성능 최적화
- [ ] `next/image` 최적화 — 이미지 `width`, `height`, `priority` 속성 검토
- [ ] Lighthouse 성능 측정 및 LCP 2.5초 이하 달성 확인
- [ ] ISR `revalidate` 값 최종 검토 (글 목록: 60초, 글 상세: 3600초)
- [ ] 불필요한 클라이언트 컴포넌트 (`'use client'`) 서버 컴포넌트로 전환 검토
- [ ] `@notionhq/client` 호출 중복 방지 (요청 단위 캐시 적용)

#### 5-2. 반응형 디자인 개선
- [ ] 모바일(375px), 태블릿(768px), 데스크탑(1280px) 3개 구간 레이아웃 점검
- [ ] 글 카드 그리드 반응형 열 수 조정 (모바일 1열, 태블릿 2열, 데스크탑 3열)
- [ ] 글 상세 페이지 본문 최대 너비(max-width) 및 가독성 개선
- [ ] 코드 블록 가로 스크롤 처리

#### 5-3. Vercel 배포
- [ ] GitHub 저장소와 Vercel 프로젝트 연결
- [ ] Vercel 환경변수 설정 (`NOTION_API_KEY`, `NOTION_DATABASE_ID`)
- [ ] 프로덕션 빌드 성공 확인
- [ ] 커스텀 도메인 설정 (선택 사항)
- [ ] 배포 후 주요 페이지(홈, 글 상세, 카테고리 필터, 검색) 동작 검증

### 완료 기준

- Lighthouse 성능 점수 90점 이상 (Performance, Accessibility, Best Practices, SEO)
- 모바일/태블릿/데스크탑 3개 구간에서 레이아웃 깨짐 없음
- Vercel 프로덕션 URL에서 모든 기능이 정상 동작함
- 빌드 로그에 오류 또는 경고 없음

---

## 진행 상황 추적

| Phase | 상태 | 시작일 | 완료일 |
|-------|------|--------|--------|
| Phase 1: 프로젝트 골격 | 미시작 | - | - |
| Phase 2: 공통 모듈 | 미시작 | - | - |
| Phase 3: 핵심 기능 | 미시작 | - | - |
| Phase 4: 추가 기능 | 미시작 | - | - |
| Phase 5: 최적화 및 배포 | 미시작 | - | - |

---

## MVP 이후 백로그

PRD 6.2절에서 정의된 MVP 이후 기능으로, 우선순위 순으로 정렬하였다.

| 우선순위 | 기능 | 설명 |
|----------|------|------|
| 1 | 다크 모드 | Tailwind `dark:` 클래스 + `next-themes` 적용 |
| 2 | OG 이미지 자동 생성 | `@vercel/og`로 글 제목 기반 OG 이미지 생성 |
| 3 | 페이지네이션 | 글 목록 페이지 분할 (한 페이지 10개) |
| 4 | RSS 피드 | `/feed.xml` 엔드포인트 생성 |
| 5 | 댓글 기능 | Giscus (GitHub Discussions 기반) 연동 |
| 6 | 글 조회수 카운트 | Vercel KV 또는 외부 DB 활용 |

---

*최초 작성일: 2026-04-21*
