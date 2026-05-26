# 빌드 오류 기록 — 2026-05-26

## 오류 개요

| 항목 | 내용 |
|------|------|
| 발생 일시 | 2026-05-26 |
| 명령어 | `npm run build` |
| 오류 코드 | `ERR_REQUIRE_ESM` |

## 오류 메시지

```
Error [ERR_REQUIRE_ESM]: require() of ES Module
  node_modules/@tailwindcss/vite/dist/index.mjs not supported.
Instead change the require of index.mjs to a dynamic import().
```

## 원인 분석

`package.json`에 `"type": "module"` 필드가 없었기 때문에 Node.js가 프로젝트를 **CommonJS(CJS) 모드**로 인식했다.

Vite가 `vite.config.ts`를 로드할 때 내부적으로 CJS 방식(`require()`)을 사용하는데,
`@tailwindcss/vite`는 **ESM 전용 패키지**(`index.mjs`만 제공)라 `require()`로 불러올 수 없어 충돌이 발생했다.

```
vite.config.ts 로드 시도
  → CJS require('@tailwindcss/vite')
  → @tailwindcss/vite는 .mjs(ESM) 전용
  → ERR_REQUIRE_ESM 발생
```

## 해결 방법

`package.json`에 `"type": "module"` 추가.
이 필드가 있으면 Node.js가 프로젝트 전체를 ESM으로 처리하여 `.mjs` 패키지를 정상적으로 `import`할 수 있다.

```diff
// package.json
{
  "name": "notes-app",
  "private": true,
+ "type": "module",
  "scripts": { ... }
}
```

## 결과

```
vite v6.4.1 building for production...
✓ 35 modules transformed.
dist/index.html                   0.78 kB │ gzip:  0.44 kB
dist/assets/index-DU4xE3h5.css   13.77 kB │ gzip:  3.36 kB
dist/assets/index-Cu4T0d3N.js   200.82 kB │ gzip: 63.15 kB
✓ built in 455ms
```

빌드 성공.

## 참고

- `@tailwindcss/vite` v4 이상은 ESM 전용이므로 `"type": "module"` 설정이 필수.
- `vite.config.ts` 자체도 `import` 구문을 사용하고 있어 ESM 방식이 올바른 설정이다.
