#### 1. 순수 fetch로 인기 영화 요청

브라우저의 `fetch`로 TMDB 인기 영화 API(`/movie/popular`)를 호출하는 `getMoviesWithFetch` 함수를 작성했다. `fetch`는 `404`나 `500` 응답에서도 Promise를 reject하지 않기 때문에 `response.ok`를 직접 확인하고 실패하면 오류를 던지도록 했다. Network 패널에서 요청 URL이 `https://api.themoviedb.org/3/movie/popular?language=ko-KR&page=1`이고 Method가 `GET`, Status가 `200 OK`인 것을 확인했다. Query String Parameters로 `language`와 `page`가 전달되는 것도 확인했다.

#### 2. ky 인스턴스로 목록, 검색, 상세 화면 연결

`ky.create`로 `tmdbClient`를 만들어 base URL과 인증 header를 한 번만 설정했다. 그 뒤 목록(`getMovies`), 검색(`searchMovies`), 상세(`getMovieDetail`) API 함수를 파일별로 나누고 `searchParams`와 `.json<T>()`로 요청과 응답 타입을 작성했다. 순수 `fetch`로 쓸 때 함수마다 반복되던 base URL, 인증 header, JSON 변환, 오류 확인 코드가 줄어든 것을 확인했다. 목록, 검색, 상세 화면 모두 실제 TMDB 데이터를 보여 준다.

#### 3. 로딩, 오류, 빈 결과, 성공 상태 구분

요청이 끝나기 전에는 데이터가 없고 실패할 수도 있어서 `movies`, `isLoading`, `errorMessage` 상태를 함께 관리하고 화면을 네 가지로 나눴다. 로딩 중에는 불러오는 중이라는 문구를, 실패하면 오류 문구를, 결과가 0건이면 `조건에 맞는 영화가 없어요.`를, 데이터가 있으면 영화 카드를 보여 준다. 검색 화면은 검색어가 비었을 때 요청하지 않고 안내 문구를 먼저 보여 준다. 상세 화면은 영화 번호가 올바르지 않으면 `올바르지 않은 영화 번호예요.`를, 존재하지 않는 영화이면 `영화를 찾을 수 없어요.`를 보여 준다.

목록, 검색, 상세 주소를 주소창에 직접 입력해 접근하고 새로고침해도 화면이 정상적으로 표시되는 것을 확인했다.

#### 4. TMDB 출처 표기, Console 확인, 빌드

화면 하단에 TMDB 출처 표기를 구현했다. 각 화면에서 Console에 빨간 오류가 없는 것을 확인했다. 시크릿 창에서도 확인한 결과 Issues에 보이던 `eval` 관련 항목은 확장 프로그램이 만든 것이었다. 마지막으로 `pnpm build`를 실행해 오류 없이 빌드되는 것을 확인했다.