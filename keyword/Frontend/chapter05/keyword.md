- HTTP와 API
    - API, endpoint, HTTP method와 status code는 요청과 응답에서 각각 어떤 역할을 할까요?

      #### 1. API, endpoint, HTTP method, status code의 역할

      #### 1-1. 한눈에 보기

      | 용어 | 속하는 곳 | 역할 |
              | --- | --- | --- |
      | API | 전체 약속 | 서로 다른 프로그램이 정해진 방법으로 데이터를 주고받게 하는 규칙 |
      | endpoint | 요청 | 어디로 보낼지 정하는 주소 |
      | HTTP method | 요청 | 무엇을 할지 정하는 동사 |
      | status code | 응답 | 결과가 어땠는지 알려 주는 숫자 |
        - 식당에서 손님이 주방에 들어가지 않고 메뉴판을 보고 주문하는 것처럼, API가 메뉴판 역할을 한다.

        ```
        요청: GET <https://api.themoviedb.org/3/movie/popular?language=ko-KR&page=1>
        응답: 200 OK + { page, results: [...], total_pages, total_results }
        ```

      #### 2. endpoint

        - 요청을 보낼 주소로, 서버가 가진 여러 기능 중 어떤 기능에 접근할지 정한다.
        - base URL은 동네, path는 호수에 비유할 수 있다. 같은 base URL이라도 path가 다르면 다른 기능이다.

      | 구성 | 값 | 역할 |
              | --- | --- | --- |
      | base URL | `https://api.themoviedb.org/3` | API 서버의 기본 주소 |
      | path | `/movie/popular` | 어떤 기능인지 구분 |
      | query | `?language=ko-KR&page=1` | 조회 조건 |

      | endpoint | 기능 |
              | --- | --- |
      | `/movie/popular` | 인기 영화 목록 |
      | `/search/movie` | 영화 검색 |
      | `/movie/{movie_id}` | 영화 상세 |

      #### 3. HTTP method

        - 해당 리소스에 어떤 동작을 할지 서버에 알리는 동사이다. endpoint가 "어디로"라면 method는 "무엇을"이다.

      | method | 역할 | CRUD |
              | --- | --- | --- |
      | GET | 데이터를 조회한다. | Read |
      | POST | 데이터를 보내 새로 만든다. | Create |
      | PUT | 대상 데이터를 요청 내용으로 전체 교체한다. | Update |
      | PATCH | 데이터의 일부만 수정한다. | Update |
      | DELETE | 데이터를 삭제한다. | Delete |
        - CRUD와 method가 언제나 일대일로 대응하지는 않으므로 API 명세에서 method, endpoint, body를 함께 확인한다.
        - 이번 TMDB 실습은 데이터를 읽기만 하므로 `GET`만 사용한다.

      #### 4. status code

        - 서버가 응답에 담아 보내는 세 자리 숫자로, 요청이 어떻게 처리되었는지 알려 준다. 첫 자리가 큰 분류를 나타낸다.

      | 분류 | 의미 | 자주 보는 코드 |
              | --- | --- | --- |
      | 2xx | 성공 | `200` 성공, `201` 생성됨, `204` 내용 없음 |
      | 3xx | 리다이렉트 | 다른 주소로 이동해야 한다. |
      | 4xx | 클라이언트 오류 | `400` 잘못된 요청, `401` 인증 실패, `403` 접근 거부, `404` 찾을 수 없음 |
      | 5xx | 서버 오류 | `500` 서버 오류 |
        - 상태 번호만 보고 사용자에게 보여 줄 문구를 단정하지 않고 응답 body의 오류 내용도 함께 확인한다.

      | 상황 | 응답 |
              | --- | --- |
      | 정상 요청 | `200` + `results` 배열이 담긴 JSON |
      | 토큰이 없거나 틀림 | `401` |
      | 존재하지 않는 영화 ID | `404` |

      #### 5. fetch와 ky에서의 처리 차이

      | 구분 | 404, 500 같은 HTTP 오류 | 처리 방법 |
              | --- | --- | --- |
      | fetch | Promise를 reject하지 않는다. | `response.ok`를 직접 확인한다. |
      | ky | 성공 범위가 아닌 status에서 오류를 발생시킨다. | `try/catch`로 처리한다. |
        - `fetch`는 네트워크가 실패했을 때만 reject하고, 서버가 응답을 돌려주면 404여도 "응답은 받았다"로 본다.
        - 그래서 네트워크 오류와 HTTP 오류를 따로 처리해야 한다.
    - 브라우저와 API 서버의 origin이 다를 때 CORS는 어떤 역할을 할까요?

      #### 1. 브라우저와 API 서버의 origin이 다를 때 CORS의 역할

      #### 1-1. 정의

        - 브라우저는 SOP(동일 출처 정책)로 다른 출처의 응답을 JavaScript가 읽는 것을 기본적으로 막는다.
        - CORS는 HTTP 헤더를 주고받는 규칙으로, 서버가 "이 출처에는 응답을 공유해도 된다"고 허용할 수 있게 해 준다.
        - 서버가 허락한 출처의 코드에만 응답을 읽게 해 주는 통행증 역할을 한다.

      | 구분 | 주소 | 설명 |
              | --- | --- | --- |
      | 프론트엔드 | `http://localhost:5173` | 프로토콜, 호스트, 포트 중 하나라도 다르면 다른 출처이다. |
      | API 서버 | `https://api.themoviedb.org` | 위 주소와 세 가지가 모두 다르므로 CORS 규칙이 적용된다. |

      #### 1-2. 누가 무엇을 하는가

      | 주체 | 역할 |
              | --- | --- |
      | 브라우저 | 요청에 `Origin` 헤더를 붙이고, 응답의 CORS 헤더를 보고 JS에 응답을 넘길지 막을지 정한다. |
      | 서버 | `Access-Control-Allow-Origin` 같은 응답 헤더로 허용할 출처를 알린다. |
      | 프론트엔드 코드 | 서버의 허용 정책을 바꿀 수 없다. |

      #### 2. 동작 흐름

        1. 브라우저가 요청에 `Origin` 헤더를 붙여 보낸다.
        2. 서버가 `Access-Control-Allow-Origin` 응답 헤더로 허용 여부를 알린다.
        3. 브라우저가 이 헤더를 보고 응답을 JS에 넘기거나 막는다.

        ```
        요청  Origin: <http://localhost:5173>
        응답  Access-Control-Allow-Origin: <http://localhost:5173>
        ```

      | `Access-Control-Allow-Origin` 값 | 의미 |
              | --- | --- |
      | `*` | 모든 출처를 허용한다. 공개 API에서만 쓰고, 쿠키 같은 자격 증명과는 함께 쓸 수 없다. |
      | 특정 출처 | 그 출처만 허용한다. 여러 출처를 허용하려면 서버가 요청의 `Origin`을 확인해 돌려줘야 한다. |

      #### 3. 요청의 유형

      | 유형 | 설명 |
              | --- | --- |
      | 단순 요청 | 확인 없이 바로 보낸다. `GET`, `HEAD`, `POST` 중 하나이고 안전 목록의 헤더만 쓰는 경우이다. |
      | Preflight 요청 | 본 요청 전에 `OPTIONS`로 "보내도 되나요?"를 먼저 묻는다. 직접 만든 헤더나 다른 method를 쓰면 발생한다. |
        - Preflight는 브라우저가 자동으로 보내므로 직접 만들 필요가 없다.
        - 워크북의 `Authorization` 헤더는 안전 목록에 없어서 preflight가 먼저 나갈 수 있다. Network 패널에서 같은 주소에 요청이 두 개 보이면 preflight와 본 요청일 수 있다.

      #### 4. CORS 오류와 해결

        - 서버가 요청 출처를 허용하지 않으면 브라우저가 응답을 막고 콘솔에 `Cross-Origin Request Blocked` 오류를 띄운다.
        - 보안상 JS 코드는 오류가 났다는 사실만 알 수 있고, 원인은 콘솔의 Reason 메시지로 확인한다.

      | 콘솔 메시지 | 의미 |
              | --- | --- |
      | `Access-Control-Allow-Origin' missing` | 응답에 해당 헤더가 없다. |
      | `Access-Control-Allow-Origin' does not match` | 요청 출처가 허용된 출처와 맞지 않다. |
      | `Credential is not supported if ... is '*'` | 자격 증명을 보내는 요청에 `*`를 쓸 수 없다. |
      | `missing token ... in 'Access-Control-Allow-Headers'` | 서버가 허용하지 않은 헤더를 사용했다. |

      | 상황 | 해결 방법 |
              | --- | --- |
      | 서버를 제어할 수 있다 | 서버에서 요청 출처를 `Access-Control-Allow-Origin`에 추가한다. |
      | 서버를 제어할 수 없다 | 프록시 서버를 거치거나, 단순 요청으로 바꾼다. |
        - `mode: "no-cors"`는 응답 body를 읽을 수 없어서 데이터를 화면에 써야 하는 이번 실습에는 쓸 수 없다.
        - 브라우저 보안 기능을 끄거나 확장 프로그램으로 우회하지 않는다.

      #### 5. CORS 오류가 보일 때 확인할 순서

        1. Network 패널에서 실제 요청이 전송되었는지 확인한다.
        2. API base URL과 protocol, host, port가 맞는지 확인한다.
        3. 콘솔의 Reason 메시지로 원인을 찾는다.
        4. 서버가 현재 프론트 출처를 허용하는지 API 문서에서 확인한다.
- Promise와 비동기 처리
    - Promise의 pending, fulfilled와 rejected 상태는 각각 무엇을 의미할까요?

      #### 1. Promise의 세 가지 상태

      #### 1-1. 정의

        - Promise는 비동기 함수가 돌려주는 객체이고, 작업의 현재 상태를 나타낸다.
        - 함수가 값을 돌려주는 시점에는 작업이 아직 끝나지 않은 경우가 많다. Promise는 나중에 성공이나 실패가 정해졌을 때 처리할 방법을 제공한다.
        - 식당의 진동벨에 비유할 수 있다. 음식이 나오지 않았어도 진동벨(Promise)을 먼저 받고, 음식이 나오거나 주문이 취소되면 벨이 울려 결과를 알려 준다.
        - 워크북의 `fetch`도 호출하면 최종 데이터가 아니라 Promise를 먼저 돌려준다.

      #### 1-2. 상태

      | 상태 | 의미 | 연결되는 핸들러 |
              | --- | --- | --- |
      | pending (대기) | 초기 상태이다. 작업이 아직 끝나지 않았다. | 아직 호출되지 않는다. |
      | fulfilled (이행) | 작업이 성공했다. 결과 값을 가진다. | `.then()` |
      | rejected (거부) | 작업이 실패했다. 실패 이유를 가진다. | `.catch()` |

        ```
                ┌─ 성공 → fulfilled → .then() 실행
        pending ┤
                └─ 실패 → rejected  → .catch() 실행
        
                          (어느 쪽이든 .finally() 실행)
        ```

        - 더 이상 pending이 아닌 fulfilled와 rejected를 settled(확정)라고 한다.
        - 확정된 뒤에는 `resolve`나 `reject`를 다시 호출해도 효과가 없다.
        - resolved는 settled와 다른 심화 개념이므로 입문 단계에서는 세 가지 상태만 구분하면 충분하다.

      #### 2. 성공과 실패의 기준은 API마다 다르다

        - `fetch()`는 네트워크 오류로 요청이 나가지 못하면 reject한다.
        - 서버가 응답을 보냈다면 `404`나 `500`이어도 fulfill한다. 그래서 `response.ok`를 직접 확인한다.

      | 상황 | `fetch`의 Promise 상태 | 처리 방법 |
              | --- | --- | --- |
      | 네트워크 오류 | rejected | `.catch()` 또는 `try/catch` |
      | 서버가 `200` 응답 | fulfilled | `response.json()`으로 데이터를 읽는다. |
      | 서버가 `404`, `500` 응답 | fulfilled | `response.ok`를 확인해 직접 오류를 던진다. |

      #### 3. then, catch, finally

      | 메서드 | 호출되는 시점 |
              | --- | --- |
      | `.then(onFulfilled)` | fulfilled일 때 |
      | `.catch(onRejected)` | rejected일 때 |
      | `.finally(callback)` | 성공과 실패에 관계없이 settled가 된 뒤 |
        - `.then()`은 새 Promise를 돌려주므로 이어서 쓸 수 있다. 이를 promise chaining이라고 한다.
        - 체인 끝에 `.catch()` 하나를 붙이면 어느 단계에서 실패해도 한 곳에서 처리한다.
        - 이미 확정된 Promise에 핸들러를 붙여도 핸들러는 호출된다.

      #### 4. 상태가 바뀌는 과정

        ```
        const movieRequest = new Promise<string>((resolve) => {
          setTimeout(() => resolve("영화 요청이 끝났어요."), 300);
        });
        
        console.log("1. 요청을 시작했어요.");
        
        movieRequest
          .then((message) => console.log(`2. ${message}`))
          .catch((error) => console.error(error))
          .finally(() => console.log("3. 요청 처리를 마쳤어요."));
        ```

      | 시점 | 상태 | 일어나는 일 |
              | --- | --- | --- |
      | `new Promise(...)` 직후 | pending | 300ms 타이머가 시작된다. |
      | 300ms 후 `resolve(...)` 호출 | fulfilled | `.then()`이 실행되어 2번이 출력된다. |
      | 그다음 | fulfilled | `.finally()`가 실행되어 3번이 출력된다. |
        - `resolve` 대신 `reject`가 호출되면 rejected가 되고 `.then()`은 건너뛴 채 `.catch()`와 `.finally()`가 실행된다.
    - `.then().catch()`와 `async/await`는 같은 비동기 흐름을 어떻게 다르게 표현하나요?

      #### 1. 두 방식의 관계

      #### 1-1. 같은 Promise를 읽는 두 가지 표현

        - `.then().catch()`와 `async/await`는 서로 다른 비동기 방식이 아니다. 둘 다 Promise가 확정(settled)된 뒤 이어서 실행할 코드를 표현하는 방법이다.
        - `.then().catch()`는 Promise에 핸들러를 연결하는 방식이다. `async/await`는 `await`로 Promise를 풀어서 값을 변수로 받는 방식이다.
        - 비유하면 같은 길을 안내하는 두 가지 방법이다. `.then()`은 "도착하면 이 일을 하세요"라는 메모를 붙이는 방식이고, `await`는 "도착할 때까지 기다렸다가 이어서 하겠다"고 줄을 세우는 방식이다.

      #### 1-2. 대응 관계

      | 구분 | `.then().catch()` | `async/await` |
              | --- | --- | --- |
      | 성공 값 받기 | `.then()` 콜백의 인자로 받는다. | `await`의 결과로 받는다. |
      | 실패 처리 | `.catch()`가 거부 이유를 받는다. | `await`가 거부 이유를 던지고, `try/catch`가 받는다. |
      | 마무리 처리 | `.finally()` | `try/catch/finally`의 `finally` |
      | 순서 연결 | `.then()`을 이어서 쓴다(체이닝). | `await` 줄을 위에서 아래로 쓴다. |
      | 함수의 반환 | `.then()`은 새 Promise를 돌려준다. | `async` 함수는 항상 Promise를 돌려준다. |

      #### 2. 같은 일을 두 방식으로 쓰기

      #### 2-1. 검색 요청 예시

      `.then().catch()` 방식이다.

        ```
        searchMovies({ query: normalizedQuery })
          .then((response) => {
            if (!ignore) setMovies(response.results);
          })
          .catch(() => {
            if (!ignore) setErrorMessage("검색 결과를 불러오지 못했어요.");
          })
          .finally(() => {
            if (!ignore) setIsLoading(false);
          });
        ```

      `async/await` 방식이다.

        ```
        async function loadSearchResults() {
          try {
            const response = await searchMovies({ query: normalizedQuery });
            if (!ignore) setMovies(response.results);
          } catch {
            if (!ignore) setErrorMessage("검색 결과를 불러오지 못했어요.");
          } finally {
            if (!ignore) setIsLoading(false);
          }
        }
        
        loadSearchResults();
        ```

        - 두 코드는 같은 Promise를 같은 순서로 처리한다.
        - 앞쪽은 핸들러를 줄줄이 연결해 흐름을 표현하고, 뒤쪽은 동기 코드처럼 위에서 아래로 읽힌다.

      #### 2-2. await가 있으면 코드가 둘로 나뉜다

        - `await` 앞의 코드는 바로 실행되고, 뒤의 코드는 Promise가 확정된 뒤 이어서 실행된다.
        - 이는 `.then()`으로 뒤쪽 코드를 연결한 것과 같다.

        ```
        async function foo(name) {
          console.log(name, "start");
          await console.log(name, "middle");
          console.log(name, "end");
        }
        
        // 위 함수는 아래와 같이 표현할 수 있다.
        function foo(name) {
          return new Promise((resolve) => {
            console.log(name, "start");
            resolve(console.log(name, "middle"));
          }).then(() => {
            console.log(name, "end");
          });
        }
        ```

      #### 3. 에러 처리 비교

      #### 3-1. 처리 방식

      | 방식 | 에러를 잡는 위치 | 설명 |
              | --- | --- | --- |
      | `.then().catch()` | 체인 끝의 `.catch()` | 체인 안의 어느 단계에서 실패해도 `.catch()` 한 곳에서 처리한다. |
      | `async/await` | `try/catch` 블록 | Promise가 거부되면 `await`가 거부 이유를 던지므로 동기 코드와 같은 방식으로 처리한다. |
      | 혼합 | `await promise.catch(...)` | `try` 없이 거부된 Promise만 처리할 수 있다. |
        - `.then()` 안에서 `throw`하면 그 `.then()`이 만든 Promise가 거부되어 `.catch()`로 이어진다.
        - `await promisedFunction().catch(...)` 형태는 함수가 항상 Promise를 반환하는 경우에만 안전하다. 함수가 동기적으로 에러를 던지면 `.catch()`가 잡지 못하므로 `try...catch`가 필요하다.
        - MDN `async function` 문서는 대부분의 `async` 함수를 Promise로도 쓸 수 있지만, 에러 처리에서는 `async` 함수가 조금 덜 실수하기 쉽다고 설명한다.

      #### 3-2. fetch와 함께 쓸 때

        - `fetch`는 404나 500 응답에서도 Promise를 reject하지 않는다.
        - 그래서 두 방식 모두 `response.ok`를 확인해 직접 오류를 던져야 한다.

        ```
        // .then() 방식
        fetch(url)
          .then((response) => {
            if (!response.ok) {
              throw new Error(`HTTP error: ${response.status}`);
            }
            return response.json();
          })
          .then((data) => console.log(data))
          .catch((error) => console.error(error));
        
        // async/await 방식
        async function load() {
          try {
            const response = await fetch(url);
            if (!response.ok) {
              throw new Error(`HTTP error: ${response.status}`);
            }
            const data
        ```

    - `await`를 사용해도 브라우저 전체가 멈추지 않는 이유는 무엇일까요?

      #### 1. await를 사용해도 브라우저 전체가 멈추지 않는 이유

      #### 1-1. await가 멈추는 것은 해당 함수뿐이다

        - `await`는 브라우저 전체를 멈추는 문법이 아니다. 그 `await`를 감싸고 있는 `async` 함수의 나머지 코드 실행만 Promise가 확정될 때까지 미룬다.
        - `await`를 만나면 제어권이 그 함수를 호출한 쪽으로 돌아가고, 호출한 쪽 코드와 다른 작업은 계속 진행된다.
        - 식당에 비유하면 직원이 주문을 넣고 진동벨을 받은 뒤 다른 손님을 응대하다가, 음식이 나와 벨이 울리면 하던 일을 이어서 하는 것과 같다.

      | 대상 | `await`의 영향 |
              | --- | --- |
      | 해당 `async` 함수의 `await` 아래 코드 | Promise가 확정될 때까지 실행이 미뤄진다. |
      | 그 함수를 호출한 쪽의 코드 | 계속 실행된다. |
      | 다른 `async` 함수나 이벤트 처리 | 계속 실행된다. |
      | 브라우저 전체 | 멈추지 않는다. |

      #### 2. 배경 원리

        - 메인 스레드는 브라우저가 사용자 이벤트를 처리하고 화면을 그리는 곳이다. JavaScript는 이 스레드에서 한 번에 한 문장씩 실행된다.
        - 비동기 작업을 가만히 기다리면 프로그램 전체가 멈추므로, JavaScript는 완료 후 처리할 코드를 콜백으로 맡겨 두고 다른 일을 계속하도록 설계되어 있다.
        - 작업이 끝나면 그 콜백이 job이 되어 큐에 들어가고, 이벤트 루프가 큐에서 하나씩 꺼내 실행한다. Promise의 `.then()` 핸들러도 이 콜백에 해당한다.
        - 그래서 `fetch()` 요청을 기다리는 동안에도 사용자 입력 같은 다른 작업을 처리할 수 있다.

      #### 3. await가 실행되는 과정

        1. `async` 함수를 호출하면 첫 `await` 앞까지는 바로 실행된다.
        2. `await`를 만나면 그 아래 코드는 일시 중지되고, 제어권은 호출한 쪽으로 돌아간다.
        3. 그동안 메인 스레드는 다른 job을 처리한다.
        4. Promise가 확정되면 멈춰 있던 코드를 이어서 실행할 microtask가 예약된다.
        5. fulfilled이면 값이 `await`의 결과가 되고, rejected이면 거부 이유가 던져진다.

        ```
        async function loadMovies() {
          console.log("1. 요청 시작");
          const response = await getMovies({ page: 1 });   // 여기서 loadMovies만 멈춘다.
          console.log("3. 응답 도착", response.results);    // Promise가 확정되면 이어서 실행된다.
        }
        
        loadMovies();
        console.log("2. 호출한 쪽 코드는 계속 실행된다.");
        ```

        - 출력 순서는 1, 2, 3이다. `loadMovies`가 멈춘 동안 호출한 쪽의 `console.log`가 먼저 실행된다.

      #### 4. 브라우저가 멈추는 경우

        - 문제가 되는 것은 `await`가 아니라 오래 걸리는 동기 코드이다.
        - 각 job은 한 번 시작하면 끝날 때까지 다른 코드가 끼어들 수 없다. job이 너무 오래 걸리면 click이나 scroll 같은 상호작용을 처리하지 못한다.

      | 경우 | 브라우저에 미치는 영향 |
              | --- | --- |
      | `await fetch(...)`로 응답을 기다린다. | 기다리는 동안 다른 작업이 처리되므로 멈추지 않는다. |
      | 큰 배열을 오래 도는 동기 반복문을 실행한다. | 그 job이 끝날 때까지 화면이 멈춘다. |
        - 무거운 계산은 job을 짧게 쪼개거나 Web Worker처럼 메인 스레드 밖에서 실행한다.

      #### 5. await를 쓸 때 알아둘 점

        - `await`는 `async` 함수 안이나 모듈의 최상위에서만 쓸 수 있다.
        - `await`는 작업을 순서대로 실행한다. 서로 의존하지 않는 요청은 `Promise.all()`로 함께 시작하는 편이 효율적이다.
        - `async` 함수는 항상 Promise를 반환한다. 호출한 쪽에서 값을 쓰려면 다시 `await`하거나 `.then()`을 연결해야 한다.
- fetch와 HTTP 요청 라이브러리
    - `fetch`에서 네트워크 오류와 HTTP 오류를 따로 처리해야 하는 이유는 무엇일까요?

      #### 1. fetch에서 네트워크 오류와 HTTP 오류를 따로 처리해야 하는 이유

      #### 1-1. 발생하는 지점이 다르다

        - 네트워크 오류는 요청이 서버에 닿지 못했거나 응답을 받지 못한 경우이다. 이때 `fetch`의 Promise는 rejected가 된다.
        - HTTP 오류는 서버가 응답은 보냈는데 status가 `404`, `500` 같은 오류인 경우이다. 이때 `fetch`의 Promise는 fulfilled가 된다.
        - `fetch`가 말하는 "성공"은 "응답을 받았다"는 뜻이다. 서버가 보낸 결과가 성공인지는 판단하지 않는다.
        - 전화에 비유하면 연결이 안 되는 것이 네트워크 오류이고, 연결은 됐는데 상대가 "그런 상품은 없어요"라고 답하는 것이 HTTP 오류이다.

      | 구분 | 네트워크 오류 | HTTP 오류 |
              | --- | --- | --- |
      | 상황 | 연결할 수 없거나 URL이 잘못된 경우 등 | 서버가 `404`, `500` 등으로 응답한 경우 |
      | 서버 응답 도착 | 아니오 | 예 |
      | `fetch`의 Promise | rejected (`TypeError`) | fulfilled |
      | 확인 방법 | `.catch()` 또는 `try/catch` | `response.ok` 또는 `response.status` |

      #### 2. 따로 처리하지 않으면 생기는 문제

        ```
        const response = await fetch(url);        // 404여도 여기까지 정상 통과한다.
        const data = await response.json();       // 오류 응답의 body를 정상 데이터처럼 읽는다.
        setMovies(data.results);                  // 오류 응답에는 results가 없어 undefined가 들어간다.
        ```

        - `try/catch`만 쓰면 네트워크 오류는 잡히지만 `404`, `500`은 `catch`에 도달하지 않는다.
        - 그 결과 오류 응답이 정상 데이터처럼 화면 로직으로 흘러 들어간다.

      #### 3. 올바른 처리 방법

        - `response.ok`로 HTTP 오류를 확인하고 직접 `throw`하면 네트워크 오류와 같은 `catch`로 합쳐진다.
        - `response.ok`는 status가 200~299 범위이면 `true`이다.

        ```
        try {
          const response = await fetch(url);
        
          if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);   // HTTP 오류를 직접 던진다.
          }
        
          const data = await response.json();
        } catch (error) {
          // 네트워크 오류와 위에서 던진 HTTP 오류가 모두 여기로 온다.
          console.error(error);
        }
        ```

        - 워크북의 `getMoviesWithFetch`도 같은 방식으로 `영화 목록 요청에 실패했어요: ${response.status}`를 던진다.
        - 사용자에게 상황별 문구를 보여 주려면 status code로 구분한다. 네트워크 오류에는 status가 없다.

      #### 4. ky에서는 어떻게 다른가

        - ky는 성공 범위가 아닌 status에서 기본적으로 오류를 발생시키므로 `response.ok`를 직접 확인할 필요가 없다.
        - 워크북의 `getMovieDetailErrorMessage`는 `isHTTPError(error)`와 `error.response.status === 404`로 HTTP 오류를 구분한다.
        - 순수 `fetch`로 먼저 작성해 보면 ky가 어떤 반복을 줄여 주는지 이해할 수 있다.
    - 순수 `fetch`로 여러 API 함수를 작성하면 어떤 설정과 처리가 반복될까요?

      #### 1. 순수 fetch로 API 함수를 여러 개 작성하면

      #### 1-1. 반복되는 설정과 처리

        - 목록, 검색, 상세 요청을 모두 `fetch`로 쓰면 endpoint와 query만 달라지고 나머지는 함수마다 똑같이 반복된다.

      | 반복되는 부분 | 왜 매번 필요한가 |
              | --- | --- |
      | base URL 붙이기 | `fetch`는 완성된 URL을 받으므로 `${env.tmdbApiBaseUrl}/...`를 매번 이어 붙인다. |
      | 인증 header | 모든 요청에 `Authorization: Bearer ...`를 넣어야 한다. |
      | query 만들기 | `URLSearchParams`를 만들고 값을 문자열로 바꿔야 한다. (`String(page)`) |
      | HTTP 오류 확인 | `fetch`는 `404`, `500`에서 reject하지 않으므로 `response.ok`를 확인하고 직접 `throw`한다. |
      | JSON 변환 | `response.json()`을 매번 호출한다. |
      | 반환 타입 명시 | `response.json()`만으로는 타입을 알 수 없어 `Promise<...>`를 직접 적는다. |

        ```
        const response = await fetch(`${env.tmdbApiBaseUrl}/movie/popular?${searchParams}`, {
          headers: { Authorization: `Bearer ${env.tmdbAccessToken}` },
        });
        
        if (!response.ok) {
          throw new Error(`영화 목록 요청에 실패했어요: ${response.status}`);
        }
        
        return response.json();
        ```

        - 위 코드가 검색, 상세 함수에서도 endpoint와 오류 문구만 바뀐 채 그대로 반복된다.

      #### 1-2. 반복이 문제가 되는 이유

        - 토큰 header나 base URL이 바뀌면 모든 함수를 찾아서 고쳐야 한다.
        - 새 함수를 만들 때 `response.ok` 확인이나 인증 header를 빠뜨리기 쉽다.
        - 요청마다 다른 부분이 반복 코드 사이에 묻혀 읽기 어렵다.

      #### 2. ky로 반복 줄이기

        - 공통 설정(base URL, 인증 header)은 인스턴스에 한 번만 둔다.

        ```
        export const tmdbClient = ky.create({
          baseUrl: `${env.tmdbApiBaseUrl}/`,
          headers: { Authorization: `Bearer ${env.tmdbAccessToken}` },
        });
        ```

      | 항목 | 순수 fetch | ky |
              | --- | --- | --- |
      | base URL, 인증 header | 함수마다 작성한다. | 인스턴스에 한 번 설정한다. |
      | query | `URLSearchParams`로 직접 만든다. | `searchParams` 옵션에 객체로 쓴다. |
      | HTTP 오류 확인 | `response.ok`를 확인하고 직접 `throw`한다. | 성공 범위가 아닌 status에서 기본적으로 오류를 발생시킨다. |
      | JSON 변환과 타입 | `response.json()`과 `Promise<...>`를 직접 쓴다. | `.json<T>()`로 한 번에 처리한다. |
        - ky로 바꿔도 endpoint path, query 값, 응답 타입은 함수마다 직접 써야 한다. 없어지는 것은 반복이지 요청의 차이가 아니다.
    - ky와 Axios는 JSON 응답을 읽고 공통 요청 처리를 추가하는 방식이 어떻게 다를까요?

      #### 1. ky와 Axios의 차이

      #### 1-1. JSON 응답을 읽는 방식

        - ky는 Fetch API 기반이고, 요청 결과에 `.json()` 같은 body 메서드를 바로 이어서 호출한다.
        - `.json<T>()`처럼 제네릭을 주면 반환 타입까지 함께 표현되고, 타입 인자를 주지 않으면 `unknown`이다.
        - Axios는 요청이 `response` 객체로 resolve되고, 서버가 보낸 응답 본문은 `response.data`에서 읽는다.
        - `response`에는 `data` 외에 `status`, `statusText`, `headers`, `config`, `request`도 들어 있다.

        ```
        // ky
        const movies = await ky.get("movie/popular").json<GetMoviesResponse>();
        
        // Axios
        const { data } = await axios.get("/movie/popular");
        ```

      #### 1-2. 공통 요청 처리를 추가하는 방식

        - 공통 설정은 둘 다 인스턴스로 만든다. ky는 `ky.create()`와 `ky.extend()`, Axios는 `axios.create()`를 쓴다.
        - 요청과 응답 전후의 공통 처리는 ky는 hooks, Axios는 interceptors로 추가한다.

      | 비교 기준 | ky | Axios |
              | --- | --- | --- |
      | 기반 | Fetch API | 브라우저와 Node.js에서 같은 Axios API로 요청 |
      | JSON 응답 | `.json<T>()`의 결과로 body를 바로 읽는다. | `response.data`에서 body를 읽는다. |
      | 인스턴스 | `ky.create()`, `ky.extend()` | `axios.create()` |
      | 공통 설정 | `baseUrl`, `headers` 등을 인스턴스에 설정한다. | `baseURL`, `timeout`, `headers` 등을 인스턴스에 설정한다. |
      | 전후 처리 | hooks: `beforeRequest`, `afterResponse`, `beforeError`, `beforeRetry`, `init` | interceptors: `interceptors.request.use`, `interceptors.response.use` |
      | HTTP 오류 | non-2xx에서 `HTTPError`를 던진다. (`throwHttpErrors` 기본값 `true`) | 2xx이면 resolve하고 그 밖이면 reject한다. (`validateStatus`로 바꿀 수 있다.) |

      #### 1-3. 공통 처리 예시

        ```
        // ky: 인증 header를 hooks로 추가
        const api = ky.create({
          hooks: {
            beforeRequest: [
              ({ request }) => {
                request.headers.set("Authorization", `Bearer ${getToken()}`);
              },
            ],
          },
        });
        
        // Axios: 인증 header를 interceptors로 추가
        const api = axios.create({ baseURL: "<https://api.example.com>" });
        
        api.interceptors.request.use((config) => {
          config.headers.Authorization = `Bearer ${getToken()}`;
          return config;
        });
        ```

      #### 1-4. 알아둘 차이

        - ky는 `.json()`처럼 body 메서드를 호출하면 상황에 맞는 `Accept` header를 자동으로 설정한다.
        - ky는 실패한 요청을 기본적으로 재시도한다. 기본 재시도 횟수는 2회이다.
        - Axios의 응답 interceptor는 보통 2xx 응답이면 `onFulfilled`, 그 밖이면 `onRejected`가 호출된다. 이 동작은 `validateStatus`에 따라 달라진다.
        - Axios에서 request interceptor는 나중에 추가한 것부터, response interceptor는 추가한 순서대로 실행된다.
        - Axios의 `eject()`와 `clear()`로 interceptor를 제거할 수 있다.

      #### 1-5. 무엇을 고를까

        - 둘 중 하나가 항상 더 좋은 것은 아니다.
        - 프로젝트에서 이미 쓰는 도구, 팀의 규칙, 필요한 기능을 기준으로 고른다.
        - 이 워크북은 앞에서 배운 `fetch`와 비교하기 쉬운 ky로 진행한다. 팀에서 Axios를 쓰기로 했다면 팀의 규칙을 따른다.
- 비동기 데이터의 UI 상태
    - 서버 데이터를 보여 주는 화면에서 로딩, 오류, 빈 결과와 성공 상태를 구분해야 하는 이유는 무엇일까요?

      #### 1. 로딩, 오류, 빈 결과, 성공을 구분해야 하는 이유

      #### 1-1. 요청 결과는 시간에 따라 바뀐다

        - 서버 요청은 끝나기 전에는 데이터가 없고, 실패할 수도 있다.
        - 데이터만 저장하면 "아직 도착하지 않음", "요청이 실패함", "결과가 0건임"을 구분할 수 없다.
        - 초기값이 빈 배열(`[]`)이면 로딩 중에도 빈 결과처럼 보인다.

      | 상태 | 의미 | 보여 줄 화면 | 구분하지 않으면 |
              | --- | --- | --- | --- |
      | 로딩 | 응답을 기다리는 중이다. | 불러오는 중 안내나 skeleton | 빈 화면이나 "결과 없음"으로 오해한다. |
      | 오류 | 요청이 실패했다. | 오류 문구 | 빈 결과처럼 보여 원인을 알 수 없다. |
      | 빈 결과 | 요청은 성공했지만 데이터가 0건이다. | "조건에 맞는 영화가 없어요." | 로딩이나 오류로 오해한다. |
      | 성공 | 데이터가 있다. | 영화 카드 | 해당 없음 |

      #### 1-2. 상태마다 사용자가 해야 할 행동이 다르다

      | 상태 | 사용자가 할 행동 |
              | --- | --- |
      | 로딩 | 기다린다. |
      | 오류 | 다시 시도하거나 나중에 다시 온다. |
      | 빈 결과 | 검색어를 바꾼다. |
      | 성공 | 영화를 선택한다. |
        - 상태를 구분해서 알려 주지 않으면 사용자는 무엇을 해야 할지 판단할 수 없다.

      #### 2. 구현 방법

      #### 2-1. 상태를 함께 관리한다

        - React 공식 문서는 컴포넌트가 가질 수 있는 시각 상태를 먼저 나열하고, 각 상태의 화면을 만든 뒤에 로직을 붙이라고 설명한다.
        - 워크북은 `movies`, `isLoading`, `errorMessage`를 함께 관리한다.
        - 요청을 시작할 때 `setIsLoading(true)`와 `setErrorMessage(null)`로 이전 상태를 지워야 이전 오류가 남지 않는다.

      #### 2-2. 확인 순서가 중요하다

        ```
        if (isLoading) return <p>영화 목록을 불러오는 중이에요.</p>;
        if (errorMessage) return <p>{errorMessage}</p>;
        if (movies.length === 0) return <p>조건에 맞는 영화가 없어요.</p>;
        
        return <MovieList movies={movies} />;
        ```

        - 로딩과 오류를 먼저 확인해야 `movies.length === 0`이 진짜 빈 결과일 때만 실행된다.
        - 검색 화면은 빈 검색어 안내를 가장 먼저 구분한 뒤 로딩, 오류, 빈 결과, 성공 순으로 표시한다.
    - 각 상태를 사용자가 이해할 수 있게 표현하려면 어떤 정보와 동작이 필요할까요?

      #### 1. 상태별로 필요한 정보와 동작

      #### 1-1. 정리

      | 상태 | 알려 줄 정보 | 필요한 동작 |
              | --- | --- | --- |
      | 로딩 | 지금 불러오는 중이라는 사실 (`영화 목록을 불러오는 중이에요.`, skeleton) | 사용자는 기다린다. |
      | 오류 | 실패했다는 사실과 상황에 맞는 이유 (`영화를 찾을 수 없어요.`) | 다시 시도 버튼 (선택 미션: 마지막으로 실패한 요청을 다시 실행) |
      | 빈 결과 | 요청은 성공했지만 결과가 없다는 사실 (`조건에 맞는 영화가 없어요.`) | 검색어를 바꾸도록 유도한다. |
      | 성공 | 영화 카드와 상세 정보 | 카드 선택, 상세 이동 |

      #### 1-2. 데이터가 없을 때의 안내

      | 경우 | 안내 |
              | --- | --- |
      | 빈 검색어 | 요청하지 않고 검색 안내를 보여 준다. |
      | 잘못된 영화 번호 | `올바르지 않은 영화 번호예요.` (요청하지 않는다.) |
      | `poster_path`가 `null` | 같은 크기의 회색 영역에 `이미지 없음`을 표시한다. |
      | `runtime`이 `null` | `상영 시간 정보가 없어요.` |

      #### 2. 공통 원칙

        - 상태는 한 번에 하나만 보여 준다. 로딩이 끝나기 전에 빈 결과를 보여 주지 않는다.
        - 문구는 사용자가 이해할 수 있는 말로 쓰고, status 숫자를 그대로 노출하지 않는다. 상태 번호만 보고 문구를 단정하지 않고 응답 body의 오류 내용도 함께 확인한다.
        - 상태가 바뀌어도 기존 디자인의 영역 크기와 비율을 유지한다.
        - 다음에 할 행동을 알려 준다. 오류에는 다시 시도, 빈 결과에는 검색어 변경이 해당한다.

      #### 3. 접근성

        - 화면을 읽어 주는 보조기술 사용자에게도 상태가 전달되어야 한다.
        - 로딩이나 성공 같은 상태 메시지는 `role="status"`를 쓴다. 기본값이 `aria-live="polite"`라서 포커스를 옮기지 않아도 안내된다.
        - 오류처럼 즉시 알려야 하는 메시지는 `role="alert"`를 쓴다. 기본값이 `assertive`이다.
        - 중요하지 않은 내용에 `role="alert"`를 쓰면 안 된다.
        - `aria-busy="true"`는 요소가 갱신되는 중이라는 뜻이다. 로딩이 끝나면 `false`로 바꾼다.
        - 안내 메시지에 포커스를 주거나 사용자가 직접 닫게 하지 않는다.
    - API 응답 타입과 화면에서 사용하는 타입을 분리하면 어떤 상황에서 도움이 될까요?

      #### 1. API 응답 타입과 화면 타입을 분리하면 도움이 되는 상황

      #### 1-1. 두 타입의 차이

      | 구분 | API 응답 타입 | 화면에서 사용하는 타입 |
              | --- | --- | --- |
      | 기준 | 서버가 보내는 JSON의 모양 | 컴포넌트가 쓰기 편한 모양 |
      | 이름 | `snake_case` 그대로 (`poster_path`, `release_date`) | 화면에 맞는 이름 (`posterUrl`, `releaseYear`) |
      | 예시 | `TmdbMovieListItem`, `TmdbMovieDetail` | 영화 카드에 필요한 값만 담은 타입 |

      #### 1-2. 도움이 되는 상황

      | 상황 | 분리하면 좋은 점 |
              | --- | --- |
      | 서버 응답 구조가 바뀐다. | 변환하는 곳만 고치면 되고 화면 컴포넌트는 그대로 둔다. |
      | 필드 이름이나 형식이 화면과 다르다. | `poster_path`를 이미지 URL로 바꾸는 식으로 화면이 쓰기 좋은 값으로 맞춘다. |
      | 화면에만 필요한 값이 있다. | 북마크 여부는 영화 `id`로 store에서 계산하므로 API 타입에 넣지 않는다. |
      | 목록, 검색, 상세 응답의 필드가 다르다. | 목록은 `genre_ids`, 상세는 `genres`, `runtime`, `tagline`이 있는데, 화면은 필요한 필드만 가진 하나의 카드 타입으로 통일한다. |
      | 로컬 데이터와 API 데이터를 같은 카드로 보여 준다. | 2주차의 영화 카드 컴포넌트를 TMDB 응답에 맞춰 고치지 않고 그대로 쓴다. |

      #### 1-3. 변환 함수(adapter) 예시

        ```
        interface MovieCard {
          id: number;
          title: string;
          posterUrl: string | null;
          releaseYear: string;
          rating: number;
        }
        
        function toMovieCard(movie: TmdbMovieListItem): MovieCard {
          return {
            id: movie.id,
            title: movie.title,
            posterUrl: getTmdbPosterUrl(movie.poster_path),
            releaseYear: movie.release_date.slice(0, 4),
            rating: movie.vote_average,
          };
        }
        ```

        - 화면 컴포넌트는 `MovieCard`만 알면 되므로 `poster_path` 같은 서버의 이름을 직접 쓰지 않는다.
        - 이 변환은 워크북의 선택 미션(TMDB 응답을 기존 영화 카드의 UI 타입으로 변환하는 adapter 함수)에 해당한다.

      #### 2. 주의할 점

        - 두 타입 모두 TypeScript가 코드를 검사할 때만 쓰이며 서버의 실제 응답을 검사하지는 않는다. API 타입은 TMDB 문서와 Network 패널의 응답으로 함께 확인한다.
        - 화면이 단순하면 분리가 오히려 과할 수 있다. 워크북도 기본 과정에서는 API 타입을 `snake_case` 그대로 작성하고, 변환은 선택 미션으로 둔다.
        - 변환 함수는 한 곳에 모아 두어야 효과가 있다. 화면 곳곳에서 변환하면 서버 응답이 바뀔 때 고칠 곳이 다시 늘어난다.