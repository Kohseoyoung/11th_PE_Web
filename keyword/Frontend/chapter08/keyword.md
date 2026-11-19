- 클라이언트 상태와 서버 상태
    - 입력값, 로그인 토큰과 회원 정보는 각각 어디에서 관리하는 것이 적절할까요? 각 상태의 원본이 어디에 있는지도 함께 생각해 보세요.

      ### 입력값, 로그인 토큰, 회원 정보는 각각 어디에서 관리해야 할까?

      #### 1. 나누는 기준: 원본이 어디에 있는가

      **서버 상태의 특징** (TanStack Query 공식 문서)

      | 특징 | 설명 |
              | --- | --- |
      | 원격 저장 | 내가 직접 통제하거나 소유하지 못하는 곳(서버)에 저장된다 |
      | 비동기 접근 | 가져오거나 바꾸려면 비동기 API가 필요하다 |
      | 공유 소유 | 다른 사람이 내가 모르는 사이에 바꿀 수 있다 |
      | 오래될 수 있음 | 관리하지 않으면 화면의 값이 서버와 달라진다 |

      **TanStack Query의 역할**

        - 서버 상태를 관리하는 라이브러리이며, 클라이언트 상태 관리를 대체하지 않는다.
        - Zustand 같은 클라이언트 상태 관리 도구와 함께 쓸 수 있다.

      **React의 관점** (React 공식 문서)

        - 상태마다 사는 위치가 다르다.
        - 입력값처럼 말단 컴포넌트 가까이에 두는 상태가 있고, 앱 최상단 가까이에 두는 상태가 있다.

      **정리**

      | 구분 | 원본 위치 | 특징 | 관리 도구 |
              | --- | --- | --- | --- |
      | 클라이언트 상태 | 브라우저 | 동기적, 내가 완전히 통제 | `useState`, Zustand |
      | 서버 상태 | 서버 DB | 비동기적, 다른 곳에서 바뀔 수 있음 | TanStack Query |

      #### 2. 입력값 → `useState`

      | 항목 | 내용 |
              | --- | --- |
      | 원본 | 사용자가 입력 중인 화면 (제출 전까지 서버에 없음) |
      | 사용 범위 | 한 컴포넌트 안 |
      | 유지 기간 | 페이지를 벗어나면 사라져도 됨 |

        ```tsx
        // login-page.tsx: 로그인 폼 입력값
        const [form, setForm] = useState({ email: "", password: "" });
        ```

      **수정 폼은 입력 전후로 원본이 바뀐다**

        - 입력 전: 서버에서 받은 닉네임을 보여 준다 (서버 상태).
        - 입력 후: 사용자가 쓴 값을 보여 준다 (클라이언트 상태).

        ```tsx
        // my-edit-page.tsx
        const [nicknameInput, setNicknameInput] = useState<string | null>(null);
        const nickname = nicknameInput ?? meQuery.data?.nickname ?? "";
        ```

      #### 3. 로그인 토큰 → Zustand + `persist`

      | 항목 | 내용 |
              | --- | --- |
      | 원본 | 발급은 서버가 하지만, 발급 이후에는 브라우저가 보관 (다시 조회하는 API가 없음) |
      | 사용 범위 | 헤더, API 요청, 라우트 접근 제한 등 여러 곳 |
      | 유지 기간 | 새로고침해도 유지되어야 함 → `persist`로 `localStorage`에 저장 |

        ```tsx
        // auth-store.ts
        export const useAuthStore = create<AuthStore>()(
          persist(
            (set) => ({
              accessToken: null,
              setAccessToken: (accessToken) => set({ accessToken }),
            }),
            { name: "umcine-auth-store", storage: createJSONStorage(() => localStorage) },
          ),
        );
        ```

      **컴포넌트 밖에서도 `getState()`로 읽는다**

        ```tsx
        // auth-api.ts: 요청 전에 토큰을 헤더에 넣음
        const accessToken = useAuthStore.getState().accessToken;
        
        // routes/_auth.tsx: 토큰이 없으면 로그인 페이지로
        if (!useAuthStore.getState().accessToken) throw redirect({ to: "/login" });
        ```

      #### 4. 회원 정보 → TanStack Query

      | 항목 | 내용 |
              | --- | --- |
      | 원본 | 서버 DB |
      | 특징 | 다른 기기에서 닉네임을 바꾸면 내 화면의 값이 오래된 값이 됨 |
      | 필요한 기능 | 캐싱, 중복 요청 제거, 갱신 시점 관리 |

      **같은 query key를 쓰는 화면은 캐시를 공유한다**

        ```tsx
        // header.tsx, my-page.tsx, my-edit-page.tsx
        const meQuery = useQuery({
          queryKey: ["members", "me"],
          queryFn: getMe,
          enabled: Boolean(accessToken),
          staleTime: 5 * 60 * 1000,
        });
        ```

      **수정에 성공하면 응답으로 캐시를 바꿔 모든 화면에 반영한다**

        ```tsx
        // use-update-me-mutation.ts
        onSuccess: (response) => {
          queryClient.setQueryData(["members", "me"], response.result);
        },
        ```

      #### 5. 전체 정리

      | 상태 | 원본 | 관리 위치 | 이번 주 코드 |
              | --- | --- | --- | --- |
      | 입력값 | 사용자 화면 | `useState` | 로그인·회원가입·수정 폼 |
      | 로그인 토큰 | 브라우저 (발급 후) | Zustand + `persist` | `auth-store.ts` |
      | 회원 정보 | 서버 DB | TanStack Query | `["members", "me"]` |
      | 즐겨찾기·평점 | 서버 DB | TanStack Query | `["favorites", ...]`, `["ratings", ...]` |

      #### 6. 파생 값은 따로 저장하지 않는다

        - memberId는 토큰 안에 들어 있으므로, store에 따로 저장하지 않고 토큰에서 계산한다.
        - 같은 정보를 두 곳에 두면 원본이 둘이 되어 값이 어긋날 수 있다.

        ```tsx
        // my-rating-panel.tsx
        const accessToken = useAuthStore((state) => state.accessToken);
        const memberId = getMemberIdFromToken(accessToken);
        ```

    - 같은 회원 정보를 전역 store와 query 캐시에 각각 저장하면 어떤 문제가 생길 수 있을까요?

      ### 같은 회원 정보를 전역 store와 query 캐시에 각각 저장하면 어떤 문제가 생길까?

      #### 1. 공식 문서의 관점

      **React 공식 문서: State 중복을 피하라**

        - 같은 데이터가 여러 state 변수나 중첩 객체에 중복되면 서로 맞춰(동기화) 두기 어렵다.
        - 그래서 중복은 가능한 한 줄여야 한다.
        - 여러 상태가 서로 "다른 말을 하는" 구조는 실수할 여지를 만든다.

      **TanStack Query 공식 문서: 서버 데이터는 서버 상태 도구로**

        - TanStack Query는 서버와 클라이언트 사이의 비동기 작업을 관리하는 서버 상태 라이브러리다.
        - Zustand 같은 클라이언트 상태 라이브러리에도 비동기 데이터를 저장할 수는 있지만, TanStack Query에 비하면 비효율적이다.

      #### 2. 생길 수 있는 문제

      | 문제 | 설명 |
              | --- | --- |
      | 값 불일치 | 한쪽만 갱신되면 화면마다 다른 닉네임이 보인다 |
      | 갱신 코드 중복 | 바꿀 때마다 두 곳을 모두 갱신해야 하고, 하나라도 빠뜨리면 버그가 된다 |
      | 캐시 기능 사용 불가 | store에는 `staleTime`, 자동 재조회, 무효화가 없어서 store 쪽 값은 계속 오래된 채로 남는다 |
      | 이전 계정 정보 잔존 | store를 `persist`로 저장하면, 로그아웃하거나 다른 계정으로 로그인해도 이전 회원 정보가 남을 수 있다 |
      | 원본이 모호함 | 어느 쪽이 진짜 값인지 알 수 없어 디버깅이 어렵다 |

      #### 3. 예시: 둘 다 저장한 경우 (잘못된 구조)

        ```tsx
        // ❌ 회원 정보를 store에도 저장
        interface AuthStore {
          accessToken: string | null;
          member: MemberInfo | null; // query 캐시와 중복
        }
        
        // use-update-me-mutation.ts
        onSuccess: (response) => {
          queryClient.setQueryData(["members", "me"], response.result);
          // store 갱신을 빠뜨림 → 헤더(store)는 옛 닉네임, 마이페이지(query)는 새 닉네임
        },
        ```

      **결과**

      | 화면 | 읽는 곳 | 보이는 닉네임 |
              | --- | --- | --- |
      | 헤더 | store | 영화광 (옛 값) |
      | 마이페이지 | query 캐시 | 유리 (새 값) |

      `invalidateQueries`로 다시 조회해도 query 캐시만 새 값이 되고, store는 그대로다.

      #### 4. 이번 주 코드의 구조

      **store에는 토큰만, 회원 정보는 query 캐시에만 둔다**

        ```tsx
        // auth-store.ts: 클라이언트 상태(토큰)만 저장
        interface AuthStore {
          accessToken: string | null;
          setAccessToken: (accessToken: string | null) => void;
        }
        ```

        ```tsx
        // header.tsx, my-page.tsx, my-edit-page.tsx: 모두 같은 key를 읽음
        const meQuery = useQuery({
          queryKey: ["members", "me"],
          queryFn: getMe,
          enabled: Boolean(accessToken),
        });
        ```

        ```tsx
        // use-update-me-mutation.ts: 한 곳만 갱신하면 모든 화면에 반영
        onSuccess: (response) => {
          queryClient.setQueryData(["members", "me"], response.result);
        },
        ```

      | 항목 | 저장 위치 | 이유 |
              | --- | --- | --- |
      | 토큰 | Zustand store | 원본이 브라우저 (클라이언트 상태) |
      | 회원 정보 | query 캐시 | 원본이 서버 DB (서버 상태) |
- 캐시 일관성
    - 서버에서 데이터를 수정했는데 다른 화면에는 이전 값이 보이는 이유는 무엇인가요?

      ### 서버에서 데이터를 수정했는데 다른 화면에는 이전 값이 보이는 이유는?

      #### 1. 근본 원인: 화면은 서버가 아니라 캐시를 보고 있다

        - 화면은 서버 원본을 직접 보지 않고, 한 번 받아 온 **사본(캐시)**을 보여 준다.
        - 서버의 원본이 바뀌어도 캐시는 그 사실을 스스로 알지 못한다.
        - 서버 상태는 소유권이 공유되어 다른 사람이나 다른 화면이 모르는 사이에 바뀔 수 있고, 관리하지 않으면 오래된 값이 된다. (TanStack Query 공식 문서)

      #### 2. 이전 값이 남는 경우

      | 원인 | 설명 |
              | --- | --- |
      | `staleTime` 이내 | 데이터가 fresh로 취급되는 동안은 다시 요청하지 않고 캐시를 그대로 읽는다 |
      | 재조회 시점이 아님 | stale 데이터도 컴포넌트가 새로 마운트되거나, 창에 다시 포커스되거나, 네트워크가 다시 연결될 때만 자동으로 다시 조회한다 |
      | 다른 query key | 수정한 key와 다른 화면이 쓰는 key가 다르면, 한쪽만 갱신된다 |
      | 중복 저장 | 같은 데이터를 store와 query 캐시에 따로 두면, 한쪽만 바뀐다 |

      **`staleTime`의 의미** (TanStack Query 공식 문서)

        - `staleTime`이 지나기 전까지는 fresh로 취급한다.
        - 예를 들어 2분으로 설정하면, 2분 동안 또는 수동으로 무효화하기 전까지 재요청 없이 캐시만 읽는다.

      #### 3. 이번 주에 실제로 겪은 사례

      **① 로그인 후 내 정보**

        ```tsx
        // header.tsx
        const meQuery = useQuery({
          queryKey: ["members", "me"],
          queryFn: getMe,
          staleTime: 5 * 60 * 1000, // 5분 동안 재요청 안 함
        });
        ```

        - 다른 계정으로 로그인해도 5분 동안은 이전 계정의 정보가 캐시에 남을 수 있다.
        - 그래서 로그인 성공 시 직접 무효화한다.

        ```tsx
        // use-login-mutation.ts
        queryClient.invalidateQueries({ queryKey: ["members", "me"] });
        ```

      **② 즐겨찾기: 화면마다 다른 key**

      | 화면 | query key |
              | --- | --- |
      | 영화 상세 | `["favorites", "status", movieId]` |
      | 영화 목록, 검색, 마이페이지 | `["favorites", "list"]` |
        - 상세 페이지에서 즐겨찾기를 바꿔도, status key만 갱신하면 마이페이지 목록은 그대로다.
        - 그래서 두 key를 각각 처리한다.

        ```tsx
        // use-favorite.ts
        onSuccess: (_data, { movieId, isFavorite }) => {
          queryClient.setQueryData(favoriteKeys.status(movieId), !isFavorite);
          queryClient.invalidateQueries({ queryKey: favoriteKeys.list });
        },
        ```

      **③ 북마크: 저장 위치가 두 곳**

        - 영화 목록은 로컬 `bookmark-store`를, 마이페이지는 서버 `/favorites`를 읽고 있었다.
        - 그래서 목록에는 북마크 5개, 마이페이지에는 1개가 보였다.
        - 목록도 서버 즐겨찾기 key를 읽게 바꿔서 해결했다.

      #### 4. 해결 방법

      | 방법 | 동작 | 사용 시점 |
              | --- | --- | --- |
      | `invalidateQueries` | 캐시를 stale로 표시하고, 화면에서 쓰는 중이면 다시 조회한다 | 응답에 바뀐 결과가 없을 때 |
      | `setQueryData` | 캐시 값을 직접 바꾼다 (추가 요청 없음) | 응답이나 요청 값으로 결과를 알 때 |
      | 같은 query key 사용 | 여러 화면이 하나의 캐시를 공유한다 | 같은 데이터를 여러 화면에서 쓸 때 |
      | 한 곳에만 저장 | store와 query 캐시에 중복 저장하지 않는다 | 항상 |

      **`invalidateQueries`의 동작** (TanStack Query 공식 문서)

        - 캐시를 stale로 표시하며, 이 표시는 `staleTime` 설정보다 우선한다.
        - 그 query를 쓰는 컴포넌트가 화면에 있으면 백그라운드에서 다시 조회한다.
    - 변경한 데이터를 다시 조회하는 방식과 캐시를 직접 수정하는 방식은 각각 어떤 상황에 적절할까요?

      ### 변경한 데이터를 다시 조회하는 방식과 캐시를 직접 수정하는 방식은 언제 써야 할까?

      #### 1. 두 방식 비교

      | 구분 | 다시 조회 (`invalidateQueries`) | 직접 수정 (`setQueryData`) |
              | --- | --- | --- |
      | 동작 | 캐시를 stale로 표시하고 서버에서 다시 받아 온다 | 캐시 값을 바로 바꾼다 |
      | 추가 요청 | 있음 (GET) | 없음 |
      | 화면 반영 | 응답이 올 때까지 기다린다 | 즉시 |
      | 정확성 | 서버 원본 그대로라 확실하다 | 내가 만든 값이 서버와 다를 수 있다 |
      | 필요한 정보 | 어떤 query key가 영향을 받는지만 알면 된다 | 바뀐 결과를 정확히 알아야 한다 |

      #### 2. 다시 조회가 적절한 상황

      | 상황 | 이유 |
              | --- | --- |
      | 응답에 바뀐 결과가 없을 때 | 캐시에 넣을 값이 없다 |
      | 서버가 값을 계산하거나 변환할 때 | ID 생성, 정렬, 평균 등은 클라이언트가 알 수 없다 |
      | 여러 query에 영향을 줄 때 | 목록, 개수 등을 하나하나 맞추기 어렵다 |
      | 정확성이 더 중요할 때 | 서버 원본을 다시 받는 것이 가장 확실하다 |

      **이번 주 코드 ① 평점 등록·수정·삭제**

        - 응답에 갱신된 평점 목록이 없으므로 목록을 다시 조회한다.

        ```tsx
        // use-ratings.ts
        onSuccess: () => {
          queryClient.invalidateQueries({ queryKey: ratingKeys.byMember(memberId) });
        },
        ```

      **이번 주 코드 ② 로그인 후 내 정보**

        - 로그인 응답에는 닉네임만 있고 이메일, 가입 시각이 없으므로 다시 조회한다.

        ```tsx
        // use-login-mutation.ts
        queryClient.invalidateQueries({ queryKey: ["members", "me"] });
        ```

      #### 3. 직접 수정이 적절한 상황

      | 상황 | 이유 |
              | --- | --- |
      | 응답에 수정된 전체 데이터가 있을 때 | 이미 받은 값으로 다시 요청하는 건 낭비다 |
      | 결과를 확실히 알 때 | 켜기/끄기처럼 결과가 정해져 있다 |
      | 즉시 반영이 중요할 때 | 기다리지 않고 바로 화면이 바뀐다 |

      **이번 주 코드 ① 내 정보 수정 (응답 데이터 사용)**

        - `PATCH /members/me` 응답에 조회와 같은 형태의 회원 정보가 담겨 있다.

        ```tsx
        // use-update-me-mutation.ts
        onSuccess: (response) => {
          queryClient.setQueryData<MemberInfo>(["members", "me"], response.result);
        },
        ```

      **이번 주 코드 ② 즐겨찾기 여부 (결과를 확실히 앎)**

        - 등록에 성공하면 `true`, 해제에 성공하면 `false`로 결과가 정해져 있다.

        ```tsx
        // use-favorite.ts
        queryClient.setQueryData(favoriteKeys.status(movieId), !isFavorite);
        ```

      **요청에 보낸 값으로 수정하는 경우**

        - 기존 캐시를 복사한 뒤 바뀐 부분만 덮어쓴다.
        - 캐시 객체를 직접 고치면 안 되고, 새 객체를 만들어야 한다 (불변 업데이트).

        ```tsx
        queryClient.setQueryData<MemberInfo>(["members", "me"], (oldData) =>
          oldData ? { ...oldData, nickname: variables.nickname } : oldData,
        );
        ```

        - 서버가 값을 변환하거나 다른 필드도 함께 바꾼다면 이 방식은 쓰면 안 된다.

      #### 4. 두 방식을 함께 쓰는 경우

      **즐겨찾기 토글**

      | 대상 | 방식 | 이유 |
              | --- | --- | --- |
      | 상세 페이지의 즐겨찾기 여부 | `setQueryData` | 결과(`true`/`false`)를 확실히 안다 |
      | 마이페이지의 즐겨찾기 목록 | `invalidateQueries` | 목록에 필요한 `favoriteId`를 응답으로 받지 못한다 |

        ```tsx
        // use-favorite.ts
        onSuccess: (_data, { movieId, isFavorite }) => {
          queryClient.setQueryData(favoriteKeys.status(movieId), !isFavorite);
          queryClient.invalidateQueries({ queryKey: favoriteKeys.list });
        },
        ```

      #### 5. 선택 기준

      | 질문 | 예 | 아니오 |
              | --- | --- | --- |
      | 응답에 수정된 전체 데이터가 있는가? | `setQueryData` (응답 값) | 다음 질문으로 |
      | 요청 값만으로 결과를 정확히 알 수 있는가? | `setQueryData` (요청 값) | `invalidateQueries` |
        - 확실하지 않으면 `invalidateQueries`를 쓴다. 요청이 한 번 늘 뿐 화면이 틀리지는 않는다.

      #### 6. Network 탭에서 확인한 차이

      | 동작 | 요청 순서 |
              | --- | --- |
      | 즐겨찾기 등록 (`setQueryData`) | `POST /favorites` 뒤에 `GET /favorites/{movieId}`가 없다 |
      | 평점 저장 (`invalidateQueries`) | `POST /ratings` 뒤에 `GET /members/5/ratings`가 다시 나간다 |