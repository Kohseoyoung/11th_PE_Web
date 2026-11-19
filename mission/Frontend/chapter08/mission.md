#### 1. 로그인 전용 폴더 만들기

#### 1-1. 폴더 구조

`_auth`로 시작하는 pathless Layout Route를 만들고, 로그인 후에만 접근할 수 있는 페이지를 그 아래에 묶었다. 이름이 `_`로 시작하므로 URL에는 `/_auth`가 붙지 않는다.

```
src/routes/
├── _auth.tsx          ← 공통 Layout Route
└── _auth/
    └── me/
        ├── index.tsx  ← /me (마이페이지)
        └── edit.tsx   ← /me/edit (내 정보 수정)
```

| 구분 | 역할 |
| --- | --- |
| `(괄호)` Route Group | 파일만 묶고, 레이아웃 컴포넌트를 가질 수 없음 |
| `_` pathless Layout Route | 파일을 묶으면서 공통 레이아웃과 `beforeLoad`를 가질 수 있음 |

미션 조건(폴더로 묶기 + 공통 Layout Route + `<Outlet />`)을 한 번에 만족하도록 `_auth` 방식을 사용했다.

#### 1-2. 핵심 코드

```tsx
// routes/_auth.tsx
export const Route = createFileRoute("/_auth")({
  // 페이지를 표시하기 전에 토큰이 없으면 로그인 페이지로 이동
  beforeLoad: () => {
    if (!useAuthStore.getState().accessToken) {
      throw redirect({ to: "/login" });
    }
  },
  component: AuthLayout,
});
```

```tsx
// components/layout/auth-layout.tsx
export function AuthLayout() {
  // 페이지에 있는 동안 401로 토큰이 지워져도 로그인 페이지로 이동
  const accessToken = useAuthStore((state) => state.accessToken);
  if (!accessToken) return <Navigate to="/login" />;

  // 하위 페이지(/me, /me/edit)를 이 자리에 렌더링
  return <Outlet />;
}
```

| 코드 | 역할 |
| --- | --- |
| `beforeLoad` | 페이지 진입 전에 토큰을 검사하고, 없으면 `/login`으로 보낸다 |
| `<Navigate />` | 페이지에 머무는 중 토큰이 사라지면 `/login`으로 보낸다 |
| `<Outlet />` | 하위 페이지를 렌더링한다 |

#### 1-3. 확인 결과

| 상황 | 결과 |
| --- | --- |
| 로그인 상태에서 `/me`, `/me/edit` 접근 | 정상 표시 |
| 토큰을 지우고 `/me` 접근 | `/login`으로 이동 |
| 로그인 전 헤더의 "내 정보" 클릭 | `/login`으로 이동 |

---

#### 2. 남은 API 연결하기

#### 2-1. 연결한 API

| 화면 | API | 기능 |
| --- | --- | --- |
| 마이페이지 | `GET /favorites` | 내 즐겨찾기 목록 조회 |
| 영화 상세 | `GET /favorites/{movieId}` | 즐겨찾기 여부 조회 |
| 영화 상세 | `POST /favorites` | 즐겨찾기 등록 |
| 영화 상세 | `DELETE /favorites/{movieId}` | 즐겨찾기 해제 |
| 영화 상세 | `GET /members/{memberId}/ratings` | 내 평점 조회 |
| 영화 상세 | `POST /ratings` | 평점 등록 |
| 영화 상세 | `PATCH /ratings/{ratingId}` | 평점 수정 |
| 영화 상세 | `DELETE /ratings/{ratingId}` | 평점 삭제 |

모든 요청은 `authApi`로 보내서 `Authorization: Bearer {토큰}` 헤더가 자동으로 붙는다.

#### 2-2. 파일 구성

| 파일 | 역할 |
| --- | --- |
| `api/favorites/favorites.ts` | 즐겨찾기 요청 함수 4개 |
| `api/ratings/ratings.ts` | 평점 요청 함수 4개 |
| `hooks/use-favorite.ts` | 즐겨찾기 query와 mutation |
| `hooks/use-ratings.ts` | 평점 query와 mutation |
| `components/movies/favorite-button.tsx` | 상세 페이지 즐겨찾기 버튼 |
| `components/movies/my-rating-panel.tsx` | 상세 페이지 평점 패널 |
| `utils/auth/get-member-id.ts` | 토큰에서 memberId 추출 |

#### 2-3. 데이터 흐름에서 고민한 점

**① memberId 구하기**

- `GET /members/{memberId}/ratings`에는 memberId가 필요하지만, 내 정보 응답에는 memberId가 없다.
- 토큰의 payload(`sub`)에 memberId가 들어 있어서, 따로 저장하지 않고 토큰에서 계산했다.
- 서명 검증 없이 읽기만 하며, 실제 권한 확인은 서버가 한다.

```tsx
// utils/auth/get-member-id.ts
const payload = token.split(".")[1];
const json = JSON.parse(atob(payload.replace(/-/g, "+").replace(/_/g, "/")));
const memberId = Number(json.memberId ?? json.sub);
```

**② 마이페이지의 영화 정보**

- `GET /favorites`는 `movieId`만 돌려준다.
- 제목과 포스터는 TMDB 상세 API로 가져왔다.
- 상세 페이지와 같은 query key(`["movies", "detail", movieId]`)를 써서, 이미 본 영화는 다시 요청하지 않는다.

```tsx
// pages/me/my-page.tsx
const movieQueries = useQueries({
  queries: pageFavorites.map((favorite) => ({
    queryKey: ["movies", "detail", favorite.movieId],
    queryFn: () => getMovieDetail(favorite.movieId),
  })),
});
```

**③ 즐겨찾기 저장 위치 통일**

- 영화 목록과 검색 페이지는 로컬 `bookmark-store`를, 마이페이지는 서버 `/favorites`를 읽고 있어서 화면마다 즐겨찾기 상태가 달랐다.
- 목록, 검색, 상세, 마이페이지가 모두 서버 즐겨찾기 query key를 읽도록 바꾸고 `bookmark-store`를 삭제했다.

#### 2-4. 변경 결과 반영

| 동작 | 갱신 방식 | 이유 |
| --- | --- | --- |
| 즐겨찾기 여부 | `setQueryData` | 등록·해제 결과(`true`/`false`)를 확실히 안다 |
| 즐겨찾기 목록 | `invalidateQueries` | 목록에 필요한 `favoriteId`를 응답으로 받지 못한다 |
| 평점 등록·수정·삭제 | `invalidateQueries` | 응답에 갱신된 평점 목록이 없다 |

```tsx
// hooks/use-favorite.ts
onSuccess: (_data, { movieId, isFavorite }) => {
  queryClient.setQueryData(favoriteKeys.status(movieId), !isFavorite);
  queryClient.invalidateQueries({ queryKey: favoriteKeys.list });
},
```

```tsx
// hooks/use-ratings.ts
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: ratingKeys.byMember(memberId) });
},
```

#### 2-5. 확인 결과

| 상황 | 결과 |
| --- | --- |
| 상세에서 즐겨찾기 등록 | 마이페이지 목록에 추가됨 |
| 마이페이지에서 영화 클릭 | 상세 페이지로 이동 |
| 상세에서 즐겨찾기 해제 | 마이페이지 목록에서 빠짐 |
| 영화 목록·검색·상세·마이페이지 | 같은 즐겨찾기 상태 표시 |
| 평점 저장 후 새로고침 | 별점과 후기가 유지됨, 버튼이 "평점 수정"으로 바뀜 |
| 평점 수정 | 수정한 별점과 후기로 바뀜 |
| 평점 삭제 | 빈 상태로 돌아가고 버튼이 "평점 저장"으로 바뀜 |
| 로그인 전 상세 페이지 | 평점 패널에 "로그인해 주세요" 표시, 즐겨찾기 클릭 시 `/login` 이동 |