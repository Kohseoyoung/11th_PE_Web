- URL과 라우팅
    - URI, URL, path와 route는 각각 무엇이며 어떤 차이가 있을까요
        1. **URI (Uniform Resource Identifier)**

      리소스를 식별하는 문자열을 넓게 부르는 말이다. 주소일 수도 있고 이름일 수도 있다.

        1. **URL (Uniform Resource Locator)**

      URI 중에서 리소스에 접근할 방법과 위치를 나타내는 주소이다. 모든 URL은 URI이지만 모든 URI가 URL은 아니므로, URL은 URI의 부분집합이다. 웹 프론트엔드에서는 주로 URL을 다룬다.

        1. **path**

      URL의 구성 요소 중 리소스의 경로를 나타내는 부분이다. 브라우저의 `URL` API에서는 `pathname`이라고 부른다.

        1. **route**

      URL의 구성 요소가 아니라 앱이 정한 규칙이다. 어떤 모양의 path를 어떤 화면과 연결할지 정해 둔 것이다.

      #### 워크북 예시로 보기

      `https://umcine.example/movies/10?from=search#reviews`

      | 구분 | 값 | 설명 |
              | --- | --- | --- |
      | scheme | `https` | 통신 방법 |
      | origin | `https://umcine.example` | scheme, host와 필요한 경우 port를 묶은 값 |
      | path | `/movies/10` | 리소스의 경로 (`pathname`) |
      | search param | `from=search` | `?` 뒤의 값 |
      | fragment (hash) | `reviews` | `#` 뒤의 값, 문서 안의 특정 위치 |

        ```tsx
        const movieUrl = new URL("https://umcine.example/movies/10?from=search#reviews");
        
        movieUrl.origin;                     // https://umcine.example
        movieUrl.pathname;                   // /movies/10
        movieUrl.searchParams.get("from");   // search
        movieUrl.hash;                       // #reviews
        ```

      route는 이 URL 안에 들어 있는 값이 아니다. `umcine`에서는 파일 이름이 route 규칙이 된다.

        ```tsx
        // routes/movies.$movieId.tsx
        export const Route = createFileRoute("/movies/$movieId")({
          component: MovieDetailPage,
        });
        ```

      `/movies/$movieId`라는 route 규칙은 `/movies/10`, `/movies/2`처럼 같은 모양의 path와 `MovieDetailPage`를 연결한다. 여기서 `$movieId` 자리에 들어오는 `10`이 path param이다.

      #### 한눈에 보는 차이

      | 구분 | 무엇인가 | 어디에 있나 | 예시 |
              | --- | --- | --- | --- |
      | URI | 리소스를 식별하는 문자열 (가장 넓은 개념) | 개념 | `https://umcine.example/movies/10` |
      | URL | 접근 방법과 위치를 가진 URI | 브라우저 주소창 | `https://umcine.example/movies/10?from=search#reviews` |
      | path | URL의 경로 부분 | URL의 구성 요소 | `/movies/10` |
      | route | path와 화면을 연결하는 앱의 규칙 | 앱 코드 | `/movies/$movieId` |
        - path는 실제로 들어온 값(`/movies/10`)이고, route는 그 값을 받아들이는 패턴(`/movies/$movieId`)이다.
        - URL은 사용자가 입력하거나 링크로 만들어지는 주소이고, route는 개발자가 정해 두는 규칙이다.
        - 하나의 route가 여러 path를 처리할 수 있다.
    - path param과 search param은 각각 어떤 값을 표현할 때 사용하는 것이 좋을까요?

      ### **1. path param**

      path 안에서 달라지는 값으로, 리소스를 식별한다. 이 값이 없으면 어떤 화면인지 정해지지 않는다.

        - 예: `/movies/10`의 `10` (route 규칙은 `/movies/$movieId`)
        - 값이 필수이고, 값마다 별도의 화면(리소스)이 된다.

      ### **2. search param**

      `?` 뒤에 `key=value` 형태로 붙는 값으로, 같은 화면을 어떻게 보여 줄지 조건을 나타낸다. 여러 개일 때는 `&`로 잇는다.

        - 예: `/search?query=스파이더맨&page=2`
        - 값이 선택이고, 없어도 화면은 열린다.

      #### 3. 언제 무엇을 쓰는가

      | 구분 | path param | search param |
              | --- | --- | --- |
      | 표현하는 값 | 리소스의 정체 (무엇인가) | 화면의 조건 (어떻게 보여 줄까) |
      | 필수 여부 | 필수 | 선택 |
      | 값이 없을 때 | 해당 route와 일치하지 않음 | 기본 상태로 표시됨 |
      | 적합한 값 | 영화 ID, 사용자 ID, 게시글 번호 | 검색어, 필터, 정렬, 페이지 번호, 유입 경로 |
      | `umcine` 예시 | `/movies/$movieId` | `/search?query=...` |

      #### 4. 판단 기준

        - 이 값이 없으면 화면이 성립하지 않는가? → path param
        - 같은 화면에서 보는 방식만 달라지는가? → search param
        - 값이 바뀌면 다른 리소스인가? → path param (`/movies/1`과 `/movies/2`는 다른 영화)
        - 값이 바뀌어도 같은 목록에 대한 다른 조건인가? → search param (`query=A`와 `query=B`는 같은 검색 화면)

      #### 예시)

        ```tsx
        // path param : 영화 상세 (영화마다 별도의 화면)
        export const Route = createFileRoute("/movies/$movieId")({
          component: MovieDetailPage,
        });
        const { movieId } = useParams({ from: "/movies/$movieId" });
        
        // search param : 검색 (같은 화면에서 조건만 달라짐)
        export const Route = createFileRoute("/search")({
          validateSearch: (search): { query?: string } => ({
            query: typeof search.query === "string" ? search.query : undefined,
          }),
          component: SearchPage,
        });
        const { query } = useSearch({ from: "/search" });
        ```

    - 검색어, 필터나 현재 화면의 상태를 URL에 포함하면 공유와 새로고침에 어떤 장점이 있을까요?

      #### 장점 정리

      | 구분 | state에만 둘 때 | URL에 둘 때 |
              | --- | --- | --- |
      | 새로고침 | 상태가 초기화되어 처음 화면으로 돌아간다 | 주소가 그대로라 같은 화면이 유지된다 |
      | 링크 공유 | 링크를 열면 기본 화면만 보인다 | 같은 검색 결과와 필터가 그대로 보인다 |
      | 북마크, 즐겨찾기 | 저장해도 기본 화면이 열린다 | 특정 검색 결과를 저장할 수 있다 |
      | 뒤로 가기, 앞으로 가기 | 이전 상태로 돌아가지 못한다 | 히스토리에 기록되어 이전 검색 상태로 돌아간다 |
      | 직접 접근 | 사용자가 조건을 다시 입력해야 한다 | 주소만 입력해도 해당 상태로 진입한다 |

      #### 1. 새로고침해도 유지된다

      `/search?query=스파이더맨`에서 새로고침하면 `query` 값이 주소에 남아 있어 같은 결과가 다시 표시된다. 검색어를 `useState`에만 두었다면 새로고침 후 입력창과 결과가 비어 버린다.

      #### 2. 링크 하나로 상태를 공유할 수 있다

      주소를 복사해 전달하면 받는 사람도 같은 검색 결과를 본다. "스파이더맨으로 검색한 화면"을 설명할 필요 없이 링크만 보내면 된다.

      #### 3. 뒤로 가기와 앞으로 가기가 자연스럽게 동작한다

      검색어를 바꿔 제출할 때마다 URL이 바뀌고 브라우저 히스토리에 쌓인다. 뒤로 가기를 누르면 이전 검색어와 결과로 돌아간다. 워크북 4.2의 확인 항목(입력창과 결과가 URL의 `query`와 함께 바뀌는지)이 이 장점을 확인하는 과정이다.

      #### 4. 상태의 기준이 한 곳으로 모인다

      URL이 화면 상태의 기준이 되면 컴포넌트 안에 같은 값을 따로 복사해 둘 필요가 줄어든다. 앞에서 배운 Single Source of Truth(기준이 되는 값은 한 곳에만 둔다)와 같은 원칙이다.

      #### 예시)

        ```tsx
        // URL에서 검색어를 읽는다
        const { query } = useSearch({ from: "/search" });
        
        // 폼을 제출하면 state가 아니라 URL을 바꾼다
        navigate({
          search: nextQuery ? { query: nextQuery } : {},
        });
        ```

      화면의 결과는 `query`에서 계산하고, 입력창의 임시 값만 `useState`로 관리한다.

      #### 어떤 상태를 URL에 둘까

      | URL에 두면 좋은 상태 | URL에 두지 않는 상태 |
              | --- | --- |
      | 검색어, 필터, 정렬 | 모달이 열려 있는지 여부 |
      | 페이지 번호 | 입력 중인 임시 텍스트 |
      | 선택한 탭 | 마우스 호버 같은 순간적인 UI 상태 |
      | 보고 있는 리소스의 ID | 비밀번호, 토큰 등 민감한 정보 |

      공유하거나 새로고침한 뒤에도 같은 화면이어야 하는 상태는 URL에 두고, 일시적이거나 민감한 상태는 두지 않는다.

- SPA와 MPA
    - SPA와 MPA는 각각 무엇이며 화면을 이동하는 방식에 어떤 차이가 있을까요?

      ### **SPA (Single Page Application)**

      처음 받은 HTML 문서를 유지한 채 JavaScript가 브라우저의 주소 기록(history)과 화면을 바꾸는 앱 구조이다. 이동할 때 필요한 데이터와 컴포넌트만 바꿔서 보여 준다.

      ### **MPA (Multi Page Application)**

      다른 URL로 이동할 때마다 서버에서 새로운 HTML 문서를 받아 화면을 바꾸는 구조이다. 페이지마다 별도의 HTML 문서가 있다.

      둘의 핵심 차이는 **URL을 이동할 때 HTML 문서를 계속 사용하는지, 새로 받는지**이다.

      #### 화면 이동 방식 비교

      | 구분 | SPA | MPA |
              | --- | --- | --- |
      | 이동 방식 | JS가 URL과 화면을 바꾼다 (클라이언트 내비게이션) | 브라우저가 서버에 요청해 새 문서를 받는다 |
      | HTML 문서 | 처음 한 번만 받고 유지한다 | 이동할 때마다 새로 받는다 |
      | 화면 갱신 | 바뀌는 부분만 다시 그린다 | 페이지 전체를 다시 그린다 |
      | 새로고침 | 이동 중에는 일어나지 않는다 | 이동할 때마다 일어난다 |
      | 상태 유지 | 메모리의 state가 이동 중에도 유지된다 | 이동하면 JS 상태가 초기화된다 |
      | 첫 로딩 | JS를 먼저 받아야 해서 상대적으로 느릴 수 있다 | 필요한 페이지만 받아 첫 화면이 빠를 수 있다 |

      #### 이동 흐름

        - **MPA**

        ```
        링크 클릭 → 서버에 요청 → 새 HTML 문서 응답 → 페이지 전체를 새로 그림
        ```

        - **SPA**

        ```
        링크 클릭 → JS가 URL 변경 + 해당 route 컴포넌트 표시 → 필요한 부분만 다시 그림
        ```

      #### 예시)

      TanStack Router의 `Link`는 전체 HTML 문서를 다시 받지 않고 현재 SPA 안에서 URL과 화면을 바꾼다.

        ```tsx
        // SPA 이동 : 문서를 다시 받지 않고 화면만 바뀐다
        <Link to="/search">검색</Link>
        
        // MPA 방식의 이동 : 서버에서 새 문서를 받는다
        <a href="/search">검색</a>
        ```

      헤더의 `내 정보`는 연결할 route가 없어 일반 `<a>`로 두었는데, 클릭하면 전체 문서를 다시 받으므로 SPA 이동이 아니다.

      **확인 방법**

        1. 브라우저 개발자 도구의 Network 탭을 연다.
        2. `Link`로 `영화`와 `검색` 메뉴를 번갈아 누르면 HTML 문서 요청이 새로 생기지 않고 화면만 바뀐다.
        3. 일반 `<a>` 링크를 누르면 문서 요청이 새로 발생하고 화면이 깜빡인다.
    - SPA의 클라이언트 내비게이션은 새로운 HTML 문서를 받는 일반적인 페이지 이동과 어떻게 다를까요?

      ### SPA의 클라이언트 내비게이션과 일반 페이지 이동의 차이

        1. **일반적인 페이지 이동 (MPA, `<a href>`)**

      링크를 누르면 브라우저가 서버에 새 URL의 HTML 문서를 요청하고, 응답받은 문서로 현재 화면을 통째로 교체한다. JavaScript 실행 환경도 새로 시작된다.

        1. **클라이언트 내비게이션 (SPA, `Link`)**

      링크를 누르면 브라우저의 기본 이동을 막고, JavaScript가 주소와 화면을 직접 바꾼다. 서버에 HTML 문서를 다시 요청하지 않는다.

      #### 동작 흐름 비교

        - **일반 페이지 이동**

        ```
        링크 클릭 → 브라우저가 서버에 HTML 요청 → 새 문서 수신 → JS 다시 실행 → 화면 전체 교체
        ```

        - **클라이언트 내비게이션**

        ```
        링크 클릭 → 기본 이동 차단 → history API로 주소만 변경 → 라우터가 URL에 맞는 route 찾기 → 해당 컴포넌트만 렌더링
        ```

      클라이언트 내비게이션은 브라우저의 History API(`pushState`)로 주소창의 URL과 히스토리를 바꾸는 방식이라, 뒤로 가기와 앞으로 가기도 일반 이동처럼 동작한다.

      #### 차이점 정리

      | 구분 | 일반 페이지 이동 | 클라이언트 내비게이션 |
              | --- | --- | --- |
      | HTML 문서 요청 | 이동할 때마다 발생 | 발생하지 않음 |
      | 바뀌는 범위 | 화면 전체 | 바뀌는 route 컴포넌트만 |
      | JS 실행 환경 | 매번 초기화 | 그대로 유지 |
      | 메모리의 state | 이동하면 사라짐 | 유지될 수 있음 |
      | 화면 깜빡임 | 있음 (새 문서 로딩) | 거의 없음 |
      | 주소 변경 방식 | 브라우저가 새 문서로 이동 | JS가 history API로 변경 |
      | 공통 레이아웃 | 매번 다시 그림 | 유지 (`Header`는 그대로, `Outlet`만 교체) |

      #### 예시)

        ```tsx
        // 클라이언트 내비게이션 : 문서를 다시 받지 않는다
        <Link to="/search">검색</Link>
        
        // 일반 페이지 이동 : 서버에서 새 문서를 받는다
        <a href="/">내 정보</a>
        ```

      `__root.tsx`의 구조 덕분에 `Header`는 이동해도 그대로 유지되고, `Outlet` 자리의 컴포넌트만 목록, 검색, 상세로 교체된다.

        ```tsx
        <>
          <Header />   {/* 이동해도 유지 */}
          <Outlet />   {/* URL에 맞는 route 컴포넌트로 교체 */}
          <Footer />
        </>
        ```

      #### 직접 확인하는 방법

        1. 개발자 도구의 Network 탭을 열고 `Doc` 필터를 선택한다.
        2. 헤더의 `Link`(`영화`, `검색`)를 번갈아 누른다. 새 HTML 문서 요청이 생기지 않고, 주소만 바뀌며 화면이 전환된다.
        3. `내 정보`(`<a>`)를 누른다. 문서 요청이 새로 생기고 화면이 깜빡인다.

      #### 주의할 점

        - 클라이언트 내비게이션은 앱 안에서 링크로 이동할 때만 해당한다.
        - `/movies/1`을 주소창에 직접 입력하거나 새로고침하면 브라우저는 서버에 HTML 문서를 요청한다. 이때 서버는 어떤 URL이든 같은 `index.html`을 내려주고, 이후 라우터가 URL을 읽어 화면을 그린다.
        - 클라이언트 내비게이션이 서버 통신을 아예 하지 않는다는 뜻은 아니다. HTML 문서를 다시 받지 않을 뿐이고, 필요한 데이터는 `fetch` 같은 요청으로 따로 가져온다.
    - SPA와 MPA는 각각 어떤 서비스나 화면에 적합할까요?

      ### SPA와 MPA가 적합한 서비스와 화면

      선택 기준은 **화면 전환이 얼마나 잦은지, 상태를 이어 가야 하는지, 검색 노출과 첫 로딩이 얼마나 중요한지**이다.

      #### 1. SPA가 적합한 경우

      화면을 자주 오가고, 이동해도 상태와 레이아웃을 유지해야 하는 서비스이다.

        - 대시보드, 관리자 페이지
        - 협업 도구, 문서 편집기, 메신저
        - 지도, 음악 플레이어처럼 이동 중에도 계속 동작해야 하는 앱
        - 로그인 후 사용하는 서비스 (검색 노출이 중요하지 않은 화면)

      **이유**

        - 화면 전환이 빠르고 깜빡임이 적다.
        - 이동해도 JS 상태와 공통 레이아웃이 유지된다. (예: 음악이 끊기지 않는다.)
        - 사용자 동작에 즉시 반응하는 앱 같은 경험을 만들기 쉽다.

      **단점**

        - 첫 로딩 때 JS를 먼저 받아야 해서 초기 화면이 늦을 수 있다.
        - 처음 받은 HTML이 거의 비어 있어, 검색엔진 노출(SEO)이나 링크 미리보기에서 불리할 수 있다.

      #### 2. MPA가 적합한 경우

      페이지마다 독립적인 문서이고, 검색 노출과 첫 화면 속도가 중요한 서비스이다.

        - 블로그, 뉴스, 문서 사이트
        - 기업 소개 사이트, 랜딩 페이지
        - 쇼핑몰의 상품 상세처럼 검색으로 유입되는 페이지
        - 화면 이동이 적고 정보를 읽는 것이 중심인 서비스

      **이유**

        - 페이지마다 완성된 HTML이 내려와 첫 화면이 빠르고 SEO에 유리하다.
        - 구조가 단순하고, 페이지가 서로 독립적이라 관리하기 쉽다.
        - JS를 많이 받지 않아도 동작한다.

      **단점**

        - 이동할 때마다 전체 문서를 다시 받아 깜빡임이 생긴다.
        - 이동하면 JS 상태가 초기화되어, 화면을 넘나드는 상태를 유지하기 어렵다.

      #### 비교 정리

      | 기준 | SPA | MPA |
              | --- | --- | --- |
      | 화면 전환 속도 | 빠름 | 이동마다 문서 로딩 |
      | 첫 로딩 | 상대적으로 느릴 수 있음 | 빠른 편 |
      | SEO | 추가 대응이 필요함 | 유리함 |
      | 상태 유지 | 이동 중에도 유지 | 이동하면 초기화 |
      | 구현 복잡도 | 라우팅, 상태 관리가 필요함 | 상대적으로 단순함 |
      | 어울리는 화면 | 앱처럼 사용하는 화면 | 문서처럼 읽는 화면 |
- Tailwind CSS
    - CSS 규칙을 직접 작성하는 방식과 비교했을 때 Tailwind CSS의 특징은 무엇일까요?

      ### CSS 직접 작성 방식과 비교한 Tailwind CSS의 특징

        1. **CSS 직접 작성**

      클래스 이름을 짓고 별도의 CSS 파일에 규칙을 작성한 뒤 JSX에서 그 이름을 연결한다.

        1. **Tailwind CSS**

      미리 정해진 작은 utility class를 TSX의 `className`에 조합하여 스타일을 만드는 도구이다. 별도의 CSS 규칙을 작성하지 않고 마크업 안에서 스타일을 정한다.

      #### 예시)

        1. **직접 작성 (`App.css` + 클래스 이름)**

        ```css
        .movie-card__poster {
          position: relative;
          aspect-ratio: 7 / 8;
          border-radius: 8px;
          overflow: hidden;
        }
        ```

        ```tsx
        <div className="movie-card__poster">
        ```

        1. **Tailwind (utility class 조합)**

        ```tsx
        <div className="relative aspect-[7/8] overflow-hidden rounded-lg">
        ```

      같은 스타일이지만 CSS 파일과 클래스 이름이 따로 필요 없다.

      #### Tailwind의 특징

      **1. utility-first**

      `relative`, `rounded-lg`, `text-sm`처럼 한 가지 역할만 하는 작은 class를 조합한다. `rounded-[10px]`처럼 대괄호로 디자인의 정확한 수치도 작성할 수 있다.

      **2. 클래스 이름을 짓지 않는다**

      `movie-card__poster` 같은 이름을 고민하거나 BEM 규칙을 관리할 필요가 없다.

      **3. 스타일이 마크업 옆에 있다**

      컴포넌트 파일 하나에서 구조와 스타일을 함께 본다. CSS 파일과 TSX 파일을 오가지 않아도 된다.

      **4. 사용한 class만 CSS로 생성한다**

      빌드할 때 소스에 적힌 class 이름을 찾아 필요한 CSS만 만든다. 안 쓰는 규칙이 쌓이지 않아 최종 CSS가 작다.

      **5. 변형자(prefix)로 상태와 반응형을 표현한다**

        ```tsx
        <ul className="grid grid-cols-1 gap-5 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-5">
        ```

      `sm:`, `lg:`, `xl:`은 해당 breakpoint 이상에서만 적용되고, `hover:`, `disabled:`, `[&.active]:` 같은 변형자로 상태에 따른 스타일도 쓴다.

      **6. 디자인 값이 일관된다**

      간격, 색, 글자 크기가 정해진 단계(`p-2`, `gap-5`, `text-sm`, `bg-blue-600`)로 제공되어 화면 전체의 값이 통일된다.

      **7. 전역 충돌이 적다**

      CSS는 전역 범위라 클래스 이름이 겹치거나 `.main` 같은 규칙을 어디서 쓰는지 찾기 어려웠다. utility class는 요소에 직접 붙으므로 삭제와 수정이 안전하다. 실제로 `App.css`를 지울 때 클래스 이름 검색으로 사용처를 확인해야 했다.

      #### 비교 정리

      | 구분 | CSS 직접 작성 | Tailwind CSS |
              | --- | --- | --- |
      | 스타일 위치 | 별도 CSS 파일 | TSX의 `className` |
      | 클래스 이름 | 직접 짓고 관리 | 미리 정해진 이름 |
      | 생성되는 CSS | 작성한 규칙 전부 | 사용한 class만 |
      | 반응형, 상태 | `@media`, 선택자 작성 | `sm:`, `hover:` 같은 prefix |
      | 디자인 값 | 값을 각자 입력 | 정해진 단계 제공 |
      | 사용처 추적 | 클래스 이름을 검색해야 함 | 요소에 바로 보임 |
    - styled-components와 Tailwind CSS는 스타일을 작성하고 생성하는 방식이 어떻게 다를까요?

      ### styled-components와 Tailwind CSS의 스타일 작성 및 생성 방식 차이

        1. **styled-components (CSS-in-JS)**

      JavaScript 안에서 템플릿 리터럴로 CSS를 작성해 스타일이 적용된 컴포넌트를 만든다. 스타일이 컴포넌트에 붙어 있다.

        1. **Tailwind CSS (utility-first)**

      미리 정해진 작은 utility class를 `className`에 조합하여 스타일을 만든다. 스타일 규칙은 Tailwind가 제공하고, 개발자는 class 이름을 고른다.

      #### 작성 방식 비교

      같은 북마크 버튼을 두 방식으로 작성하면 다음과 같다.

        - **styled-components**

        ```tsx
        import styled from "styled-components";
        
        const BookmarkButton = styled.button<{ $active: boolean }>`
          position: absolute;
          top: 8px;
          right: 8px;
          width: 32px;
          height: 32px;
          border-radius: 6px;
          background: ${({ $active }) => ($active ? "#2563eb" : "rgba(17, 24, 39, 0.6)")};
        `;
        
        <BookmarkButton $active={isBookmarked} />
        ```

        - **Tailwind CSS**

        ```tsx
        <button
          className={cn(
            "absolute right-2 top-2 size-8 rounded-md",
            isBookmarked ? "bg-blue-600" : "bg-gray-900/60",
          )}
        />
        ```

      #### 스타일이 생성되는 방식

      | 구분 | styled-components | Tailwind CSS |
              | --- | --- | --- |
      | 생성 시점 | 주로 런타임 (브라우저에서 실행될 때) | 빌드 시점 |
      | 생성 과정 | 컴포넌트가 렌더링될 때 CSS를 만들고 `<style>` 태그에 삽입 | 소스에 적힌 class 이름을 스캔해 CSS 파일 생성 |
      | class 이름 | 라이브러리가 해시 이름을 자동 생성 (예: `sc-a1b2c`) | 미리 정해진 이름을 그대로 사용 (예: `rounded-md`) |
      | 결과물 | JS 번들에 CSS 문자열이 포함됨 | 정적 CSS 파일 |

      #### 작성 방식의 차이

      **1. 스타일을 어디에 쓰는가**

        - styled-components: 컴포넌트 선언부에 CSS 문법 그대로 작성한다. 스타일이 입혀진 컴포넌트에 이름을 붙이는 구조이다.
        - Tailwind: JSX의 `className`에 미리 정해진 class를 나열한다. 이름 붙이기와 별도 선언이 필요 없다.

      **2. 동적 스타일을 어떻게 만드는가**

        - styled-components: props 값을 받아 JS 코드로 스타일을 계산한다. 값의 제약이 거의 없다.
        - Tailwind: 조건마다 완성된 class 이름을 작성하고 `cn`으로 조합한다. 값이 조건에 따라 정해진 단계 중 하나여야 한다.

      **3. 클래스 이름 관리**

        - styled-components: 해시 class가 자동 생성되어 이름 충돌이 없다. 컴포넌트에 스타일이 묶여 있다.
        - Tailwind: 이름을 짓지 않아도 된다. 대신 class를 조합하는 규칙을 익혀야 한다.

      #### 장단점 정리

      | 구분 | styled-components | Tailwind CSS |
              | --- | --- | --- |
      | 장점 | 컴포넌트와 스타일이 한 단위로 묶임, props 기반 동적 스타일이 자유로움 | 런타임 비용이 없음, 최종 CSS가 작음, 디자인 값이 일관됨 |
      | 단점 | 런타임에 스타일을 만들어 성능 부담이 생길 수 있음, 서버 렌더링과의 설정이 필요할 수 있음 | `className`이 길어짐, class 이름을 익혀야 함 |
      | 스타일 재사용 | 컴포넌트로 재사용 | class 조합을 변수나 컴포넌트로 재사용 |
    - Tailwind CSS에서 `bg-${color}-500`처럼 class 이름을 동적으로 조합하면 스타일이 생성되지 않을 수 있는 이유는 무엇일까요?

      ### Tailwind CSS에서 `bg-${color}-500`처럼 class 이름을 동적으로 조합하면 스타일이 생성되지 않을 수 있는 이유

        - **핵심 이유**

      Tailwind는 코드를 실행해서 class를 알아내는 것이 아니라, **빌드할 때 소스 파일의 텍스트를 읽어 완성된 class 이름을 찾고** 그 class의 CSS만 생성한다. 문자열을 조립해서 만드는 class는 소스에 완성된 형태로 적혀 있지 않으므로 Tailwind가 찾지 못한다.

      #### Tailwind가 CSS를 만드는 과정

        ```
        1. 빌드 시점에 소스 파일(.tsx 등)을 문자열로 스캔한다.
        2. 스캔한 텍스트 중 Tailwind class와 일치하는 이름을 찾는다.
        3. 찾은 class에 해당하는 CSS 규칙만 생성한다.
        ```

      이 스캔은 JavaScript를 실행하지 않는 단순한 텍스트 검색이다. 변수 `color`에 어떤 값이 들어갈지 알 수 없다.

      #### 동적 조합이 실패하는 예

        ```tsx
        // 잘못된 방식 : 완성된 class 이름이 소스에 없다
        const color = isBookmarked ? "blue" : "gray";
        <button className={`bg-${color}-500`} />
        ```

        - 소스에 적혀 있는 텍스트는 `bg-${color}-500`이다.
        - Tailwind는 `bg-blue-500`이나 `bg-gray-500`이라는 문자열을 한 번도 보지 못한다.
        - 그래서 해당 CSS 규칙이 생성되지 않고, 브라우저에서 `bg-blue-500` class가 붙어도 적용할 스타일이 없다.

      #### 올바른 방식 : 조건마다 완성된 class 이름을 작성

        ```tsx
        // 완성된 이름이 소스에 그대로 적혀 있다
        <button className={isBookmarked ? "bg-blue-500" : "bg-gray-500"} />
        ```

        - 두 class 이름이 소스에 완전한 문자열로 존재하므로 스캔에서 발견된다.
        - 두 규칙이 모두 CSS에 생성되고, 조건에 따라 알맞은 것이 적용된다.

      #### 여러 상태가 있을 때 : 완성된 이름의 매핑 객체

        ```tsx
        const colorClass = {
          blue: "bg-blue-500",
          gray: "bg-gray-500",
          red: "bg-red-500",
        };
        
        <button className={colorClass[color]} />
        ```

        - 객체 값에 완성된 class 이름이 모두 적혀 있어 스캔에서 발견된다.
        - 값에 따라 알맞은 class를 고르는 로직은 JS에서, class 이름 자체는 소스에 전부 적어 둔다.
        - umcine의 북마크 버튼처럼 `cn`으로 조합할 때도 같은 원칙을 지켰다.

        ```tsx
        className={cn(
          "absolute right-2 top-2 size-8 rounded-md",
          isBookmarked ? "bg-blue-600" : "bg-gray-900/60",
        )}
        ```

      #### 값이 정말 동적일 때 : CSS 변수 활용

      색상이 사용자 입력이나 서버 데이터처럼 미리 정해지지 않는 경우에는 class 이름을 조립하지 않고 CSS 변수를 사용한다.

        ```tsx
        <div
          className="bg-(--card-color)"
          style={{ "--card-color": color } as React.CSSProperties}
        />
        ```

        - class 이름은 `bg-(--card-color)`로 고정되어 소스에 적혀 있다.
        - 실제 색상 값은 런타임에 `style`로 전달하므로 빌드 시점에 알 필요가 없다.

      #### 비교 정리

      | 방식 | 예시 | 결과 |
              | --- | --- | --- |
      | 문자열 조합 | ``bg-${color}-500`` | 완성된 이름이 소스에 없어 CSS가 생성되지 않음 |
      | 조건마다 완성된 이름 | `cond ? "bg-blue-500" : "bg-gray-500"` | 정상 |
      | 완성된 이름의 매핑 객체 | `{ blue: "bg-blue-500" }[color]` | 정상 |
      | CSS 변수 | `bg-(--card-color)` + `style` | 정상 (값이 런타임에 결정될 때) |
    - 상태에 따라 달라지는 스타일은 완성된 class 이름의 매핑이나 CSS 변수를 이용해 어떻게 표현할 수 있을까요?

      ### 상태에 따라 달라지는 스타일을 표현하는 방법

      **원칙**

      Tailwind는 빌드할 때 소스의 텍스트에서 완성된 class 이름을 찾아 CSS를 생성한다. 그래서 class 이름을 문자열로 조립하지 않고, 상태에 따라 **완성된 이름을 선택**하거나, 값만 바뀌는 부분을 **CSS 변수**로 넘긴다.

      #### 1. 조건식으로 완성된 class 선택

      상태가 두 가지일 때 사용한다. 두 class 이름이 모두 소스에 완성된 형태로 적혀 있다.

        ```tsx
        <button
          className={cn(
            "absolute right-2 top-2 grid size-8 place-items-center rounded-md",
            isBookmarked ? "bg-blue-600" : "bg-gray-900/60",
          )}
        >
        ```

        - 항상 적용되는 공통 class와 상태에 따라 달라지는 class를 구분해서 작성한다.
        - `cn`(`clsx` + `tailwind-merge`)이 조건에 맞는 class만 이어 붙이고, 충돌하는 class는 뒤의 값을 남긴다.

      #### 2. 완성된 class 이름의 매핑 객체

      상태가 세 가지 이상일 때 사용한다. 상태 값을 key로, 완성된 class를 value로 둔다.

        ```tsx
        const statusClass = {
          idle: "bg-gray-100 text-gray-500",
          active: "bg-blue-600 text-white",
          error: "bg-red-100 text-red-600",
        } as const;
        
        type Status = keyof typeof statusClass;
        
        function Badge({ status }: { status: Status }) {
          return <span className={cn("rounded-md px-2 py-1 text-xs", statusClass[status])} />;
        }
        ```

        - 모든 class 이름이 객체 안에 완성된 문자열로 적혀 있어 Tailwind가 전부 찾는다.
        - `keyof typeof`로 key 타입을 뽑으면 존재하지 않는 상태를 넘길 때 타입 오류가 난다.
        - 상태가 늘어나면 객체에 항목만 추가하면 된다.

      #### 3. 상태 속성에 반응하는 변형자 사용

      요소가 이미 가진 속성(`aria-pressed`, `disabled`, `data-*`)에 맞춰 스타일을 정한다. 조건식 없이 마크업의 상태가 곧 스타일의 기준이 된다.

        ```tsx
        <button
          aria-pressed={isBookmarked}
          className="bg-gray-900/60 aria-pressed:bg-blue-600"
        />
        ```

        - `aria-pressed={true}`일 때만 `bg-blue-600`이 적용된다.
        - 접근성 속성과 스타일 상태가 한곳에서 일치한다.
        - 헤더의 활성 메뉴도 같은 방식이다. `Link`가 붙이는 `active` class에 `[&.active]:font-bold`로 반응한다.

      #### 4. CSS 변수

      값이 미리 정해져 있지 않고 런타임에 결정될 때 사용한다. class 이름은 고정하고 값만 `style`로 전달한다.

        ```tsx
        <div
          className="bg-(--card-color)"
          style={{ "--card-color": color } as React.CSSProperties}
        />
        ```

        - `bg-(--card-color)`라는 완성된 이름이 소스에 있으므로 CSS가 생성된다.
        - 실제 색상 값은 렌더링할 때 `style`로 들어가므로 빌드 시점에 알 필요가 없다.
        - 사용자가 고른 색, 서버에서 받은 값, 진행률(퍼센트) 같은 연속적인 값에 적합하다.

      #### 선택 기준

      | 상황 | 방법 | 예시 |
              | --- | --- | --- |
      | 상태가 2가지 | 조건식 + `cn` | `isBookmarked ? "bg-blue-600" : "bg-gray-900/60"` |
      | 상태가 3가지 이상 | 완성된 class의 매핑 객체 | `statusClass[status]` |
      | 요소의 속성으로 상태가 드러남 | 속성 변형자 | `aria-pressed:bg-blue-600`, `disabled:opacity-30` |
      | 값이 런타임에 결정됨 | CSS 변수 | `bg-(--card-color)` + `style` |

      #### 하면 안 되는 방식

        ```tsx
        // class 이름을 문자열로 조립하면 완성된 이름이 소스에 없어 CSS가 생성되지 않는다
        <button className={`bg-${color}-600`} />
        ```