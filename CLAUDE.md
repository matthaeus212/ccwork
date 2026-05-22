# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

React 19 + TypeScript + Vite 기반 노트 앱 실습 프로젝트. 백엔드는 JSON Server(`db.json`)를 사용하며, 프론트와 동시에 실행된다.

- 앱: http://localhost:5173
- API: http://localhost:3001/notes

## 주요 명령어

```bash
npm run dev        # Vite + JSON Server 동시 실행 (개발 시 이걸 쓸 것)
npm run server     # JSON Server만 단독 실행
npm run build      # tsc + vite build
npm run lint       # ESLint --fix
npm run format     # Prettier --write
npm test           # Vitest 단일 실행
npm run test:watch # Vitest watch 모드
```

## 아키텍처

### 데이터 흐름

```
JSON Server (db.json)
    ↑↓ fetch (REST)
src/api/notes.ts          ← 모든 HTTP 호출 집중 (fetchNotes / createNote / updateNote / deleteNote)
    ↑↓
src/context/NotesContext.tsx  ← 전역 상태 (notes[], loading, error) + CRUD 메서드
    ↑↓ useNotes() hook
컴포넌트 (NoteList / NoteEditor / NoteItem)
```

### 상태 관리 패턴

- `NotesContext`가 서버 상태의 단일 진실 공급원.
- API 성공 후 로컬 `setNotes`로 즉시 반영 (낙관적 업데이트 아님 — 서버 응답값 사용).
- `selectedNoteId` / `isCreating` UI 상태는 `App.tsx`에서 관리하고 props로 내려준다.

### 컴포넌트 구조

- `Layout` — 사이드바(NoteList) + 메인(NoteEditor) 2단 레이아웃
- `NoteList` → `NoteItem` 목록 렌더링 + 삭제 버튼
- `NoteEditor` — 생성/편집 겸용. `isCreating` 플래그로 모드 분기

### 스타일

Tailwind CSS v4 (`@tailwindcss/vite` 플러그인). 별도 설정 파일 없이 vite.config에서 플러그인으로 적용.

### 타입

`src/types/note.ts`의 `Note` 인터페이스가 전체 앱의 공용 타입. `tags` 필드는 아직 미구현(향후 강의에서 추가 예정).

## 테스트

Vitest + @testing-library/react. `vite.config.ts`에 테스트 설정이 통합되어 있다(`globals: true`, `environment: jsdom`). 셋업 파일: `src/test-setup.ts`.

## 프로젝트 구조

```
ccwork/
├── src/
│   ├── api/
│   │   └── notes.ts          # 모든 HTTP 호출 집중 (컴포넌트에서 직접 fetch 금지)
│   ├── components/
│   │   ├── Layout.tsx        # 사이드바 + 메인 2단 레이아웃 래퍼
│   │   ├── NoteEditor.tsx    # 생성/편집 겸용 폼 (isCreating 플래그로 분기)
│   │   ├── NoteItem.tsx      # 노트 목록의 개별 항목 + 삭제 버튼
│   │   └── NoteList.tsx      # 노트 목록 렌더링 + 새 노트 버튼
│   ├── context/
│   │   └── NotesContext.tsx  # 전역 상태(notes, loading, error) + CRUD 메서드
│   ├── types/
│   │   └── note.ts           # Note 인터페이스 (앱 전체 공용 타입)
│   ├── App.tsx               # 루트 컴포넌트 — UI 상태(selectedNoteId, isCreating) 관리
│   ├── main.tsx              # React 엔트리포인트
│   └── index.css             # Tailwind CSS 진입점 (@import "tailwindcss")
├── db.json                   # JSON Server 데이터 파일 (로컬 DB)
├── vite.config.ts            # Vite + Vitest + Tailwind 플러그인 통합 설정
└── package.json
```

## 네이밍 패턴 규칙

### 파일명

| 종류 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | PascalCase.tsx | `NoteEditor.tsx` |
| Context | PascalCase + Context 접미사, .tsx | `NotesContext.tsx` |
| API 모듈 | camelCase.ts | `notes.ts` |
| 타입 정의 | lowercase.ts | `note.ts` |

### 함수 / 변수

- **컴포넌트**: named export (`export function NoteEditor`). `App.tsx`만 default export.
- **커스텀 훅**: `use` 접두사 — `useNotes()`
- **이벤트 핸들러**: `handle` 접두사 — `handleSave`, `handleSelectNote`, `handleDone`
- **API 함수**: 동사 + 명사 — `fetchNotes`, `createNote`, `updateNote`, `deleteNote`
- **Context 노출 메서드**: 내부 액션 동사 — `addNote`, `editNote`, `removeNote`

### 타입 / 인터페이스

- Props 인터페이스: `{ComponentName}Props` (예: `NoteEditorProps`)
- Context 타입: `{Name}ContextType` (예: `NotesContextType`)
- Context 객체: `{Name}Context` (예: `NotesContext`)
- Provider 컴포넌트: `{Name}Provider` (예: `NotesProvider`)

## 네이밍 패턴 위반 현황

현재 코드베이스에서 위 규칙과 어긋난 부분.

| 파일 | 위반 규칙 | 위치 |
|------|-----------|------|
| `src/components/NoteItem.tsx` | 이벤트 핸들러 `handle` 접두사 누락 — 인라인 화살표 함수 사용 | line 13, 25-28 |
| `src/test-setup.ts` | 파일명 kebab-case — camelCase여야 함 (`testSetup.ts`) | 파일명 |

### NoteItem.tsx 상세

```tsx
// 현재 (규칙 위반)
onClick={() => onSelect(note.id)}
onClick={(e) => { e.stopPropagation(); onDelete(note.id); }}

// 규칙대로
const handleSelect = () => onSelect(note.id);
const handleDelete = (e: React.MouseEvent) => { e.stopPropagation(); onDelete(note.id); };
```

### test-setup.ts 상세

- `test-setup.ts` → `testSetup.ts`로 변경 시 `vite.config.ts`의 `setupFiles` 경로도 함께 수정 필요.

## API 호출 패턴

### 원칙

- 모든 `fetch` 호출은 `src/api/notes.ts`에만 작성한다. 컴포넌트나 Context에서 직접 `fetch`하지 않는다.
- 각 API 함수는 `!res.ok`이면 즉시 `throw new Error(...)` — 호출부에서 try/catch로 처리.
- 타임스탬프(`createdAt`, `updatedAt`)는 API 레이어에서 자동 설정한다. 컴포넌트에서 넘기지 않는다.

### 함수 시그니처 패턴

```ts
// 목록 조회 — 반환 타입 명시
export async function fetchNotes(): Promise<Note[]>

// 생성 — id/타임스탬프는 Omit으로 제외
export async function createNote(note: Omit<Note, 'id' | 'createdAt' | 'updatedAt'>): Promise<Note>

// 수정 — Partial로 부분 업데이트
export async function updateNote(id: string, updates: Partial<Note>): Promise<Note>

// 삭제 — 반환값 없음
export async function deleteNote(id: string): Promise<void>
```

### Context → 컴포넌트 연결 패턴

```
API 함수 호출 (src/api/notes.ts)
    ↓ 성공 시
setNotes() 로컬 상태 즉시 반영 (서버 응답값 사용)
    ↓
useNotes() 훅으로 컴포넌트에 노출
```

- Context 메서드(`addNote`, `editNote`, `removeNote`)는 API 호출 + 로컬 상태 갱신을 함께 처리.
- 컴포넌트는 Context 메서드만 호출하고, 상태 변경 로직에 관여하지 않는다.
- API 에러는 Context 밖(컴포넌트 핸들러)에서 `alert` 또는 에러 상태로 처리.
