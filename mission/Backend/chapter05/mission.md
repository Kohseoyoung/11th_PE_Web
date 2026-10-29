### 1. 미션 내용

필수 미션인 "실제 화면에서 확인하기"를 수행했다. CORS를 설정한 뒤 브라우저 화면(`localhost:5173`)에서 백엔드(`localhost:8080`)의 API 3개를 모두 호출해 정상 응답을 받았다.

### 2. 구현한 API

| API | Method | URL | 응답 |
| --- | --- | --- | --- |
| 닉네임 중복 체크 | GET | /members/nickname/{nickname} | `{ "available": boolean }` |
| 이메일 중복 체크 | GET | /members/email/{email} | `{ "available": boolean }` |
| 유저별 평점 조회 | GET | /members/{memberId}/ratings | 평점 목록 (최신순) |

### 3. 진행 과정

#### 3-1. 프로젝트 세팅

- 기존 Spring 워크북 레포 안에 `movie` 하위 폴더를 만들고, 그 안에 새 Gradle 프로젝트를 생성했다.
- Spring Boot 3.5.0, Java 21 기준으로 Web, Data JPA, MySQL Driver, Lombok, Validation 의존성을 사용했다.
- 레이어 기준으로 `controller`, `service`, `repository`, `entity`, `dto` 패키지를 나누었다.
- IntelliJ에서는 `movie` 폴더 자체를 열어 작업했다.

#### 3-2. DB 준비

- [이번에 사용한 DB와 테이블 이름을 적는다. 예: 이전 과제와 같은 `umc_11th_db`를 공유하고, 영화 서비스용 테이블은 ~로 구성했다.]
- 중복 체크와 평점 조회 확인용 더미 데이터를 member와 rating에 넣었다.
- `application.yml`에서 비밀번호는 `${DB_PASSWORD}` 환경변수로 처리해 레포에 올라가지 않게 했다.

#### 3-3. 코드 구현

| 계층 | 구현 내용 |
| --- | --- |
| Entity | `BaseTimeEntity`(createdAt, updatedAt 공통 부모), `Member`, `Rating` |
| Repository | `existsByNickname`, `existsByEmail`, `findByMember_MemberIdOrderByRatingIdDesc` |
| DTO | `NicknameCheckResponse`, `EmailCheckResponse`, `RatingResponse` (record) |
| Service | 중복 여부를 반대로 뒤집어 `available`로 반환, 존재하지 않는 유저는 404 처리 |
| Controller | `/members` 하위 3개 GET 엔드포인트 |

#### 3-4. Postman 검증

| API | 요청 | 결과 |
| --- | --- | --- |
| 닉네임 중복 체크 | GET /members/nickname/수현 | 200 OK, `available: false` |
| 이메일 중복 체크 | GET /members/email/noco@umc.com | 200 OK, `available: false` |
| 유저별 평점 조회 | GET /members/1/ratings | 200 OK, 평점 2건 최신순 |
| 존재하지 않는 유저 | GET /members/99/ratings | 404 Not Found |

#### 3-5. CORS 설정

- 브라우저는 동일 출처 정책 때문에 포트가 다른 서버의 응답을 막는다. 프론트(5173)와 백엔드(8080)는 출처가 다르므로 서버가 허용 헤더를 내려주어야 한다.
- `WebMvcConfigurer`를 구현한 `WebConfig`에서 `http://localhost:5173` 출처의 GET, POST, PATCH, DELETE 요청을 허용했다.

#### 3-6. 실제 화면에서 확인

- 간이 프론트(`index.html`)에 버튼 3개를 두고 각 API를 호출하게 했다. 정적 서버는 `npx serve -l 5173`으로 띄웠다.
- 세 API 모두 콘솔 에러 없이 화면에 응답이 표시되는 것을 확인했다.