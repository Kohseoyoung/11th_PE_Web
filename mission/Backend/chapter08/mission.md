#### 1. 구현한 API

| API | Method | URL | 성공 응답 |
| --- | --- | --- | --- |
| 즐겨찾기 등록 | POST | `/favorites` | 201 |
| 내 즐겨찾기 조회 | GET | `/favorites` | 200 |
| 즐겨찾기 여부 조회 | GET | `/favorites/{movieId}` | 200 |
| 즐겨찾기 해제 | DELETE | `/favorites/{movieId}` | 200 |
| 영화 기록 저장 | POST | `/movie-records` | 201 |

모든 API는 `Authorization: Bearer {accessToken}` 헤더로 회원을 식별하며, URL·Body·쿼리에 `memberId`를 받지 않는다.

#### 2. 즐겨찾기 API 결과

| 요청 | 결과 | 캡처 |
| --- | --- | --- |
| `POST /favorites` (movieId 550) | 201, `COMMON201` | [캡처] |
| 같은 요청 재전송 | 409, `FAVORITE409_1` | [캡처] |
| `GET /favorites` | 200, 목록에 550 | [캡처] |
| `GET /favorites/550` | 200, `isFavorite: true` | [캡처] |
| `DELETE /favorites/550` | 200, `result: null` | [캡처] |
| `GET /favorites/550` (해제 후) | 200, `isFavorite: false` | [캡처] |
| `GET /favorites` (해제 후) | 200, `[]` | [캡처] |

#### 3. 영화 기록 저장과 트랜잭션

`POST /movie-records`는 `addToFavorites`가 `true`이면 평점과 즐겨찾기를 함께 저장한다. `false`이면 평점만 저장한다.

**트랜잭션 적용**

`RatingService`와 `FavoriteService`를 연달아 호출하면 각각 별도 트랜잭션이 될 수 있어서, 전용 `MovieRecordService`를 만들고 `create()` 메서드에 `@Transactional`을 붙였다. 저장 전에 평점 중복(`RATING409_1`)과 즐겨찾기 중복(`FAVORITE409_1`)을 먼저 검사한다.

**두 저장을 묶은 이유**: 평점만 저장되고 즐겨찾기가 저장되지 않는 상태를 막기 위해, 하나라도 실패하면 모두 rollback되도록 한 트랜잭션으로 묶었다.

#### 4. 실패 상황 확인 (실습 4)

| 상황 | 결과 | 캡처 |
| --- | --- | --- |
| 토큰 없이 `POST /favorites` | 401, `AUTH401_2` | [캡처] |
| 같은 `movieId` 두 번 등록 | 409, `FAVORITE409_1` | [캡처] |
| 즐겨찾기하지 않은 영화 여부 조회 | 200, `isFavorite: false` | [캡처] |
| 존재하지 않는 즐겨찾기 해제 | 404, `FAVORITE404_1` | [캡처] |
| `movieId`에 `0` 전송 | 400, `COMMON4001` | [캡처] |
| 빈 목록 조회 | 200, `[]` | [캡처] |

#### 5. 확인 포인트

- 요청의 URL, Body, 쿼리에 `memberId`가 없고, 토큰에서 회원을 식별한다.
- 영화 기록 Service에서 평점과 즐겨찾기 저장이 하나의 `@Transactional` 안에서 처리된다.
- 서버의 사전 검사는 명확한 오류 메시지를 위한 것이고, DB의 `(member_id, movie_id)` UNIQUE 제약은 동시 요청까지 막는 마지막 안전망이다.