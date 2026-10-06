### 1. 공개 영화 화면에 라우팅 연결하기

**검색 화면**

- `validateSearch`로 `query`를 문자열로 검증하고, 없으면 `undefined`로 처리한다.
- `useSearch`로 `query`를 읽고, `trim()`과 `toLowerCase()`로 정규화한 뒤 `includes()`로 제목과 원제를 비교한다.
- 폼 제출 시 `preventDefault()` 후 `useNavigate`로 `query`를 URL에 반영한다.
- 검색어가 없으면 `검색어를 입력해 주세요.`, 결과가 없으면 `검색 결과가 없어요.`를 표시한다.
- 결과에는 검색어, 결과 수, 포스터, 제목, 원제, 개봉일, 줄거리, 상세 링크를 표시한다.

  ![검색화면.png](%EA%B2%80%EC%83%89%ED%99%94%EB%A9%B4.png)


**상세 화면**

- `useParams`로 `movieId`를 읽고 `Number(movieId)`로 변환해 로컬 데이터에서 영화를 찾는다.
- 배경 이미지, 포스터, 제목, 원제, 개봉일, 장르, 러닝타임, 태그라인, 줄거리를 Figma 화면에 맞춰 표시한다.
- 일치하는 영화가 없으면 `영화를 찾을 수 없어요.`를 표시한다.

  ![영화검색.png](%EC%98%81%ED%99%94%EA%B2%80%EC%83%89.png)

  ![영화id없음.png](%EC%98%81%ED%99%94id%EC%97%86%EC%9D%8C.png)

**화면 이동**

- 영화 카드의 포스터와 제목, 검색 결과의 `상세 보기`를 `Link`로 연결했다.
- 헤더의 `영화`, `검색` 메뉴도 `Link`로 바꿔 새로고침 없이 화면을 전환한다.

  ![영화상세페이지.png](%EC%98%81%ED%99%94%EC%83%81%EC%84%B8%ED%8E%98%EC%9D%B4%EC%A7%80.png)


**확인 결과**

- [x]  `/` 영화 목록 표시
- [x]  `/search?query=스파이더맨` 직접 입력 시 결과 표시
- [x]  `/search`, `/search?query=zzz`에서 각각 안내 문구 표시
- [x]  `/movies/1` 새로고침 후에도 같은 영화 유지, 배경 이미지와 포스터 표시
- [x]  `/movies/999`에서 `영화를 찾을 수 없어요.` 표시
- [x]  카드와 검색 결과에서 상세 화면으로 이동

#### 2. 기존 CSS를 Tailwind CSS로 옮기기

- `tailwindcss`와 `@tailwindcss/vite`를 설치하고 `vite.config.ts`에 `tailwindcss()`를 추가했다. `src/index.css`는 `@import "tailwindcss";`로 교체했다.
- `clsx`와 `tailwind-merge`로 `utils/cn.ts`를 만들었다.
- `Header`, `MovieCard`, `MovieGrid`, `Pagination`, `Footer`, 목록, 검색, 상세 화면의 스타일을 utility class로 옮겼다.
- 스타일을 모두 옮긴 뒤 `App.css`를 import하는 파일이 없음을 확인하고 `App.css`를 삭제했다.

**상태에 따라 달라지는 class는 `cn`으로 조합**

```tsx
// utils/cn.ts
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// components/movies/movie-card.tsx
<button
  className={cn(
    "absolute right-2 top-2 grid size-8 place-items-center rounded-md",
    isBookmarked ? "bg-blue-600" : "bg-gray-900/60",
  )}
>
```

**확인 결과**

- [x]  Figma 데스크톱 화면(목록, 검색, 상세)과 비교
- [x]  브라우저 Console 오류 없음
- [x]  `pnpm build` 성공