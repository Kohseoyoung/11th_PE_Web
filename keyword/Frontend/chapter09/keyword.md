- 프레임워크의 역할
    - Next.js는 TanStack Router에 더해 어떤 기능을 제공하나요?

      #### 1. TanStack Router란?

        - URL에 맞는 화면을 연결하고 페이지 이동을 관리하는 **React용 라우팅 라이브러리**다.

      #### 2. Next.js란?

        - **풀스택 웹 앱을 만드는 React 프레임워크**다.
        - UI는 React 컴포넌트로 만들고, 라우팅·렌더링·최적화 같은 나머지 기능은 Next.js가 제공한다.
        - 번들러와 컴파일러 같은 하위 도구 설정도 자동으로 해 준다.

      #### 3. Next.js가 추가로 제공하는 기능

      | 기능 | 내용 |
              | --- | --- |
      | Server / Client Component | 컴포넌트마다 서버에서 실행할지 브라우저에서 실행할지 나눈다 |
      | 서버 렌더링 (SSR, SSG, PPR) | 요청 때 또는 빌드 때 서버에서 HTML을 미리 만든다 |
      | 서버 데이터 조회 | Server Component에서 `await fetch`로 데이터를 바로 조회한다 |
      | 캐싱과 재검증 | `"use cache"`와 revalidate로 데이터와 화면을 캐싱하고 갱신한다 |
      | Server Functions | 폼 제출 같은 데이터 변경을 서버 함수로 처리한다 |
      | Route Handlers | 같은 프로젝트 안에 API 엔드포인트를 만든다 |
      | Proxy | 요청 처리 전에 리다이렉트, 헤더 수정 등을 한다 |
      | 이미지 최적화 | `next/image`로 리사이즈, webp 변환, lazy loading을 한다 |
      | 폰트 최적화 | `next/font`로 폰트를 앱에서 직접 불러와 로딩을 최적화한다 |
      | 메타데이터 / OG 이미지 | `metadata` export로 title, description, OG 태그를 만든다 |
      | 빌드 설정 | 번들러와 컴파일러를 자동으로 구성한다 |
    - 고객 대상 서비스와 사내 서비스 등 상황이 달라져도 Next.js가 항상 가장 좋은 선택일까요?

      #### 1. 결론

        - 항상 그렇지는 않다.
        - Next.js의 장점은 대부분 **서버가 화면을 미리 만들어 보내는 것**에서 나온다. 그게 필요 없는 서비스라면 굳이 쓸 이유가 줄어든다.

      #### 2. 고객 대상 서비스 → Next.js가 잘 맞는다

      처음 들어온 사람에게 잘 보여야 하는 서비스다.

      | 필요한 것 | 이유 |
              | --- | --- |
      | 검색에 잘 나와야 한다 | 서버가 만든 HTML은 검색 엔진이 내용을 읽기 쉽다 |
      | 첫 화면이 빨라야 한다 | 빈 화면이 길면 사용자가 나간다 |
      | 링크 공유가 예쁘게 보여야 한다 | 카톡에 보냈을 때 제목과 이미지가 떠야 한다 |

      예: 쇼핑몰, 블로그, 영화 소개 사이트

      #### 3. 사내 서비스 → 기존 방식(Vite + React)으로도 충분하다

      회사 사람만 로그인해서 쓰는 서비스다.

      | 특징 | 이유 |
              | --- | --- |
      | 검색에 나올 필요가 없다 | 오히려 나오면 안 된다 |
      | 첫 화면 속도가 덜 중요하다 | 같은 사람이 매일 켜 두고 쓴다 |
      | 버튼, 표, 입력창이 많다 | 어차피 대부분 브라우저에서 동작해야 한다 |
      | 배포가 간단하다 | Vite는 파일만 올리면 되고, Next.js는 서버가 필요할 수 있다 |

      예: 관리자 페이지, 대시보드

      #### 4. 정리

      | 상황 | 선택 |
              | --- | --- |
      | 많은 사람이 검색해서 들어오는 서비스 | Next.js |
      | 로그인한 내부 사람만 쓰는 서비스 | Vite + React로도 충분 |
- Server Component와 Client Component
    - 어떤 기준으로 Server Component와 Client Component를 나누면 좋을까요?

      #### 1. 기본 원칙

        - Next.js(App Router)에서는 **기본이 Server Component**다.
        - **브라우저에서만 할 수 있는 일**이 있을 때만 Client Component로 바꾼다.

      #### 2. Client Component를 사용하는 경우

      `"use client"`를 붙인다.

      | 상황 | 예시 |
              | --- | --- |
      | 사용자가 클릭하거나 입력한다 | `onClick`, `onChange` |
      | 화면 상태가 바뀐다 | `useState`, `useReducer` |
      | 화면이 그려진 뒤에 뭔가 한다 | `useEffect` |
      | 브라우저 기능을 쓴다 | `localStorage`, `window`, `alert` |
      | 브라우저용 라이브러리를 쓴다 | TanStack Query의 `useQuery`, Zustand |

      예: 좋아요 버튼, 검색 입력창, 모달, 로그인 폼

      #### 3. Server Component를 사용하는 경우

      아무것도 붙이지 않는다.

      | 상황 | 예시 |
              | --- | --- |
      | 데이터를 가져와서 보여주기만 한다 | 영화 목록, 영화 상세 정보 |
      | 비밀 정보를 쓴다 | API 키, DB 접근 |
      | 내용이 바뀌지 않는다 | 헤더, 푸터, 소개 문구 |
      | 페이지 정보를 설정한다 | `metadata` |

      #### 4. 나누는 방법

      **페이지 전체가 아니라 필요한 부분만** Client Component로 뺀다.

        ```
        영화 상세 페이지 (Server)
         ├─ 포스터, 제목, 줄거리 (Server)  ← 데이터 보여주기만 함
         └─ 좋아요 버튼 (Client)          ← 클릭해야 함
        ```

        - 페이지 전체에 `"use client"`를 붙이면 그 안의 모든 코드가 브라우저로 간다.
        - 버튼만 빼면 브라우저로 보내는 코드가 줄어들어 더 가볍다.

      #### 5. 정리

      | 질문 | 답 |
              | --- | --- |
      | 클릭, 입력, 상태, 브라우저 기능이 필요한가? | 예 → Client |
      | 보여주기만 하거나 서버 데이터를 다루는가? | 예 → Server |
    - `"use client";` 지시어를 `page.tsx` 최상단에 선언해도 될까요?

      #### 1. 결론

        - **선언할 수는 있다.** 에러 없이 동작한다.
        - 하지만 **꼭 필요한 경우가 아니면 권장하지 않는다.**

      #### 2. 선언하면 생기는 일

      | 영향 | 설명 |
              | --- | --- |
      | 페이지 전체가 Client Component가 된다 | 그 파일에서 import한 컴포넌트도 모두 브라우저로 보내진다 |
      | 브라우저로 보내는 JS가 늘어난다 | 보여주기만 하는 부분까지 브라우저 코드에 포함돼서 무거워진다 |
      | `metadata`를 쓸 수 없다 | `metadata`, `generateMetadata`는 Server Component에서만 export할 수 있다 |
      | 컴포넌트에 `async`를 쓸 수 없다 | `await fetch`로 서버에서 바로 데이터를 가져올 수 없다 |
      | 비밀 정보를 다루기 어렵다 | API 키 같은 코드가 브라우저로 노출될 수 있다 |

      #### 3. 더 좋은 방법

      상호작용이 필요한 부분만 따로 파일로 빼서 `"use client"`를 붙인다.

        ```tsx
        // app/components/alert-button.tsx
        "use client";
        
        export default function AlertButton() {
          return <button onClick={() => alert("Hello")}>Click Me</button>;
        }
        ```

        ```tsx
        // app/page.tsx (Server Component 그대로)
        import AlertButton from "./components/alert-button";
        
        export default function HomePage() {
          return (
            <div>
              <h1>This is Home Page.</h1>
              <AlertButton />
            </div>
          );
        }
        ```

      #### 4. 정리

      | 상황 | 선택 |
              | --- | --- |
      | 페이지 일부만 클릭·입력이 필요하다 | 그 부분만 Client Component로 분리 |
      | 페이지 거의 전부가 상호작용이고 `metadata`도 필요 없다 | `page.tsx`에 선언해도 괜찮음 |
- 렌더링 방식
    - `"use client";` 지시어가 붙은 Client Component는 브라우저에서만 렌더링될까요?

      #### 1. 결론

        - **아니다.** Client Component도 **처음 화면은 서버에서 HTML로 먼저 만든다.**
        - `"use client"`는 "브라우저에서만 실행"이 아니라 "브라우저에서도 실행된다"는 뜻이다.

      #### 2. 실제로 일어나는 일

      | 순서 | 어디서 | 하는 일 |
              | --- | --- | --- |
      | 1 | 서버 | Client Component를 한 번 실행해서 첫 화면 HTML을 만든다 |
      | 2 | 브라우저 | HTML을 받아 바로 화면에 보여준다 (아직 클릭은 안 됨) |
      | 3 | 브라우저 | JS를 받아 컴포넌트를 다시 실행하고 이벤트를 연결한다 |
      | 4 | 브라우저 | 이제 클릭, 입력 같은 상호작용이 가능해진다 |

      3번 과정을 하이드레이션(hydration)이라고 한다. 이미 그려진 HTML에 JS를 연결해서 살아 움직이게 만드는 과정이다.

      #### 3. Server Component와 비교

      | 구분 | 서버에서 실행 | 브라우저에서 실행 |
              | --- | --- | --- |
      | Server Component | O | X |
      | Client Component | O (첫 HTML) | O (하이드레이션 이후) |

      #### 4. 그래서 주의할 점

      Client Component의 코드도 서버에서 한 번 실행된다. 그런데 서버에는 `window`, `localStorage`가 없어서, 컴포넌트 본문에서 바로 쓰면 에러가 난다.

        ```tsx
        "use client";
        
        // ❌ 서버에서 실행될 때 에러 (localStorage가 없음)
        const token = localStorage.getItem("token");
        
        // ✅ 브라우저에서만 실행되는 시점에 사용
        useEffect(() => {
          const token = localStorage.getItem("token");
        }, []);
        
        // ✅ 이벤트 핸들러도 브라우저에서만 실행됨
        <button onClick={() => localStorage.setItem("a", "1")} />
        ```

      #### 5. 정리

        - Client Component는 **서버에서 첫 HTML을 만들고, 브라우저에서 이어받아 동작한다.**
        - 브라우저 API는 `useEffect`나 이벤트 핸들러 안에서만 쓴다.
    - SSR과 SSG를 더 정리해 보고, ISR과 ISSG가 각각 어떤 의미로 쓰이는지 조사해 보세요. 페이지 생성 시점과 렌더링 방식을 기준으로 서로 어떤 관계가 있는지도 살펴보세요.

      #### 1. SSR (Server-Side Rendering)

        - **요청이 올 때마다** 서버가 HTML을 새로 만든다.
        - 장점: 항상 최신 데이터를 보여준다.
        - 단점: 요청마다 서버가 일해야 해서 느리고 서버 부담이 크다.
        - 예: 로그인한 사용자별 마이페이지, 실시간 정보

      #### 2. SSG (Static Site Generation)

        - **빌드할 때 한 번** HTML을 만들어 두고, 요청이 오면 만들어 둔 파일을 그대로 준다.
        - 장점: 매우 빠르고 서버 부담이 거의 없다.
        - 단점: 내용을 바꾸려면 다시 빌드해야 한다. 페이지 수가 늘어날수록 빌드 시간도 비례해서 길어진다.
        - 예: 소개 페이지, 워크북의 `/new`

      #### 3. ISR (Incremental Static Regeneration)

        - SSG처럼 미리 만들어 두되, **정해진 시간이 지나면 백그라운드에서 다시 만든다.**
        - 빌드 때 만들지 않은 페이지(예: 새로 추가된 글)도 **처음 요청이 올 때 만들어서** 저장해 둔다.
        - `revalidatePath`, `revalidateTag`로 시간과 상관없이 원할 때 갱신할 수도 있다.
        - 예: 블로그 글, 상품 상세 페이지

      #### 4. ISSG (Incremental Static Site Generation)

        - **Next.js 공식 용어가 아니다.** 공식 문서에는 ISR만 나온다.
        - 커뮤니티에서는 두 가지 뜻으로 섞어 쓴다.

      | 쓰임 | 의미 |
              | --- | --- |
      | ISR과 같은 뜻 | "정적 페이지를 점진적으로 만든다"는 의미로 ISR을 달리 부른 것 |
      | 처음 요청 시 생성 | 빌드 때 모든 페이지를 만들지 않고, **처음 요청이 올 때 만들어 저장**하는 방식만 가리킴 (재생성은 포함하지 않음) |

      #### 5. 생성 시점과 렌더링 방식 기준 비교

      | 방식 | HTML 생성 시점 | 다시 만드는 시점 | 데이터 신선도 | 속도 |
              | --- | --- | --- | --- | --- |
      | SSR | 요청할 때마다 | 매 요청 | 항상 최신 | 느림 |
      | SSG | 빌드할 때 | 다시 빌드할 때 | 빌드 시점 그대로 | 매우 빠름 |
      | ISR | 빌드할 때 + 처음 요청할 때 | 일정 시간 후 또는 직접 요청할 때 | 조금 늦게 최신화 | 빠름 |
      | ISSG | 처음 요청할 때 (또는 ISR과 같음) | 쓰는 사람마다 다름 | 생성 시점 기준 | 처음만 느리고 이후 빠름 |

      #### 6. 관계

        - **SSR과 SSG는 양 끝이다.** SSR은 "매번 새로", SSG는 "한 번만 미리" 만든다.
        - **ISR은 둘의 중간이다.** SSG의 속도를 유지하면서, SSR처럼 내용이 최신으로 바뀔 수 있게 만든 방식이다.
        - **ISSG는 ISR의 일부 또는 다른 이름이다.** "모든 페이지를 빌드 때 만들지 않는다"는 점에 초점을 둔 표현이다.

        ```
        SSG ──────── ISR / ISSG ──────── SSR
        미리 한 번      미리 + 가끔 다시       매번 새로
        빠름, 오래됨                        느림, 최신
        ```

      #### 7. 9주차 실습과 연결

        - `○ /new`: 내용이 고정이라 빌드 때 만든다 → **SSG**
        - `◐ /[id]`: 틀은 빌드 때, 할 일 내용은 요청 때 만든다 → 정적 + **SSR**이 섞인 **부분 사전 렌더링(PPR)**
        - 워크북 설정처럼 Cache Components를 켜면 ISR은 `revalidate` 대신 `"use cache"`와 `cacheLife`로 설정한다.