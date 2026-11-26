## 1. 나머지 API Swagger 문서화

| API | 토큰 | 실패 응답 |
| --- | --- | --- |
| GET /members/{memberId}/ratings | X | 404 MEMBER404_1 |
| POST /ratings | O | 401 AUTH401_2, 404 MEMBER404_1, 409 RATING409_1 |
| PATCH, DELETE /ratings/{ratingId} | O | 401 AUTH401_2, 403 RATING403_1, 404 RATING404_1 |
| GET /members/me | O | 401 AUTH401_2, 404 MEMBER404_1 |
| PATCH /members/me | O | 401 AUTH401_2, 404 MEMBER404_1, 409 MEMBER409_1 |
| GET /favorites, GET /favorites/{movieId} | O | 401 AUTH401_2 |
| DELETE /favorites/{movieId} | O | 401 AUTH401_2, 404 FAVORITE404_1 |
| POST /movie-records | O | 401 AUTH401_2, 404 MEMBER404_1, 409 RATING409_1 / FAVORITE409_1 |
- 공개 API와 인증 API가 섞인 컨트롤러는 `@SecurityRequirement`를 메서드 단위로 붙였다.
- 반복되는 실패 응답 예시는 `docs/SwaggerExamples`에 상수로 모았다.
- 회원가입이 문서에는 200, 실제로는 201이라 `Undocumented`가 표시되어 201 응답을 추가했다.

## 2. 평점 서비스 단위 테스트

| 테스트 | 확인 내용 |
| --- | --- |
| 같은 영화에 평점 중복 등록 | RATING409_1 예외, save 호출 안 됨 |
| 남의 평점 수정 | RATING403_1 예외, save 호출 안 됨 |
- builder로 만든 Member는 id가 null이라 `ReflectionTestUtils.setField`로 id를 넣었다.

## 3. 평점 API 통합 테스트

| 테스트 | 확인 내용 |
| --- | --- |
| 토큰 없이 POST /ratings | 401, AUTH401_2 |
| 토큰을 담아 POST /ratings | 201, COMMON201, DB에 저장됨 |

## 4. 전체 테스트 결과

- 단위 4개, 통합 7개, 컨텍스트 로딩 1개, 총 12개 통과