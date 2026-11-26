9주차 미션: UMCine Next.js 마이그레이션
1. 개요
   항목	내용
   기존	umcine/ (Vite + TanStack Router + TanStack Query + zustand + ky)
   변경	umcine-next/ (Next.js 16 App Router + TanStack Query + zustand + ky)
   위치	레포 최상위, 기존 umcine/ 옆
   결과	pnpm build 성공
2. 폴더 구조 변경
   기존	변경	이유
   src/main.tsx, index.html	삭제	Next가 진입점과 HTML을 직접 만든다
   src/routes/	src/app/	파일 기반 라우팅 규칙이 다르다
   src/pages/	src/views/	src/pages가 있으면 Next가 Pages Router로 인식해 충돌한다
   src/index.css	src/app/globals.css	루트 layout에서 전역 CSS를 import한다
   @/* → ./src/*	그대로 유지	기존 import 경로를 수정하지 않기 위해서다
3. 라우트 변환
   TanStack Router	Next App Router
   __root.tsx	app/layout.tsx
   index.tsx	app/page.tsx
   movies.$movieId.tsx	app/movies/[movieId]/page.tsx
   search.tsx	app/search/page.tsx
   login.tsx, signup.tsx	app/login/page.tsx, app/signup/page.tsx
   _auth.tsx (pathless)	app/(auth)/layout.tsx (route group)
   _auth/me/index.tsx, _auth/me/edit.tsx	app/(auth)/me/page.tsx, app/(auth)/me/edit/page.tsx
   notFoundComponent	app/not-found.tsx
   원래 화면 코드는 views/에 두고, page.tsx는 그 컴포넌트를 렌더링만 한다.
4. 라우터 API 교체
   TanStack Router	Next
   <Link to="/x">	<Link href="/x"> (next/link)
   to="/movies/$movieId" params={...}	href={`/movies/${id}`}
   useNavigate() / navigate({ to })	useRouter() / router.push()
   useRouterState().location.pathname	usePathname()
   useParams({ from })	useParams<{ movieId: string }>()
   useSearch({ from })	useSearchParams().get("query")
   <Navigate to />	useEffect + router.replace()
   <Outlet />	children
5. Provider 설정
   app/providers.tsx를 클라이언트 컴포넌트로 분리하고 QueryClientProvider, ReactQueryDevtools를 둔다.
   QueryClient는 useState(() => new QueryClient())로 만든다. 모듈 최상단에서 만들면 서버에서 요청끼리 캐시가 공유될 수 있다.
6. 인증 가드
   기존	변경	이유
   _auth beforeLoad redirect	AuthLayout의 useEffect에서 router.replace("/login")	토큰이 localStorage에 있어서 서버에서 확인할 수 없다
   login beforeLoad redirect	GuestOnly 컴포넌트 추가	로그인 상태면 /로 보낸다
7. "use client" 경계
   App Router의 컴포넌트는 기본적으로 서버 컴포넌트다.
   훅, 상태, 이벤트, localStorage를 쓰는 컴포넌트 중 서버 파일(page, layout)이 직접 import하는 진입 파일에만 붙인다.
   그 아래에서 import되는 파일(hooks, stores, movie-card 등)은 자동으로 클라이언트 번들에 포함된다.
   적용 파일
   providers.tsx, header.tsx, auth-layout.tsx, guest-only.tsx
   views/ 아래 page 컴포넌트 7개
   footer.tsx는 상태가 없어서 서버 컴포넌트로 유지한다.
8. Suspense
   next.config.ts의 cacheComponents: true 설정에서는 usePathname, useSearchParams처럼 요청마다 달라지는 값을 쓰는 부분이 Suspense 안에 있어야 한다.
   루트 layout에서 Header와 children을 각각 <Suspense>로 감쌌다.
9. 환경변수
   기존	변경
   VITE_TMDB_API_BASE_URL	NEXT_PUBLIC_TMDB_API_BASE_URL
   VITE_TMDB_ACCESS_TOKEN	NEXT_PUBLIC_TMDB_ACCESS_TOKEN
   VITE_API_BASE_URL	NEXT_PUBLIC_API_BASE_URL
   import.meta.env.X	process.env.X
   Next는 NEXT_PUBLIC_ 접두사가 붙은 변수만 브라우저 코드에 넣는다.
   빌드할 때 process.env.NEXT_PUBLIC_X라는 글자를 그대로 값으로 바꾸는 방식이라, 구조분해로 접근하면 안 된다.
10. 빌드 결과
    표시	라우트	의미
    ○ Static	/, /login, /signup, /search, /me, /me/edit	빌드할 때 HTML 뼈대를 만들고, 데이터는 클라이언트에서 조회한다
    ◐ Partial Prerender	/movies/[movieId]	정적 부분만 미리 만들고, 동적 부분은 요청할 때 채운다
11. 트러블슈팅
    문제	원인	해결
    로그인, 중복 확인 요청 실패 (CORS)	백엔드가 localhost:5173만 허용한다	백엔드 CORS 허용 origin에 http://localhost:3000을 추가한다
    기존 로그인 상태가 사라짐	localStorage는 origin(포트)별로 따로 저장된다	다시 로그인한다 (계정은 DB에 그대로 있다)
    VS Code에서 @/... 모듈을 찾지 못함	paths를 바꾸기 전의 TS 서버 캐시가 남아 있다	TypeScript: Restart TS Server
    새로고침할 때 hydration 경고	서버는 localStorage를 못 읽어서 로그아웃 상태로 렌더링한다	동작에는 영향이 없다 (추후 개선 가능)